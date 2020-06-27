---
title: Windows Dockerfile を最適化する
description: Windows コンテナー用に Dockerfile を最適化します。
keywords: Docker, コンテナー
author: cwilhit
ms.date: 05/03/2019
ms.topic: tutorial
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: 25297432a3e515a130c9710431fd550f2efd8226
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192359"
---
# <a name="optimize-windows-dockerfiles"></a>Windows Dockerfile を最適化する

Docker のビルド プロセスと結果の Docker イメージの両方を最適化する方法は多数あります。 この記事では、Docker のビルド プロセスがどのように機能するかと、Windows コンテナーのイメージを最適に作成する方法について説明します。

## <a name="image-layers-in-docker-build"></a>Docker ビルド内のイメージ レイヤー

Docker ビルドを最適化するには、Docker のビルドがどのように機能するかを理解する必要があります。 Docker ビルド プロセスでは、Dockerfile が使用されて、アクション可能な各命令が 1 つずつ専用の一時的なコンテナーで実行されます。 その結果、アクション可能な各命令に対して新しいイメージ レイヤーが作成されます。

たとえば、次のサンプル Dockerfile では、`mcr.microsoft.com/windows/servercore:ltsc2019` ベース OS イメージを使用して、IIS をインストールし、簡単な Web サイトを作成しています。

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

この Dockerfile では、2 つのレイヤーを持つイメージが生成されると想定されています。1 つはコンテナー OS イメージ用、2 つ目は IIS と Web サイトを含むものです。 ただし、実際のイメージには多くのレイヤーがあって、各レイヤーはその前のレイヤーに依存しています。

これを明確にするために、サンプルの Dockerfile で作成されたイメージに対して `docker history` コマンドを実行してみましょう。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

出力には、このイメージには 4 つのレイヤーがあることが示されています。基本レイヤーと、Dockerfile 内の各命令にマップされている 3 つの追加レイヤーです。 最下位のレイヤー (この例では `6801d964fda5`) は、ベース OS イメージを表します。 1 つ上のレイヤーは、IIS のインストールです。 次のレイヤーには、新しい Web サイトなどが含まれます。

Dockerfiles は、イメージ レイヤーの最小化、ビルドのパフォーマンスの最適化、読みやすさを通したアクセシビリティの最適化を目的として記述できます。 最終的に、同じイメージ ビルド タスクを実行するにはさまざまな方法があります。 Dockerfile の形式がビルド時間と作成されるイメージにどのように影響するかを理解すれば、オートメーションのエクスペリエンスを向上させることになります。

## <a name="optimize-image-size"></a>イメージ サイズを最適化する

対象分野の要件によっては、Docker コンテナー イメージを作成するときにイメージ サイズが重要な要素になることがあります。 コンテナー イメージは、レジストリとホストの間を移動され、エクスポートおよびインポートされて、最終的にはスペースを消費します。 このセクションでは、Windows コンテナーの Docker ビルド プロセス時にイメージ サイズを最小限に抑える方法について説明します。

Dockerfile のベスト プラクティスについては、[Docker.com の、Dockerfile を作成するときのベスト プラクティス](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)を参照してください。

### <a name="group-related-actions"></a>関連するアクションをグループ化する

それぞれの `RUN` 命令によってコンテナー イメージ内に 1 つの新しいレイヤーが作成されるため、複数のアクションを 1 つの `RUN` 命令にグループ化すると、Dockerfile のレイヤーの数を減らすことができます。 レイヤーの数を最小限に抑えてもイメージのサイズに大きな影響はありませんが、後の例で示すように、関連するアクションをグループ化するとイメージのサイズに大きな影響があります。

このセクションでは、同じことを実行する Dockerfile の 2 つの例を比較します。 ただし、1 つの Dockerfile にはアクションごとに 1 つの命令があるのに対して、別の Dockerfile では関連するアクションがグループ化されています。

グループ化されていない次の例の Dockerfile では、Windows 用の Python をダウンロードしてインストールし、インストールが完了したらダウンロードしたセットアップ ファイルを削除します。 この Dockerfile では、各アクションに固有の `RUN` 命令が指定されています。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

結果として作成されるイメージは、`RUN` 命令ごとに 1 つずつ、3 つの追加レイヤーで構成されます。

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

2 番目の例は、まったく同じ操作を実行する Dockerfile です。 ただし、関連するすべてのアクションが単一の `RUN` 命令のもとにグループ化されています。 `RUN` 命令内の各ステップは、Dockerfile では新しい行に記述されていて、改行には "\\" 文字が使用されています。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

結果のイメージにあるのは、`RUN` 命令に対する 1 つの追加レイヤーだけです。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>余分なファイルを削除する

Dockerfile 内に、使用後には必要がないインストーラーなどのファイルがある場合は、それを削除してイメージ サイズを小さくすることができます。 この処理は、イメージ レイヤーへのファイルのコピーが行われるのと同じステップで行う必要があります。 そうすれば、ファイルが下位レベルのイメージ レイヤーに残らないようにできます。

次の例の Dockerfile では、Python パッケージがダウンロードされて実行された後、削除されます。 このすべてを 1 つの `RUN` 操作で行うので、作成されるイメージ レイヤーは 1 つです。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>ビルドの速さを最適化する

### <a name="multiple-lines"></a>複数の行

操作を複数の独立した命令に分割し、Docker のビルドの速さを最適化することができます。 複数の `RUN` 操作では、`RUN` 命令ごとに独立したレイヤーが作成されるため、キャッシュの有効性が向上します。 まったく同じ命令が、異なる Docker ビルド操作で既に実行されている場合、このキャッシュ済みの操作 (イメージ レイヤー) が再利用されるので、Docker ビルドの実行時間が短縮されます。

次の例では、Apache と Visual Studio の両方の再配布パッケージをダウンロードし、インストールした後、不要になったファイルの削除によってクリーンアップを行っています。 これは、1 つの `RUN` 命令ですべて実行されます。 これらのアクションのいずれかが更新された場合、すべてのアクションが再実行されます。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \

  # Download software ; \

  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \

  # Install Software ; \

  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \

  # Remove unneeded files ; \

  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

結果のイメージには 2 つのレイヤーがあります。1 つはベース OS イメージ用で、もう 1 つのレイヤーには 1 つの `RUN` 命令のすべての操作が含まれています。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

比較して、こちらでは同じ操作が 3 つの `RUN` 命令に分割されています。 この場合、各 `RUN` 命令はコンテナー イメージ レイヤーにキャッシュされており、以降の Dockerfile ビルドでは、再実行する必要があるのは変更されたものだけです。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

結果のイメージは、4 つのレイヤーで構成されます。ベース OS イメージ用のレイヤーが 1 つと、3 つの `RUN` 命令それぞれに 1 つです。 各 `RUN` 命令は独自のレイヤーで実行されたため、それ以降、この Dockerfile を実行したり、異なる Dockerfile で同じ命令セットを実行したりすると、キャッシュされているイメージ レイヤーが使用され、ビルド時間が短縮されます。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

次のセクションで説明するように、イメージ キャッシュを使用する場合は、命令の順序をどのようにするかが重要です。

### <a name="ordering-instructions"></a>命令の順序指定

Dockerfile は上から下に処理され、各命令はキャッシュされているレイヤーと比較されます。 キャッシュされたレイヤーがない命令が見つかると、その命令およびそれ以降のすべての命令が新しいコンテナー イメージ レイヤーで処理されます。 このため、命令の配置の順序が重要になります。 変化しない命令は、Dockerfile の前の方に配置します。 変化する可能性のある命令は、Dockerfile の後の方に配置します。 そうすることで、既存のキャッシュが役に立たなくなる可能性が減ります。

この例は、Dockerfile の命令の順序指定がキャッシュの有効性にどのような影響を与えるかを示しています。 この簡単な例の Dockerfile には、番号が付いたフォルダーが 4 つあります。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

結果のイメージには 5 つのレイヤーがあります。ベース OS イメージ用に 1 つ、`RUN` 命令ごとに 1 つずつです。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

次に示すこの Dockerfile は少し変更されて、3 つ目の `RUN` 命令が新しいファイルに変更されました。 この Dockerfile に対して Docker ビルドを実行すると、前の例と同じ最初の 3 つの命令は、キャッシュされたイメージ レイヤーを使用します。 ただし、変更された `RUN` 命令はキャッシュされていないので、変更された命令と、後続のすべての命令に対して新しいレイヤーが作成されます。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

新しいイメージのイメージ ID を、このセクションの最初の例に含まれるものと比較すると、下から上に最初の 3 つのレイヤーは共有されていますが、4 つ目と 5 つ目は一意になっていることがわかります。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>見た目の最適化

### <a name="instruction-case"></a>命令の大文字と小文字の区別

Dockerfile の命令の大文字と小文字は区別されませんが、規則では大文字を使うようになっています。 これにより、命令の呼び出しと命令の操作が区別されて、読みやすさが向上しています。 以下の 2 つの例では、大文字で記述していない Dockerfile と大文字で記述してある Dockerfile を比較しています。

次に示すのは、大文字で記述していない Dockerfile です。

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

次に示すのは、大文字を使用している同一の Dockerfile です。

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>行の折り返し

長くて複雑な操作は、バックスラッシュ `\` 文字によって複数行に分割できます。 次の Dockerfile は、Visual Studio の再頒布可能パッケージをインストールし、インストーラー ファイルを削除して、構成ファイルを作成します。 これら 3 つの操作すべてが 1 行で指定されています。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

1 つの `RUN` 命令の各操作が独自の行で指定されるように、バックスラッシュを使用してコマンドを分割できます。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>この後の参考資料

[Windows 上の Dockerfile](manage-windows-dockerfile.md)

[Dockerfile を作成するときのベスト プラクティス (Docker.com)](https://docs.docker.com/engine/reference/builder/)
