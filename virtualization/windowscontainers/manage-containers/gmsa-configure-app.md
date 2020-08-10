---
title: グループ管理サービス アカウントを使用するようにアプリを構成する
description: Windows コンテナーでグループ管理サービス アカウント (gMSA) を使用するようにアプリを構成する方法。
keywords: docker、コンテナー、active directory、gmsa、アプリ、アプリケーション、グループ管理サービス アカウント、グループ管理サービス アカウント、構成
author: rpsqrd
ms.date: 09/10/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: fa14f2fa953373f8dfea74f822b025be0910fd54
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985266"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>gMSA を使用するようにアプリを構成する

一般的な構成では、コンテナーには 1 つのグループ管理サービス アカウント (gMSA) のみが与えられ、コンテナー コンピューター アカウントからネットワーク リソースへの認証が試行されるたびにそれが使用されます。 つまり、gMSA ID を使用する必要がある場合、**Local System** または **Network Service** としてアプリを実行する必要があります。

## <a name="run-an-iis-app-pool-as-network-service"></a>IIS アプリ プールを Network Service として実行する

コンテナーで IIS Web サイトをホストしている場合、gMSA を利用するために必要なことは、アプリプール ID を **Network Service** に設定することだけです。 これを Dockerfile で実行するには、次のコマンドを追加します。

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

以前に IIS アプリプールに静的ユーザー資格情報を使用したことがある場合は、それらの資格情報の代わりに gMSA を検討してください。 開発環境、テスト環境、運用環境で gMSA を変更できます。コンテナー イメージを変更することなく、IIS によって現在の ID が自動的に取得されます。

## <a name="run-a-windows-service-as-network-service"></a>Windows サービスを Network Service として実行する

コンテナー化されたアプリを Windows サービスとして実行する場合は、Dockerfile で **Network Service** として実行するようにサービスを設定できます。

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>任意のコンソール アプリを Network Service として実行する

IIS または Service Manager でホストされていない汎用コンソール アプリでは、多くの場合、コンテナーを **Network Service** として実行することが最も簡単です。アプリによって自動的に gMSA コンテキストが継承されるためです。 この機能は、Windows Server バージョン 1709 以降で使用できます。

次の行を Dockerfile に追加して、既定で Network Service として実行するようにします。

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

`docker exec` を使用して、Network Service としてコンテナーに 1 回限り接続することもできます。 これは、通常はコンテナーが Network Service として実行されていないときに、実行中のコンテナーの接続の問題をトラブルシューティングする場合に特に役立ちます。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>次の手順

gMSA を使用すると、アプリの構成だけでなく、次のことを行うことができます。

- [コンテナーを実行する](gmsa-run-container.md)
- [コンテナーを調整する](gmsa-orchestrate-containers.md)

セットアップ中に問題が発生した場合は、[トラブルシューティング ガイド](gmsa-troubleshooting.md)で考えられる解決策をご確認ください。
