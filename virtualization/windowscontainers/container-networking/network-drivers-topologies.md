---
title: Windows コンテナーネットワークドライバー
description: ネットワーク ドライバーと Windows コンテナーのトポロジ。
keywords: Docker, コンテナー
author: daschott
ms.date: 08/13/2020
ms.topic: conceptual
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: d28b1cf2bd8295001b26b821c1269efb1f7b4070
ms.sourcegitcommit: ecbee9197074eeb0ae612b28fcf1144e28bf9789
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/25/2020
ms.locfileid: "88810122"
---
# <a name="windows-container-network-drivers"></a>Windows コンテナーネットワークドライバー

Windows で Docker によって作成された既定の 'nat' ネットワークを活用することに加えて、ユーザーはカスタム コンテナー ネットワークを定義できます。 ユーザー定義ネットワークは、Docker CLI コマンドを使用して作成でき [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) ます。 Windows では、次の種類のネットワーク ドライバーを利用できます。


### <a name="nat-network-driver"></a>NAT ネットワークドライバー

"Nat" ドライバーを使用して作成されたネットワークに接続されているコンテナーは、 *内部* hyper-v スイッチに接続され、ユーザー指定の () ip プレフィックスから ip アドレスを受け取り ``--subnet`` ます。 コンテナー ホストからコンテナー エンドポイントへのポート フォワーディングおよびマッピングがサポートされています。

  >[!TIP]
  > 既定の "nat" ネットワークで使用されるサブネットは、 `fixed-cidr` [Docker デーモン構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)の設定を使用してカスタマイズできます。

  >[!NOTE]
  > Windows Server 2019 (またはそれ以降) で作成された NAT ネットワークは、再起動後に保持されなくなります。

##### <a name="creating-a-nat-network"></a>NAT ネットワークの作成

サブネットを含む新しい NAT ネットワークを作成するには、次のようにし `10.244.0.0/24` ます。
```powershell
docker network create -d "nat" --subnet "10.244.0.0/24" my_nat
```


### <a name="transparent-network-driver"></a>透過的なネットワークドライバー

"Transparent" ドライバーを使用して作成されたネットワークに接続されたコンテナーは、 *外部* hyper-v スイッチによって物理ネットワークに直接接続されます。 物理ネットワークの IP は、静的に割り当てることも (ユーザー指定の ``--subnet`` オプションが必要)、外部の DHCP サーバーを使用して動的に割り当てることもできます。

  >[!NOTE]
  >次の要件により、Azure Vm では、透過ネットワーク経由でのコンテナーホストの接続はサポートされていません。

  > が必要: このモードが仮想化シナリオで使用されている場合 (コンテナーホストが VM の場合)、 _MAC アドレスのスプーフィングが必要_です。

##### <a name="creating-a-transparent-network"></a>透過的なネットワークを作成する

サブネット `10.244.0.0/24` 、ゲートウェイ `10.244.0.1` 、DNS サーバー `10.244.0.7` 、および VLAN ID を持つ新しい透過ネットワークを作成するには、次のようにし `7` ます。
```powershell
docker network create -d "transparent" --subnet 10.244.0.0/24 --gateway 10.244.0.1 -o com.docker.network.windowsshim.vlanid=7 -o com.docker.network.windowsshim.dnsservers="10.244.0.7" my_transparent
```


### <a name="overlay-network-driver"></a>ネットワークドライバーのオーバーレイ 

よくは、Docker の群れや Kubernetes などのコンテナーによって使用されます。オーバーレイネットワークに接続されているコンテナーは、複数のコンテナーホスト間で同じネットワークに接続されている他のコンテナーと通信できます。 各オーバーレイネットワークは、プライベート IP プレフィックスで定義された独自の IP サブネットを使用して作成されます。 オーバーレイネットワークドライバーは VXLAN カプセル化を使用して、テナントコンテナーネットワーク間のネットワークトラフィックの分離を実現し、オーバーレイネットワーク間での IP アドレスの再利用を可能にします。
  > 必要条件: オーバーレイネットワークを作成するために必要な [前提条件](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) を環境が満たしていることを確認します。

  > が必要: Windows Server 2019 では、 [KB4489899](https://support.microsoft.com/help/4489899)が必要です。

  > が必要: Windows Server 2016 では、 [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)が必要です。

  >[!NOTE]
  >Windows Server 2019 以降では、Docker 群れによって作成されたオーバーレイネットワークは、送信接続用の VFP NAT ルールを利用します。 これは、特定のコンテナーが1つの IP アドレスを受け取ることを意味します。 また、やなどの ICMP ベースのツール `ping` は、 `Test-NetConnection` デバッグ状況で TCP/UDP オプションを使用して構成する必要があります。

##### <a name="creating-a-overlay-network"></a>オーバーレイネットワークの作成

サブネット `10.244.0.0/24` 、DNS サーバー、および VSID を使用して新しいオーバーレイネットワークを作成するには、次のように `168.63.129.16` し `4096` ます。
```powershell
docker network create -d "overlay" --attachable --subnet "10.244.0.0/24" -o com.docker.network.windowsshim.dnsservers="168.63.129.16" -o com.docker.network.driver.overlay.vxlanid_list="4096" my_overlay
```


### <a name="l2bridge-network-driver"></a>L2bridge ネットワークドライバー

"L2bridge" ドライバーを使用して作成されたネットワークに接続されているコンテナーは、 *外部* hyper-v スイッチを介して物理ネットワークに接続されます。 L2bridge では、コンテナーのネットワークトラフィックは、受信時と送信時のレイヤー2アドレス変換 (MAC の再書き込み) 操作により、ホストと同じ MAC アドレスを持ちます。 データセンターでは、これにより、有効期間が短いコンテナーの MAC アドレスを学習する必要があるスイッチのストレスが軽減されます。 L2bridge ネットワークは、次の2つの異なる方法で構成できます。
  1. L2bridge ネットワークは、コンテナーホストと同じ IP サブネットを使用して構成されています
  2. L2bridge ネットワークが新しいカスタム IP サブネットで構成されています

  構成2では、ゲートウェイとして機能するホストネットワークコンパートメントにエンドポイントを追加し、指定されたプレフィックスのルーティング機能を構成する必要があります。

##### <a name="creating-a-l2bridge-network"></a>L2bridge ネットワークの作成

サブネット `10.244.0.0/24` 、ゲートウェイ `10.244.0.1` 、DNS サーバー、 `10.244.0.7` および VLAN ID 7 を使用して新しい l2bridge ネットワークを作成するには、次のようにします。
```powershell
docker network create -d "l2bridge" --subnet 10.244.0.0/24 --gateway 10.244.0.1 -o com.docker.network.windowsshim.vlanid=7 -o com.docker.network.windowsshim.dnsservers="10.244.0.7" my_transparent
```

  >[!TIP]
  >L2bridge ネットワークは高度なプログラミングが可能です。L2bridge を構成する方法の詳細については、 [こちら](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923)を参照してください。


### <a name="l2tunnel-network-driver"></a>L2tunnel ネットワークドライバー

作成は l2bridge と同じですが、 _このドライバーは Microsoft Cloud スタック (Azure) でのみ使用する必要があり_ます。 L2bridge の唯一の違いは、すべてのコンテナートラフィックが SDN ポリシーが適用されている仮想化ホストに送信されることです。これにより、コンテナーの [Azure ネットワークセキュリティグループ](/azure/virtual-network/security-overview) などの機能が有効になります。


## <a name="network-topologies-and-ipam"></a>ネットワークトポロジと IPAM

次の表に、各ネットワーク ドライバーの内部 (コンテナー間) および外部接続で、ネットワーク接続がどのように提供されるかを示します。

### <a name="networking-modesdocker-drivers"></a>ネットワークモード/Docker ドライバー

  | Docker Windows ネットワーク ドライバー | 標準的な使用 | コンテナーからコンテナー (単一ノード) | コンテナーから外部 (単一ノード + マルチノード) | コンテナーとコンテナー間 (マルチノード) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (既定)** | 開発者向け | <ul><li>同一サブネット: Hyper-V 仮想スイッチを介したブリッジ接続</li><li> クロスサブネット: サポートされていません (NAT 内部プレフィックスが1つのみ)</li></ul> | 管理 vNIC (WinNAT にバインド) を経由してルーティング | 直接サポート外: ホストを経由してポートを公開する必要があります |
  | **Transparent** | 開発者または小規模な開発向け | <ul><li>同一サブネット: Hyper-V 仮想スイッチを介したブリッジ接続</li><li>クロス サブネット: コンテナー ホストを経由してルーティング</li></ul> | (物理) ネットワーク アダプターへの直接アクセスでコンテナー ホストを経由してルーティング | (物理) ネットワーク アダプターへの直接アクセスでコンテナー ホストを経由してルーティング |
  | **合わせる** | マルチノードに適しています。Kubernetes で利用可能な Docker の群れに必須 | <ul><li>同一サブネット: Hyper-V 仮想スイッチを介したブリッジ接続</li><li>クロス サブネット: ネットワーク トラフィックをカプセル化し、Mgmt vNIC を経由してルーティング</li></ul> | 直接サポートされない-windows server 2016 の NAT ネットワークまたは Windows Server 2019 の VFP NAT 規則に接続されている2番目のコンテナーエンドポイントが必要です。  | 同一/クロス サブネット: VXLAN を使用してネットワーク トラフィックをカプセル化し、Mgmt vNIC を経由してルーティング |
  | **L2Bridge** | Kubernetes および Microsoft SDN に使用 | <ul><li>同一サブネット: Hyper-V 仮想スイッチを介したブリッジ接続</li><li> クロス サブネット: コンテナーの MAC アドレスを入口と出口で書き換えてルーティング</li></ul> | コンテナーの MAC アドレスを入口と出口で書き換え | <ul><li>同一サブネット: ブリッジ接続</li><li>クロスサブネット: WSv1809 以上での管理 vNIC によるルーティング</li></ul> |
  | **L2Tunnel**| Azure のみ | 同一/クロス サブネット: ポリシーが適用される物理ホストの Hyper-V 仮想スイッチに折り返し | トラフィックは Azure の仮想ネットワーク ゲートウェイを経由する必要があります | 同一/クロス サブネット: ポリシーが適用される物理ホストの Hyper-V 仮想スイッチに折り返し |

### <a name="ipam"></a>IPAM

各ネットワーク ドライバーで、IP アドレスの割り当ては異なります。 Windows では、ホスト ネットワーク サービス (HNS) を使用して nat ドライバーに IPAM を提供し、Docker の Swarm モード (内部 KVS) と連携して overlay に IPAM を提供します。 その他のすべてのネットワーク ドライバーは、外部の IPAM を使用します。

| ネットワーク モード/ドライバー | IPAM |
| -------------------------|:----:|
| NAT | 内部 NAT サブネットプレフィックスからのホストネットワークサービス (HNS) による動的 IP の割り当てと割り当て |
| 透明 | コンテナー ホストのネットワーク プレフィックス内で IP アドレスから静的または動的 (外部 DHCP サーバーを使用して) IP 割り当て/設定 |
| オーバーレイ | Docker エンジン Swarm モードで管理されるプレフィックスからの動的 IP 割り当てと HNS による設定 |
| L2Bridge | 指定されたサブネットプレフィックスからのホストネットワークサービス (HNS) による動的 IP の割り当てと割り当て |
| L2Tunnel | Azure のみ: プラグインから動的 IP 割り当て/設定 |

### <a name="service-discovery"></a>サービス探索

サービスの検出は、特定の Windows ネットワーク ドライバーについてのみサポートされます。

| ドライバー名 | ローカル サービス検出  | グローバル サービス検出 |
| :--- | :---------------     |  :---                |
| nat | YES | Docker EE で使用可能 |
| overlay | YES | はい (Docker EE または kube) |
| transparent | NO | NO |
| l2bridge | はい (kube を使用) | はい (kube を使用) |
