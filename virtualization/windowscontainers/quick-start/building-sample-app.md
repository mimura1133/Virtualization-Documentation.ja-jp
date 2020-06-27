---
title: .NET Core アプリをコンテナー化する
description: コンテナーを使用してサンプル .NET Core アプリを構築する方法について説明します
keywords: Docker, コンテナー
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: quickstart
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 79de3f520e6d22c338fc95473200286b77cb4467
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192519"
---
# <a name="containerize-a-net-core-app"></a>.NET Core アプリをコンテナー化する

このトピックでは、既存のサンプル .NET アプリをデプロイするために Windows コンテナーとしてパッケージ化する方法を説明します。この作業は、「[作業の開始:コンテナー用の Windows を準備する](set-up-environment.md)」で説明されているように環境をセットアップし、「[最初の Windows コンテナーを実行する](run-your-first-container.md)」で説明されているように最初のコンテナーを実行した後に行います。

また、コンピューターに Git ソース管理システムがインストールされている必要があります。 これをインストールするには、[Git](https://git-scm.com/download) にアクセスしてください。

## <a name="clone-the-sample-code-from-github"></a>GitHub からサンプル コードを複製する

すべてのコンテナー サンプル ソース コードは、`windows-container-samples` という名前のフォルダー内の [Virtualization-Documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation) Git リポジトリ (略式で "リポ" とも呼ばれます) に保管されています。

1. PowerShell セッションを開き、このリポジトリを格納しようとしているフォルダーにディレクトリを移動します。 (他の種類のコマンド プロンプト ウィンドウも同様に機能しますが、このコマンド例では PowerShell を使用しています。)
2. 現在の作業ディレクトリにリポジトリを複製します。

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 以下のコマンドを使用して、`Virtualization-Documentation\windows-container-samples\asp-net-getting-started` の下にあるサンプル ディレクトリに移動し、Dockerfile を作成します。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/) はメイクファイルに似ています。これは、コンテナー イメージの構築方法をコンテナー エンジンに指示する手順の一覧です。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Dockerfile を記述する

先ほど作成した Dockerfile を任意のテキスト エディターで開き、以下の内容を追加します。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

各行に分けて、それぞれの手順で何が行われるかを説明します。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

最初の一連の行では、コンテナー構築の基盤として使う基本イメージを宣言します。 ローカル システムにこのイメージがまだ存在しない場合、docker が自動的に取得を試みます。 `mcr.microsoft.com/dotnet/core/sdk:2.1` は、インストールされた .NET Core 2.1 SDK と一緒のパッケージで提供されているため、バージョン 2.1 をターゲットとする ASP .NET Core プロジェクトをビルドするタスクに対応できます。 次の手順では、コンテナー内の作業ディレクトリが `/app` になるように変更しています。これで、この後に続くすべてのコマンドがこのコンテキストで実行されます。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

次にこれらの手順で、.csproj ファイルを `build-env` コンテナーの `/app` ディレクトリにコピーします。 このファイルをコピーすると、そこから .NET の読み取りが行われ、プロジェクトに必要なすべての依存関係とツールが外部から取得されます。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

すべての依存関係が .NET によって `build-env` コンテナーにプルされると、次の手順で、すべてのプロジェクト ソース ファイルがコンテナーにコピーされます。 次に、アプリケーションをリリース構成と共に公開するよう.NET に指定し、出力パスを指定します。

コンパイルが成功するはずです。 ここで、最終イメージをビルドする必要があります。

> [!TIP]
> このクイックスタートでは、ソースから .NET Core プロジェクトをビルドします。 コンテナー イメージのビルド時には、運用環境のペイロードとその依存関係_のみ_をコンテナー イメージに含めることが推奨されます。 .Net Core ランタイムのみが必要なため、.NET Core SDK を最終イメージに含めたくありません。そのため、Dockerfile は、アプリをビルドするために SDK と共にパッケージ化された、`build-env` という名前の一時コンテナーを使用するように記述されます。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

アプリケーションが ASP.NET であるため、このラインタイムを含めてイメージを指定します。 次に、一時コンテナーの出力ディレクトリから、すべてのファイルを最終コンテナーにコピーします。 コンテナーは、コンテナーの起動時に新しいアプリをエントリ ポイントとして使用して実行するように構成します

Dockerfile は、_多段階ビルド_を実行するように記述しました。 Dockerfile を実行するときには、一時コンテナーである `build-env` を .NET Core 2.1 SDK と共に使用してサンプル アプリをビルドした後、最終的なコンテナーのサイズを最小限にするために、出力されたバイナリを、.NET Core 2.1 ランタイムのみを含む別のコンテナーにコピーします。

## <a name="build-and-run-the-app"></a>アプリをビルドして実行する

Dockerfile が記述されたので、Docker に Dockerfile を指し示して、イメージをビルドしてから実行するように指示できます。

1. コマンド プロンプト ウィンドウで、Dockerfile が置かれているディレクトリに移動し、[docker build](https://docs.docker.com/engine/reference/commandline/build/) コマンドを実行して Dockerfile からコンテナーをビルドします。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 新しくビルドされたコンテナーを実行するには、[docker run](https://docs.docker.com/engine/reference/commandline/run/) コマンドを実行します。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   このコマンドの詳細を確認しておきましょう。

   * `-d` により、Docker は 'デタッチ' されたコンテナーを実行します。これは、コンテナー内のコンソールにフックされるコンソールがないということです。 コンテナーはバックグラウンドで実行されます。
   * `-p 5000:80` により、ホスト上のポート 5000 をコンテナーのポート 80 にマップするよう Docker に指示します。 各コンテナーは、独自の IP アドレスを取得します。 ASP .NET は既定で、ポート 80 でリッスンします。 ポート マッピングを使用すると、マップされたポートでホストの IP アドレスに移動することができ、Docker はすべてのトラフィックをコンテナー内の宛先ポートに転送するようになります。
   * `--name myapp` では、Docker によって実行時に割り当てられるコンテナー ID を検索する代わりに、クエリで使用するのに便利な名前をこのコンテナーに割り当てるよう Docker に指示します。
   * `my-asp-app` は、Docker の実行対象となるイメージです。 これは、`docker build` プロセスの結果として生成されるコンテナー イメージです。

3. Web ブラウザーを開いて `http://localhost:5000` に移動し、次のスクリーンショットに示すように、コンテナー化されたアプリケーションを表示します。

   >![コンテナー内の localhost から実行されている ASP.NET Core Web ページ](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>次の手順

1. 次の手順は、コンテナー化された ASP.NET Web アプリを、Azure Container Registry を使用してプライベート レジストリに発行することです。 この操作で、アプリを組織内でデプロイすることができます。

   > [!div class="nextstepaction"]
   > [プライベート コンテナー レジストリを作成する](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   [コンテナー イメージをレジストリ](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)にプッシュするセクションに移動したら、コンテナー レジストリ (`my-asp-app`) と共に、パッケージ化したばかりの ASP.NET アプリの名前 (例: `contoso-container-registry`) を指定します。

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   アプリのサンプルとそれに関連付けられている Dockerfile の詳細については、[その他のコンテナー サンプル](../samples.md)を参照してください。

2. アプリをコンテナー レジストリに発行したら、次の手順は、Azure Kubernetes Service で作成する Kubernetes クラスターにアプリをデプロイすることです。

   > [!div class="nextstepaction"]
   > [Kubernetes クラスターを作成する](https://docs.microsoft.com/azure/aks/windows-container-cli)
