---
title: Windows コンテナー用に gMSA を作成する
description: Windows コンテナー用にグループ管理サービス アカウント (gMSA) を作成する方法。
keywords: docker, コンテナー, active directory, gmsa, グループ管理サービス アカウント, グループ管理サービス アカウント
author: rpsqrd
ms.date: 01/03/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: d8faf3d16a7cd07016df334a2299ce4f0eca46f2
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985136"
---
# <a name="create-gmsas-for-windows-containers"></a>Windows コンテナー用に gMSA を作成する

Windows ベースのネットワークは、一般的に、Active Directory (AD) を使用して、ユーザー、コンピューター、およびその他のネットワーク リソース間の認証と承認を容易にしています。 エンタープライズ アプリケーション開発者は、多くの場合、アプリケーションを AD に統合し、ドメインに参加しているサーバーで実行するように設計して、統合 Windows 認証を利用しています。これにより、ユーザーや他のサービスは、ID を使用してアプリケーションに自動的かつ透過的にサインインできます。

Windows コンテナーはドメインに参加できませんが、Active Directory ドメイン ID を使用してさまざまな認証シナリオをサポートできます。

これを実現するには、[グループ管理サービス アカウント](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA) を使用して実行するように Windows コンテナーを構成します。これは、複数のコンピューターでパスワードを知ることなく ID を共有できるように設計された、Windows Server 2012 で導入された特殊な種類のサービス アカウントです。

gMSA を使用してコンテナーを実行すると、コンテナー ホストによって Active Directory ドメイン コントローラーから gMSA パスワードが取得され、コンテナー インスタンスに渡されます。 コンピューター アカウント (SYSTEM) でネットワーク リソースにアクセスする必要がある場合は常に、コンテナーでは gMSA 資格情報が使用されます。

この記事では、Windows コンテナーで Active Directory グループ管理サービス アカウントを使い始める方法について説明します。

## <a name="prerequisites"></a>前提条件

グループ管理サービス アカウントを使用して Windows コンテナーを実行するには、次のものが必要です。

- Windows Server 2012 以降を実行しているドメイン コントローラーが少なくとも 1 つある Active Directory ドメイン。 フォレストまたはドメインには、gMSA を使用するための機能レベルの要件はありませんが、gMSA パスワードを配布するには、Windows Server 2012 以降を実行しているドメイン コントローラーを使用する必要があります。 詳細については、[Active Directory の gMSA に関する要件](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)に関するページを参照してください。
- gMSA アカウントを作成するためのアクセス許可。 gMSA アカウントを作成するには、ドメイン管理者であるか、"*msDS-GroupManagedServiceAccount オブジェクトの作成*" のアクセス許可が委任されているアカウントを使用する必要があります。
- インターネットにアクセスして、CredentialSpec PowerShell モジュールをダウンロードします。 切断された環境で作業している場合は、インターネットにアクセスできるコンピューターに[モジュールを保存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)して、開発マシンまたはコンテナー ホストにコピーできます。

## <a name="one-time-preparation-of-active-directory"></a>Active Directory の 1 回限りの準備

ドメインで gMSA をまだ作成していない場合は、Key Distribution Service (KDS) ルート キーを生成する必要があります。 KDS は、gMSA パスワードの作成、ローテーション、および承認済みホストへのリリースの役割を担います。 コンテナー ホストで gMSA を使用してコンテナーを実行する必要がある場合は、KDS に接続して現在のパスワードを取得します。

KDS ルート キーが既に作成されているかどうかを確認するには、AD PowerShell ツールがインストールされているドメイン コントローラーまたはドメイン メンバー上でドメイン管理者として次の PowerShell コマンドレットを実行します。

```powershell
Get-KdsRootKey
```

コマンドからキー ID が返される場合は準備が完了しているので、「[グループ管理サービス アカウントを作成する](#create-a-group-managed-service-account)」セクションに進みます。 それ以外の場合は、KDS ルート キーの作成に進みます。

複数のドメイン コントローラーを使用する運用環境またはテスト環境では、PowerShell で次のコマンドレットをドメイン管理者として実行し、KDS ルート キーを作成します。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

キーがすぐに有効になることを意味するコマンドですが、KDS ルート キーがレプリケートされ、すべてのドメイン コントローラーで使用できるようになるまでに 10 時間かかります。

ドメイン内にドメイン コントローラーが 1 つしかない場合は、10 時間前にキーを有効になるように設定すると、プロセスを迅速にすることができます。

>[!IMPORTANT]
>運用環境ではこの手法を使用しないでください。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>グループ管理サービス アカウントを作成する

統合 Windows 認証を使用するすべてのコンテナーには、少なくとも 1 つの gMSA が必要です。 System または Network Service として実行されているアプリからネットワーク上のリソースにアクセスする場合は常に、プライマリ gMSA が使用されます。 コンテナーに割り当てられたホスト名に関係なく、gMSA の名前はネットワーク上のコンテナーの名前になります。 コンテナー コンピューター アカウントとは異なる ID としてコンテナーでサービスまたはアプリケーションを実行する場合に備えて、追加の gMSA を使用してコンテナーを構成することもできます。

gMSA を作成するときは、複数の異なるマシンで同時に使用できる共有 ID も作成します。 gMSA パスワードへのアクセスは、Active Directory アクセス制御リストによって保護されています。 gMSA アカウントごとにセキュリティ グループを作成し、関連するコンテナー ホストをセキュリティ グループに追加して、パスワードへのアクセスを制限することをお勧めします。

最後に、コンテナーにはサービス プリンシパル名 (SPN) が自動的に登録されないため、gMSA アカウント用に少なくともホスト SPN を手動で作成する必要があります。

通常、ホストまたは http SPN は gMSA アカウントと同じ名前を使用して登録されますが、クライアントがロード バランサーの背後からコンテナー化されたアプリケーションにアクセスする場合、または gMSA 名とは異なる DNS 名を使用する場合は、別のサービス名の使用が必要になる可能性があります。

たとえば、gMSA アカウントの名前が "WebApp01" で、ユーザーが `mysite.contoso.com` のサイトにアクセスする場合は、gMSA アカウントに `http/mysite.contoso.com` SPN を登録する必要があります。

アプリケーションによっては、独自のプロトコルのために追加の SPN が必要になる場合があります。 たとえば、SQL Server には `MSSQLSvc/hostname` SPN が必要です。

gMSA を作成するために必要な属性を次の表に示します。

|gMSA のプロパティ | 必須の値 | 例 |
|--------------|----------------|--------|
|名前 | 任意の有効なアカウント名。 | `WebApp01` |
|DnsHostName | アカウント名に付加されるドメイン名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 少なくともホスト SPN を設定し、必要に応じて他のプロトコルを追加します。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | コンテナー ホストを含むセキュリティ グループ。 | `WebApp01Hosts` |

gMSA の名前を決定したら、PowerShell で次のコマンドレットを実行して、セキュリティ グループと gMSA を作成します。

> [!TIP]
> 次のコマンドを実行するには、**Domain Admins** セキュリティ グループに属するアカウントか、**msDS-GroupManagedServiceAccount オブジェクトの作成**アクセス許可が委任されているアカウントを使用する必要があります。
> [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) コマンドレットは、[リモート サーバー管理ツール](https://aka.ms/rsat)の AD PowerShell Tools の一部です。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01$", "ContainerHost02$", "ContainerHost03$"
```

開発、テスト、運用の各環境用に個別の gMSA アカウントを作成することをお勧めします。

## <a name="prepare-your-container-host"></a>コンテナー ホストを準備する

gMSA を使用して Windows コンテナーを実行する各コンテナー ホストは、ドメインに参加し、gMSA パスワードを取得できるアクセス権を持っている必要があります。

1. コンピューターを Active Directory ドメインに参加させます。
2. ホストが gMSA パスワードへのアクセスを制御するセキュリティ グループに属していることを確認します。
3. コンピューターを再起動して、新しいグループ メンバーシップを取得します。
4. [Docker Desktop for Windows 10](https://docs.docker.com/docker-for-windows/install/) または [Docker for Windows Server](https://docs.docker.com/install/windows/docker-ee/) を設定します。
5. (推奨) [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) を実行して、ホストで gMSA アカウントを使用できることを確認します。 コマンドから **False** が返される場合は、[トラブルシューティングの手順](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)に従ってください。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>資格情報の指定を作成する

資格情報の指定ファイルは、コンテナーで使用する gMSA アカウントに関するメタデータを含む JSON ドキュメントです。 ID 構成をコンテナー イメージから分離しておくと、資格情報の指定ファイルを交換するだけで、コンテナーに使用する gMSA を変更できます。コードを変更する必要はありません。

資格情報の指定ファイルを作成するには、ドメインに参加しているコンテナー ホストで [CredentialSpec PowerShell モジュール](https://aka.ms/credspec)を使用します。
ファイルを作成したら、それを他のコンテナー ホストまたはコンテナー オーケストレーターにコピーできます。
コンテナーに代わってコンテナー ホストによって gMSA が取得されるため、資格情報の指定ファイルには gMSA パスワードなどのシークレットは含まれていません。

Docker では、Docker データ ディレクトリ内の **CredentialSpecs** ディレクトリ以下に資格情報の指定ファイルがあると想定されています。 既定のインストールでは、このフォルダーは `C:\ProgramData\Docker\CredentialSpecs` にあります。

コンテナー ホスト上で資格情報の指定ファイルを作成するには、次の手順を実行します。

1. RSAT AD PowerShell ツールをインストールする
    - Windows サーバーの場合は、**Install-WindowsFeature RSAT-AD-PowerShell** を実行します。
    - Windows 10 バージョン 1809 以降の場合、**Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'** を実行します。
    - 以前のバージョンの Windows 10 については、<https://aka.ms/rsat> を参照してください。
2. 次のコマンドレットを実行して、[CredentialSpec PowerShell モジュール](https://aka.ms/credspec)の最新バージョンをインストールします。

    ```powershell
    Install-Module CredentialSpec
    ```

    コンテナー ホストでインターネットにアクセスできない場合は、インターネットに接続されたマシンで `Save-Module CredentialSpec` を実行し、モジュール フォルダーを `C:\Program Files\WindowsPowerShell\Modules` またはコンテナー ホストの `$env:PSModulePath` の別の場所にコピーします。

3. 次のコマンドレットを実行して、新しい資格情報の指定ファイルを作成します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    このコマンドレットを実行すると、既定では、指定した gMSA 名がコンテナーのコンピューター アカウントとして使用され、資格情報の指定が作成されます。 このファイルは、gMSA ドメインとファイル名のアカウント名を使用して、Docker CredentialSpecs ディレクトリに保存されます。

    ファイルを別のディレクトリに保存する場合は、`-Path` パラメーターを使用します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -Path "C:\MyFolder\WebApp01_CredSpec.json"
    ```

    コンテナーでセカンダリ gMSA としてサービスまたはプロセスを実行している場合は、追加の gMSA アカウントを含む資格情報の指定を作成することもできます。 そのためには、`-AdditionalAccounts` パラメーターを使用します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    サポートされているパラメーターの完全な一覧については、`Get-Help New-CredentialSpec -Full` を実行します。

4. 次のコマンドレットを使用して、すべての資格情報の指定とその完全なパスの一覧を表示できます。

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>次の手順

gMSA アカウントを設定したので、これを使用して次のことを実行できます。

- [アプリを構成する](gmsa-configure-app.md)
- [コンテナーを実行する](gmsa-run-container.md)
- [コンテナーを調整する](gmsa-orchestrate-containers.md)

設定中に問題が発生した場合は、[トラブルシューティング ガイド](gmsa-troubleshooting.md)で考えられる解決策をご確認ください。

## <a name="additional-resources"></a>その他の資料

- gMSA の詳細については、「[グループ管理サービス アカウントの概要](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)」を参照してください。
- ビデオ デモについては、Ignite 2016 の[録画されたデモ](https://youtu.be/cZHPz80I-3s?t=2672)をご覧ください。
