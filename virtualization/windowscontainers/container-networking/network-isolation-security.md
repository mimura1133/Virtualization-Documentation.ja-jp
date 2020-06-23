---
title: Windows コンテナー ネットワーク
description: Windows コンテナー内のネットワーク分離とセキュリティ。
keywords: Docker, コンテナー
author: jmesser81
ms.date: 03/27/2018
ms.topic: conceptual
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 78f9240ccb184b182247617aba116d6ac5533a02
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192089"
---
# <a name="network-isolation-and-security"></a>ネットワーク分離とセキュリティ

## <a name="isolation-with-network-namespaces"></a>ネットワーク名前空間を使用した分離

各コンテナーのエンドポイントは、自身の__ネットワーク名前空間__内に置かれます。 管理ホストの vNIC とホスト ネットワーク スタックは、既定のネットワーク名前空間に置かれます。 同じホスト上のコンテナー間でネットワークの分離を強制するために、Windows Server のコンテナーごとにネットワーク名前空間が作成され、コンテナーのネットワークアダプターがインストールされている Hyper-v 分離の下で実行されます。 Windows Server コンテナーでは、仮想スイッチへの接続に Host vNIC を使用します。 Hyper-v の分離では、仮想スイッチにアタッチするために、統合 VM NIC (ユーティリティ VM には公開されません) を使用します。

![テキスト](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>ネットワークのセキュリティ

使用されているコンテナーとネットワーク ドライバーによっては、Windows ファイアウォールと [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/) の組み合わせでポート ACL が適用されます。

### <a name="windows-server-containers"></a>Windows Server コンテナー

これらは、Windows ホストのファイアウォール (ネットワークの名前空間に対応) と共に VFP を使用します。

* 既定の発信: ALLOW ALL
* 既定の着信: (TCP、UDP、ICMP、IGMP の) 未承諾ネットワーク トラフィックに対して ALLOW ALL 
  * これらのプロトコル以外のネットワーク トラフィックに対して DENY ALL

  >[!NOTE]
  >Windows Server より前のバージョンでは、バージョン1709と Windows 10 は、既定の受信規則はすべて拒否されていました。 これらの古いリリースを実行しているユーザーは、(ポートフォワーディング) を使用して受信許可規則を作成でき ``docker run -p`` ます。

### <a name="hyper-v-isolation"></a>Hyper-V による分離

Hyper-v の分離で実行されるコンテナーには、独自の分離されたカーネルがあるため、次の構成を使用して、独自の Windows ファイアウォールのインスタンスを実行します。

* Windows ファイアウォール (ユーティリティ VM 内で実行) と VFP の両方で既定 ALLOW ALL

![テキスト](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes ポッド

[Kubernetes ポッド](https://kubernetes.io/docs/concepts/workloads/pods/pod/)では、エンドポイントがアタッチされるインフラストラクチャコンテナーが最初に作成されます。 インフラストラクチャやワーカーコンテナーなど、同じポッドに属するコンテナーは、共通のネットワーク名前空間 (同じ IP およびポート空間) を共有します。

![テキスト](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>既定のポート ACL のカスタマイズ

既定のポート Acl を変更する場合は、最初にホストネットワークサービスのドキュメントを参照してください (まもなく追加されるリンク)。 次のコンポーネント内のポリシーを更新する必要があります。

>[!NOTE]
>透過的モードと NAT モードでの Hyper-v の分離では、現在、既定のポート Acl のプログラミングを行うことはできません。 このことは、次の表の中で "X" として示されています。

| ネットワークドライバー | Windows Server コンテナー | Hyper-V による分離  |
| -------------- |-------------------------- | ------------------- |
| 透明 | Windows ファイアウォール | X |
| NAT | Windows ファイアウォール | X |
| L2Bridge | 両方 | VFP |
| L2Tunnel | 両方 | VFP |
| オーバーレイ  | 両方 | VFP |