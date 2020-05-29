---
title: Windows コンテナーの gMSA のトラブルシューティング
description: Windows コンテナーのグループ管理サービス アカウント (gMSA) のトラブルシューティング方法。
keywords: docker, コンテナー, active directory, gmsa, グループ管理サービス アカウント, グループ管理サービス アカウント, トラブルシューティング, 解決
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910242"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Windows コンテナーの gMSA のトラブルシューティング

## <a name="known-issues"></a>既知の問題

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>コンテナーのホスト名は、Windows Server 2016 および Windows 10 バージョン 1709 および 1803 の gMSA 名と一致する必要があります

Windows Server 2016 バージョン 1709 または 1803 を実行している場合、コンテナーのホスト名は gMSA SAM アカウント名と一致する必要があります。

ホスト名が gMSA 名と一致しない場合、受信 NTLM 認証要求と名前および SID 変換 (ASP.NET メンバーシップ ロール プロバイダーなどの多くのライブラリで使用されます) は失敗します。 ホスト名と gMSA 名が一致しない場合でも、Kerberos は引き続き正常に機能します。

この制限は Windows Server 2019 で修正され、割り当てられたホスト名に関係なく、コンテナーには常にネットワーク上の gMSA 名が使用されるようになりました。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>複数のコンテナーで gMSA を同時に使用すると、Windows Server 2016 と Windows 10 バージョン 1709 および 1803 で断続的な障害が発生する

すべてのコンテナーで同じホスト名を使用する必要があるため、2 つ目の問題は、Windows Server 2019 より前のバージョンの Windows および Windows 10 バージョン 1809 に影響します。 複数のコンテナーに同じ ID とホスト名が割り当てられている場合、同時に 2 つのコンテナーが同じドメイン コントローラーと通信すると、競合状態が発生することがあります。 別のコンテナーが同じドメイン コントローラーと通信すると、同じ ID を使用している以前のコンテナーとの通信は取り消されます。 これにより、断続的な認証エラーが発生する可能性があります。また、コンテナー内で `nltest /sc_verify:contoso.com` を実行すると、信頼エラーとして確認されることがあります。

Windows Server 2019 の動作を変更して、コンテナー ID をマシン名から分離し、複数のコンテナーが同じ gMSA を同時に使用できるようにしました。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Windows 10 バージョン 1703、1709、および 1803 では、Hyper-V による分離コンテナーで gMSA を使用できません

Windows 10 および Windows Server バージョン 1703、1709、および 1803 上で Hyper-V による分離コンテナーと共に gMSA を使用しようとすると、コンテナーの初期化が停止するか、失敗します。

このバグは、Windows Server 2019 および Windows 10 バージョン 1809 で修正されました。 Windows Server 2016 および Windows 10 バージョン 1607 上で gMSA を使用して Hyper-V による分離コンテナーを実行することもできます。

## <a name="general-troubleshooting-guidance"></a>一般的なトラブルシューティング ガイダンス

gMSA を使用してコンテナーを実行しているときにエラーが発生した場合は、次の手順で根本的な原因を特定することができます。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>ホストで gMSA を使用できることを確認する

1. ホストがドメインに参加していて、ドメイン コントローラーに到達できることを確認します。
2. RSAT から AD PowerShell Tools をインストールし、[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) を実行して、コンピューターに gMSA を取得するためのアクセス権があるかどうかを確認します。 コマンドレットから **False** が返される場合、コンピューターは gMSA のパスワードにアクセスできません。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. **Test-ADServiceAccount** から **False** が返される場合、gMSA パスワードにアクセスできるセキュリティ グループにホストが属していることを確認します。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. gMSA のパスワードを取得できるセキュリティ グループにホストが属していても、**Test-ADServiceAccount** が失敗する場合は、必要に応じて、コンピューターを再起動し、現在のグループ メンバーシップを反映する新しいチケットを取得します。

#### <a name="check-the-credential-spec-file"></a>資格情報の指定ファイルを確認する

1. [CredentialSpec PowerShell モジュール](https://aka.ms/credspec)から **Get-CredentialSpec** を実行して、マシン上のすべての資格情報の指定を特定します。 資格情報の指定は、Docker ルート ディレクトリ以下の "CredentialSpecs" ディレクトリに格納する必要があります。 **docker info -f "{{.DockerRootDir}}"** を実行すると、Docker ルート ディレクトリを見つけることができます。
2. CredentialSpec ファイルを開き、次のフィールドが正しく入力されていることを確認します。
    - **Sid**: gMSA アカウントの SID
    - **MachineAccountName**: gMSA SAM アカウント名 (完全なドメイン名やドル記号は含めないでください)
    - **DnsTreeName**: Active Directory フォレストの FQDN
    - **DnsName**: gMSA が属するドメインの FQDN
    - **NetBiosName**:gMSA が属するドメインの NETBIOS 名
    - **GroupManagedServiceAccounts/Name**: gMSA SAM アカウント名 (完全なドメイン名やドル記号は含めないでください)
    - **GroupManagedServiceAccounts/Scope**: ドメイン FQDN の 1 つのエントリと NETBIOS の 1 つのエントリ

    入力は、次の完全な資格情報の指定例のようになります。

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. 資格情報の指定ファイルのパスがオーケストレーション ソリューションに対して正しいことを確認します。 Docker を使用している場合は、コンテナー実行コマンドに `--security-opt="credentialspec=file://NAME.json"` が含まれていることを確認します。この "NAME.json" は、**Get-CredentialSpec** によって出力された名前に置き換えられます。 名前は、Docker ルート ディレクトリ以下の CredentialSpecs フォルダーを基準とした相対的なフラット ファイル名です。

### <a name="check-the-firewall-configuration"></a>ファイアウォールの構成を確認する

コンテナーまたはホスト ネットワークで厳格なファイアウォール ポリシーを使用している場合、Active Directory ドメイン コントローラーまたは DNS サーバーへの必要な接続がブロックされる可能性があります。

| プロトコルとポート | 目的 |
|-------------------|---------|
| TCP と UDP 53 | DNS |
| TCP と UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP と UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

コンテナーからドメイン コントローラーに送信するトラフィックの種類によっては、必要に応じて追加のポートへのアクセスを許可します。
Active Directory で使用されるポートの完全な一覧については、「[Active Directory および Active Directory Domain Services のポート要件](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)」を参照してください。

### <a name="check-the-container"></a>コンテナーを確認する

1. Windows Server 2019 または Windows 10 バージョン 1809 より前のバージョンの Windows を実行している場合、コンテナーのホスト名は gMSA 名と一致する必要があります。 `--hostname` パラメーターが gMSA の短い名前 (ドメイン コンポーネントなし、たとえば、"webapp01.contoso.com" ではなく "webapp01") と一致していることを確認します。

2. コンテナーのネットワーク構成をチェックして、gMSA のドメインのドメイン コントローラーをコンテナーで解決してアクセスできることを確認します。 コンテナー内の正しく構成されていない DNS サーバーは、ID の問題の一般的な原因です。

3. コンテナーで次のコマンドレットを実行して (`docker exec` または同等のものを使用して)、コンテナーにドメインへの有効な接続があるかどうかを確認します。

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    gMSA が使用可能であり、ネットワーク接続によってコンテナーがドメインと通信できる場合、信頼性の検証によって `NERR_SUCCESS` が返されます。 失敗した場合は、ホストとコンテナーのネットワーク構成を確認します。 どちらもドメイン コントローラーと通信できる必要があります。

4. コンテナーが有効な Kerberos チケット保証チケット (TGT) を取得できるかどうかを確認します。

    ```powershell
    klist get krbtgt
    ```

    このコマンドを実行すると、"A ticket to krbtgt has been retrieved successfully" (krbtgt へのチケットが正常に取得されました) が返され、チケットの取得に使用されたドメイン コントローラーが一覧表示されます。 TGT を取得できても、前の手順の `nltest` が失敗する場合は、gMSA アカウントが正しく構成されていない可能性があります。 詳細については、「[gMSA アカウントを確認する](#check-the-gmsa-account)」を参照してください。

    コンテナー内の TGT を取得できない場合は、DNS またはネットワーク接続に問題がある可能性があります。 コンテナーでドメイン DNS 名を使用してドメイン コントローラーを解決できることと、ドメイン コントローラーがコンテナーからルーティング可能であることを確認します。

5. [gMSA](gmsa-configure-app.md) を使用するようにアプリが構成されていることを確認します。 gMSA を使用しても、コンテナー内のユーザー アカウントは変更されません。 そうではなく、他のネットワーク リソースと通信するときに、System アカウントに gMSA が使用されます。 つまり、gMSA ID を利用するには、Network Service または Local System としてアプリを実行する必要があります。

    > [!TIP]
    > `whoami` を実行するか、別のツールを使用してコンテナー内の現在のユーザー コンテキストを識別する場合、gMSA の名前自体は表示されません。 これは、ドメイン ID ではなくローカル ユーザーとして常にコンテナーにサインインするためです。 コンピューター アカウントがネットワーク リソースと通信するときは常に gMSA が使用されるため、アプリを Network Service または Local System として実行する必要があります。

### <a name="check-the-gmsa-account"></a>gMSA アカウントを確認する

1. コンテナーが正しく構成されているように見えても、ユーザーまたは他のサービスがコンテナー化されたアプリに対して自動的に認証できない場合は、gMSA アカウントの SPN を確認します。 クライアントでは、アプリケーションに到達したときの名前で gMSA アカウントが検索されます。 つまり、たとえば、ロード バランサーまたは別の DNS 名を介してクライアントからアプリに接続する場合、gMSA に追加の `host` SPN が必要になる可能性があります。

2. gMSA とコンテナー ホストが同じ Active Directory ドメインに属していることを確認します。 gMSA が別のドメインに属している場合、コンテナー ホストから gMSA のパスワードを取得できません。

3. ドメインに gMSA と同じ名前のアカウントが 1 つだけ存在することを確認します。 gMSA オブジェクトの SAM アカウント名にはドル記号 ($) が追加されているため、同じドメイン内で gMSA に "myaccount$" という名前を付け、無関係なユーザー アカウントに "myaccount" という名前を付けることができます。 これにより、ドメイン コントローラーまたはアプリケーションが gMSA を名前で検索する必要がある場合に、問題が発生する可能性があります。 次のコマンドを使用して、似た名前のオブジェクトを AD で検索できます。

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. gMSA アカウントで制約のない委任を有効にしている場合は、[UserAccountControl 属性](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)の `WORKSTATION_TRUST_ACCOUNT` フラグが有効であることを確認します。 アプリで名前を SID に、またはその逆に解決する必要がある場合と同様に、このフラグは、コンテナー内の NETLOGON がドメイン コントローラーと通信するために必要です。 次のコマンドを使用して、フラグが正しく構成されているかどうかを確認できます。

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    上記のコマンドから `False` を返される場合は、次を使用して `WORKSTATION_TRUST_ACCOUNT` フラグを gMSA アカウントの UserAccountControl プロパティに追加します。 このコマンドを実行すると、UserAccountControl プロパティから `NORMAL_ACCOUNT`、`INTERDOMAIN_TRUST_ACCOUNT`、および `SERVER_TRUST_ACCOUNT` フラグもクリアされます。

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
