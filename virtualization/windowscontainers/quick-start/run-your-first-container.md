---
title: Windows 10 上の Windows コンテナーおよび Linux コンテナー
description: コンテナー展開のクイック スタート
keywords: docker, コンテナー, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: quickstart
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 950097bae359ce717e1f4427aaec0b87198fef42
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192899"
---
# <a name="get-started-run-your-first-windows-container"></a>作業の開始:最初の Windows コンテナーを実行する

このトピックでは、次のトピックで説明されているように環境をセットアップした後に、最初の Windows コンテナーを実行する方法を説明します: 「[作業の開始:コンテナー用の Windows を準備する](./set-up-environment.md)」。 コンテナーを実行するには、最初に基本イメージをインストールします。これにより、コンテナーにオペレーティング システム サービスの基本の層が提供されます。 次に、基本イメージに基づいたコンテナー イメージを作成して実行します。 詳細については、以下の説明を参照してください。

## <a name="install-a-container-base-image"></a>コンテナーの基本イメージをインストールする

すべてのコンテナーは、コンテナー イメージから作成されます。 Microsoft では、基本イメージと呼ばれるスターター イメージをいくつか提供しています (詳細については「[コンテナーの基本イメージ](../manage-containers/container-base-images.md)」を参照してください)。 この手順では、軽量な Nano Server 基本イメージをプル (ダウンロードしてインストール) します。

1. コマンド プロンプト ウィンドウ (組み込みコマンド プロンプト、PowerShell、[Windows ターミナル](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)など) を開き、次のコマンドを実行して基本イメージをダウンロードし、インストールします。

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > `no matching manifest for unknown in the manifest list entries` というエラー メッセージが表示される場合は、Linux コンテナーを実行するように Docker が構成されていないことを確認してください。

2. イメージのダウンロードが終了したら、ローカルの docker イメージ リポジトリにクエリを実行し、そのイメージがシステム上に存在することを確認します。ダウンロードの待機中には [EULA](../images-eula.md) をお読みください。 コマンド `docker images` を実行すると、インストールされているイメージの一覧が返されます。

   次に、Nano Server イメージを示している出力の例を示します。

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Windows コンテナーを実行する

この簡単な例では、"Hello World" というコンテナー イメージを作成して展開します。 これらのコマンドは、最適なエクスペリエンスのために、管理者特権でのコマンド プロンプト ウィンドウで実行します (ただし、Windows PowerShell ISE は使用しないでください。コンテナーがハングしているように見えるため、コンテナーを扱う対話型セッションでは機能しません)。

1. コマンド プロンプト ウィンドウに次のコマンドを入力して、対話型セッションで `nanoserver` イメージからコンテナーを起動します。

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. コンテナーの起動後、コマンド プロンプト ウィンドウにより、コンテキストがコンテナーに変更されます。 コンテナー内で単純な "Hello World" テキスト ファイルを作成してから、次のコマンドを入力してコンテナーを終了します。

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```

3. [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) コマンドを実行し、先ほど終了したコンテナーのコンテナー ID を取得します。

   ```console
   docker ps -a
   ```

4. 最初に実行したコンテナーでの変更が含まれる新しい "HelloWorld" イメージを作成します。 これを行うには、[docker commit](https://docs.docker.com/engine/reference/commandline/commit/) コマンドを実行し、`<containerid>` を実際のコンテナー ID に置き換えます。

   ```console
   docker commit <containerid> helloworld
   ```

   操作を完了すると、hello world スクリプトを含むカスタム イメージが作成されます。 これは、[docker images](https://docs.docker.com/engine/reference/commandline/images/) コマンドで確認できます。

   ```console
   docker images
   ```

   この場合の出力例を以下に示します。

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 最後に、コマンドライン (cmd.exe) が停止したらコンテナーを自動的に削除する `--rm` パラメーターを指定して [docker run](https://docs.docker.com/engine/reference/commandline/run/) コマンドを使用し、新しいコンテナーを実行します。

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```
   この結果、Docker は "HelloWorld" イメージからコンテナーを作成し、Docker はそのコンテナーで cmd.exe のインスタンスを開始し、cmd.exe はファイルを読み取り、シェルにそのコンテンツを出力しました。 最後の手順として、Docker はコンテナーを停止し、削除しました。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [サンプル アプリをコンテナー化する方法を確認する](./building-sample-app.md)
