---
title: ネットワーク トポロジ
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
description: Windows および Linux でサポートされているネットワークトポロジ。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 5eeee17dc6dfc87357d80c8b8fd7a29f05fc4c35
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985296"
---
# <a name="network-solutions"></a>Network Solutions #

[Kubernetes マスターノードを設定](./creating-a-linux-master.md)したら、ネットワークソリューションを選択できます。 仮想[クラスターサブネット](./getting-started-kubernetes-windows.md#cluster-subnet-def)を複数のノードにルーティングできるようにするには、いくつかの方法があります。 現在、Windows on Windows では、次のいずれかのオプションを選択します。 Kubernetes

1. [Flannel](#flannel-in-vxlan-mode)などの CNI プラグインを使用して、オーバーレイネットワークをセットアップします。
2. [Flannel](#flannel-in-host-gateway-mode)などの CNI プラグインを使用してルートをプログラムします (l2bridge ネットワークモードを使用します)。
3. サブネットをルーティングするスマート[トップラック (ToR) スイッチ](#configuring-a-tor-switch)を構成します。

> [!tip]
> Windows には4つ目のネットワークソリューションがあります。このソリューションでは、Open vSwitch (OvS) と Open Virtual Network ([]) が利用されています。 このドキュメントの内容は記載されていませんが、[これらの手順](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)を読んで設定することができます。

## <a name="flannel-in-vxlan-mode"></a>Vxlan モードの Flannel

Vxlan モードの Flannel を使用すると、構成可能な仮想オーバーレイネットワークを設定できます。このネットワークでは、VXLAN トンネリングを使用してノード間でパケットをルーティングします。

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel の Kubernetes マスターを準備する
クラスターの[Kubernetes マスター](./creating-a-linux-master.md)で多少の準備を行うことをお勧めします。 Flannel を使用する場合は、iptables チェーンにブリッジされた IPv4 トラフィックを有効にすることをお勧めします。 これは、次のコマンドを使用して行うことができます。

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Flannel をダウンロード & 構成する ###
最新の Flannel マニフェストをダウンロードします。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Vxlan ネットワークバックエンドを有効にするには、次の2つのセクションを変更する必要があります。

1. `net-conf.json`のセクションで `kube-flannel.yml` 、次のようにダブルチェックします。
 * クラスターサブネット (例: "10.244.0.0/16") は、必要に応じて設定されます。
 * VNI 4096 はバックエンドで設定されています
 * ポート4789はバックエンドで設定されています
2. `cni-conf.json`のセクションで `kube-flannel.yml` 、ネットワーク名をに変更 `"vxlan0"` します。

上記の手順を適用すると、は次のようになり `net-conf.json` ます。
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]
> Linux で Flannel を Flannel と相互運用するには、VNI を4096に設定し、ポート4789を使用する必要があります。 他の VNIs のサポートは近日対応予定です。 これらのフィールドの詳細については、「 [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 」を参照してください。

は次のようになり `cni-conf.json` ます。
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]
> 上記のオプションの詳細については、Linux 用の公式の CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、[ポートマップ](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)、[ブリッジ](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)プラグインに関するドキュメントを参照してください。

### <a name="launch-flannel--validate"></a>Flannel の起動 & 検証 ###
次を使用して Flannel を起動します。

```bash
kubectl apply -f kube-flannel.yml
```

次に、Flannel ポッドが Linux ベースであるため、linux の[Nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)パッチを DaemonSet に適用して、linux `kube-flannel-ds` のみを対象にします (後で Windows で Flannel "flanneld" を起動します)。

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> いずれかのノードが x86-64 ベースでない場合は、 `-amd64` 上のプロセッサアーキテクチャに置き換えます。

数分後に、Flannel ポッドネットワークがデプロイされている場合は、すべてのポッドが実行中であることがわかります。

```bash
kubectl get pods --all-namespaces
```

![text](media/kube-master.png)

Flannel DaemonSet にも NodeSelector が適用されている必要があり `beta.kubernetes.io/os=linux` ます。

```bash
kubectl get ds -n kube-system
```

![text](media/kube-daemonset.png)

> [!tip]
> 残りの flannel-* デーモンセットでは、そのプロセッサアーキテクチャに一致するノードがない場合にはスケジュールされないため、これらは無視または削除できます。

> [!tip]
> 間違え? Flannel v 0.11.0 の完全な例を次に示します。これらの手順は、既定のクラスターサブネットに事前に適用されています[。 .yml kube-flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) . `10.244.0.0/16`

正常に完了したら、[次の手順](#next-steps)に進みます。

## <a name="flannel-in-host-gateway-mode"></a>ホスト-ゲートウェイモードでの Flannel

[Flannel vxlan](#flannel-in-vxlan-mode)と共に、Flannel ネットワークのもう1つのオプションは*ホストゲートウェイモード*(ホスト gw) です。これにより、ターゲットノードのホストアドレスを次ホップとして使用して、各ノードの静的ルートを他のノードのポッドサブネットにプログラミングすることができます。

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel の Kubernetes マスターを準備する

クラスターの[Kubernetes マスター](./creating-a-linux-master.md)で多少の準備を行うことをお勧めします。 Flannel を使用する場合は、iptables チェーンにブリッジされた IPv4 トラフィックを有効にすることをお勧めします。 これは、次のコマンドを使用して行うことができます。

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Flannel をダウンロード & 構成する ###
最新の Flannel マニフェストをダウンロードします。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Windows/Linux の両方でホスト gw ネットワークを有効にするために、変更する必要があるファイルが1つあります。

`net-conf.json`Kube-flannel の .yml のセクションで、次のことを確認します。
1. 使用されているネットワークバックエンドの種類は、ではなくに設定され `host-gw` `vxlan` ます。
2. クラスターサブネット (例: "10.244.0.0/16") は、必要に応じて設定されます。

2つの手順を適用すると、は次のようになり `net-conf.json` ます。
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Flannel の起動 & 検証 ###
次を使用して Flannel を起動します。

```bash
kubectl apply -f kube-flannel.yml
```

次に、Flannel ポッドが Linux ベースであるため、linux [Nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)パッチを DaemonSet に適用して `kube-flannel-ds` linux のみを対象にします (参加時に Windows で Flannel "flanneld" ホストエージェントプロセスを起動します)。

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> いずれかのノードが x86-64 ベースでない場合は、 `-amd64` 上の手順を目的のプロセッサアーキテクチャに置き換えます。

数分後に、Flannel ポッドネットワークがデプロイされている場合は、すべてのポッドが実行中であることがわかります。

```bash
kubectl get pods --all-namespaces
```

![text](media/kube-master.png)

Flannel DaemonSet にも NodeSelector が適用されている必要があります。

```bash
kubectl get ds -n kube-system
```

![text](media/kube-daemonset.png)

> [!tip]
> 残りの flannel-* デーモンセットでは、そのプロセッサアーキテクチャに一致するノードがない場合にはスケジュールされないため、これらは無視または削除できます。

> [!tip]
> 間違え? Flannel v 0.11.0 の完全な例を次に示します。この2つの手順は、既定のクラスターサブネットに事前に適用されています。 [.yml kube-flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) . `10.244.0.0/16`

正常に完了したら、[次の手順](#next-steps)に進みます。

## <a name="configuring-a-tor-switch"></a>ToR スイッチの構成 ##
> [!NOTE]
> [ネットワークソリューションとして Flannel](#flannel-in-host-gateway-mode)を選択した場合は、このセクションをスキップできます。
ToR スイッチの構成は、実際のノードの外部で行われます。 詳細については、公式の[Kubernetes ドキュメント](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)を参照してください。


## <a name="next-steps"></a>次のステップ ##
このセクションでは、ネットワークソリューションを選択して構成する方法について説明します。 これで、手順4の準備ができました。

> [!div class="nextstepaction"]
> [Windows ワーカーの参加](./joining-windows-workers.md)