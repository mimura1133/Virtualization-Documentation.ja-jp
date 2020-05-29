---
title: gMSA を使用してコンテナーを実行する
description: グループ管理サービス アカウント (gMSA) を使用して Windows コンテナーを実行する方法。
keywords: docker, コンテナー, active directory, gmsa, グループ管理サービス アカウント, グループ管理サービス アカウント
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b997cf79cdf7f1782b6299198859714563c45f8c
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853936"
---
# <a name="run-a-container-with-a-gmsa"></a>gMSA を使用してコンテナーを実行する

グループ管理サービス アカウント (gMSA) でコンテナーを実行するには、[docker run](https://docs.docker.com/engine/reference/run) の `--security-opt` パラメーターに資格情報の指定ファイルを指定します。

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Windows Server 2016 バージョン 1709 および 1803 では、コンテナーのホスト名は gMSA の短い名前と一致する必要があります。

前の例では、gMSA SAM アカウント名は "webapp01" であるため、コンテナーのホスト名も "webapp01" という名前になります。

Windows Server 2019 以降では、ホスト名フィールドは必須ではありませんが、明示的に別の値を指定した場合でも、コンテナーではホスト名ではなく gMSA 名で自身が識別されます。

gMSA が正しく動作しているかどうかを確認するには、コンテナーで次のコマンドレットを実行します。

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Trusted DC Connection Status と Trust Verification Status が `NERR_Success` でない場合は、[トラブルシューティング手順](gmsa-troubleshooting.md#check-the-container)に従って問題をデバッグします。

コンテナー内から gMSA ID を確認するには、次のコマンドを実行してクライアント名を確認します。

```powershell
PS C:\> klist get webapp01

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

PowerShell または別のコンソール アプリを gMSA アカウントとして開くには、通常の ContainerAdministrator (NanoServer 場合は ContainerUser) アカウントではなく、Network Service アカウントで実行するようにコンテナーに要求できます。

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Network Service として実行している場合は、gMSA としてネットワーク認証をテストするには、ドメイン コントローラー上の SYSVOL に接続します。

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>次の手順

コンテナーの実行に加えて、gMSA を使用して次のことを行うこともできます。

- [アプリを構成する](gmsa-configure-app.md)
- [コンテナーを調整する](gmsa-orchestrate-containers.md)

設定中に問題が発生した場合は、[トラブルシューティング ガイド](gmsa-troubleshooting.md)で考えられる解決策をご確認ください。
