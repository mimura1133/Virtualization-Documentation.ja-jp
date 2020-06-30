---
title: Windows Docker ホストのリモート管理
description: Windows Server を実行しているリモート Docker ホストを安全に管理する方法。
keywords: Docker, コンテナー
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: how-to
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
ms.openlocfilehash: 8739b257dbd5f0681d990e5dc47235325740f849
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192379"
---
# <a name="remote-management-of-a-windows-docker-host"></a>Windows Docker ホストのリモート管理

`docker-machine` がない場合でも、リモートにアクセス可能な Docker ホストを Windows Server 2016 VM 上に作成できます。

手順は非常に簡単です。

* [dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/) を使って、サーバー上で証明書を作成します。
IP アドレスに対する証明書を作成する場合は、IP アドレスが変わったときに証明書を再作成しなくて済むように静的 IP の使用を検討することをお勧めします。

* Docker サービス `Restart-Service Docker` を再起動します。
* 着信トラフィックを許可する NSG 規則を作成して、port docker の TLS ポート 2375 と 2376 を利用可能に設定します。 セキュリティで保護された接続には、2376 の許可のみが必要です。
  ポータルは、次のように、NSG 構成を表示します。![NSG](media/nsg.png)

* Windows ファイアウォール経由の着信接続を許可します。
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* コンピューター上にあるユーザーの docker フォルダー (例: `c:\users\chris\.docker`) からファイル `ca.pem`、'cert.pem'、'key.pem' をローカル コンピューターにコピーします。 たとえば、RDP セッションから、Ctrl + C キーと Ctrl + V キーを使ってファイルをコピーできます。
* リモート Docker ホストに接続できることを確認します。 WMIMgmt.msc
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## <a name="troubleshooting"></a>トラブルシューティング
### <a name="try-connecting-without-tls-to-determine-your-nsg-firewall-settings-are-correct"></a>TLS を使わずに接続を試みて、NSG ファイアウォール設定が正しいことを確認します。
接続エラーは、通常、次のように表示されます。
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

暗号化されていない接続を許可します。これには、
```
{
    "tlsverify":  false,
}
```
を `c"\programdata\docker\config\daemon.json` に追加し、サービスを再起動します。

次のようなコマンド ラインを使ってリモート ホストに接続します。
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### <a name="cert-problems"></a>証明書の問題
IP アドレスまたは DNS 名に対して作成されていない証明書を使って Docker ホストにアクセスすると、次のエラーが発生します。
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
ホストのパブリック IP の DNS 名が w.x.y.z であること、および DNS 名が証明書の[コモン ネーム](https://www.ssl.com/faqs/common-name/) (`SERVER_NAME` 環境変数または dockertls に指定された `IP_ADDRESSES` 変数のいずれかの IP アドレス) と一致していることを確認します。

### <a name="cryptox509-warning"></a>crypto/x509 の警告
次の警告が表示されることがあります。
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
この警告は特に問題ありません。
