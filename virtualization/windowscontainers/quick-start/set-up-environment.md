---
title: Windows 10 上の Windows コンテナーおよび Linux コンテナー
description: コンテナー用の Windows 10 または Windows Server をセットアップしたら、最初のコンテナー イメージの実行に進みます。
keywords: docker, コンテナー, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 13d8f1ead90b2c2c86afe9f596717c1c09905895
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "80929979"
---
# <a name="get-started-prep-windows-for-containers"></a>作業の開始:コンテナー用の Windows を準備する

このチュートリアルでは、以下の方法について説明します。

- コンテナー用の Windows 10 または Windows Server をセットアップする
- 最初のコンテナー イメージを実行する
- 単純な .NET Core アプリケーションをコンテナー化する

## <a name="prerequisites"></a>前提条件

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server でコンテナーを実行するには、Windows Server (半期チャネル)、Windows Server 2019、または Windows Server 2016 を実行している物理サーバーまたは仮想マシンが必要です。

テスト用として、[Windows Server 2019 評価版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )または [Windows Server Insider Preview](https://insider.windows.com/for-business-getting-started-server/) のコピーをダウンロードできます。

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

Windows 10 でコンテナーを実行するには、以下のものが必要です。

- Anniversary Update (バージョン 1607) 以降を適用した Windows 10 Professional または Enterprise を実行する 1 台の物理コンピューター システム。
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) を有効にする必要があります。

> [!NOTE]
>  Windows 10 October Update 2018 以降、開発またはテスト用として、Windows 10 Enterprise または Professional のプロセス分離モードで Windows コンテナーを実行することをユーザーに許可するようになりました。 詳細については、[FAQ](../about/faq.md) に関するページを参照してください。 
> 
> Windows Server コンテナーでは、開発者に運用環境で使用されるのと同じカーネル バージョンと構成を提供するため、既定では Windows 10 上の Hyper-V を使用します。 Hyper-v の分離の詳細については、Microsoft ドキュメントの、[概念](../manage-containers/hyperv-container.md)に関するページを参照してください。

---
<!-- stop tab view -->

## <a name="install-docker"></a>Docker のインストール

最初の手順は、Windows コンテナーを操作するために必要な Docker をインストールすることです。 Docker では、共通の API とコマンド ライン インターフェイス (CLI) を使用して、コンテナーの標準ランタイム環境が提供されます。

構成の詳細については、「[Windows 上の Docker エンジン](../manage-docker/configure-docker-daemon.md)」を参照してください。

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

Windows Server に Docker をインストールするには、Microsoft によって発行された [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider) という [OneGet プロバイダー PowerShell モジュール](https://github.com/oneget/oneget)を利用できます。 このプロバイダーにより、Windows でコンテナー機能が有効になり、Docker のエンジンとクライアントがインストールされます。 以下にその方法を示します。

1. 管理者特権で PowerShell セッションを開き、[PowerShell ギャラリー](https://www.powershellgallery.com/packages/DockerMsftProvider)から Docker-Microsoft PackageManagement Provider をインストールします。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   NuGet プロバイダーのインストールを求めるメッセージが表示されたら、「`Y`」と入力して同様にインストールします。

2. PackageManagement PowerShell モジュールを使用して、最新バージョンの Docker をインストールします。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   PowerShell でパッケージ ソース "DockerDefault" を信頼するかどうかの確認を求められたら、「`A`」と入力してインストールを続行します。
3. インストールが完了したら、コンピューターを再起動します。

   ```powershell
   Restart-Computer -Force
   ```

後で Docker を更新する場合は、次のことを行います。

- `Get-Package -Name Docker -ProviderName DockerMsftProvider` を実行して、インストールされているバージョンを確認します
- `Find-Package -Name Docker -ProviderName DockerMsftProvider` を実行して、最新のバージョンを検索します
- 準備ができたら、`Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` を実行してアップグレードした後、`Start-Service Docker` を実行します

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

以下の手順を使用して、Windows 10 Professional エディションおよび Enterprise エディションに Docker をインストールできます。 

1. [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) をダウンロードしてインストールします。まだ持っていない場合は無料の Docker アカウントを作成します。 詳細については、[Docker のマニュアル](https://docs.docker.com/docker-for-windows/install)を参照してください。

2. インストール時に、既定のコンテナーの種類を Windows コンテナーに設定します。 切り替えをインストール完了後に行うには、次に示すように Windows システム トレイの Docker 項目を使用するか、PowerShell プロンプトで次のコマンドを使用します。

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![[Switch to Windows containers] (Windows コンテナーに切り替え) コマンドが表示されている Docker システム トレイ メニュー。](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>次の手順

環境が正しく構成されたので、次のリンクを使用してコンテナーの実行方法を確認してください。

> [!div class="nextstepaction"]
> [最初のコンテナーを実行する](./run-your-first-container.md)
