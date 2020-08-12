---
title: Windows サービスとしての Kubernetes の実行
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: how-to
description: Kubernetes コンポーネントを Windows サービスとして実行する方法について説明します。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: caf364045064744a6c3175047ab5bb77b34b742a
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161782"
---
# <a name="kubernetes-components-as-windows-services"></a>Windows サービスとしての Kubernetes コンポーネント

flanneld.exe、kubelet.exe、kube-proxy.exe などのプロセスを Windows サービスとして実行するように構成することもできます。 これにより、プロセスが予期しないプロセスまたはノードクラッシュに応じて自動的に再起動するなど、追加のフォールトトレランスの利点がもたらされます。


## <a name="prerequisites"></a>前提条件
1. [nssm.exe](https://nssm.cc/download)をディレクトリにダウンロードしました `c:\k`
2. ノードをクラスターに参加させ、前のノードで [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) または [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) スクリプトを実行します。

## <a name="registering-windows-services"></a>Windows サービスの登録
、、を[a sample script](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1)登録し `kubelet` `kube-proxy` `flanneld.exe` てバックグラウンドで Windows サービスとして実行する nssm.exe を使用するサンプルスクリプトを実行できます。

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementip"></a>[ManagementIP](#tab/ManagementIP)
Windows ノードに割り当てられた IP アドレス。 これは、を使用して見つけることができ `ipconfig` ます。

| パラメーター | 既定値 |
|---|---|
| `-ManagementIP` | n.A. |


# <a name="networkmode"></a>[NetworkMode](#tab/NetworkMode)
ネットワーク `l2bridge` ソリューションとして選択されたネットワークモード (flannel ホスト gw) または `overlay` (flannel vxlan)。 [network solution](./network-topologies.md)

> [!Important]
> `overlay` ネットワークモード (flannel vxlan) には、Kubernetes v 1.14 バイナリ以上が必要です。

| パラメーター | 既定値 |
|---|---|
| `-NetworkMode` | `12bridge` |

# <a name="clustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
[クラスターサブネットの範囲](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

| パラメーター | 既定値 |
|---|---|
| `-ClusterCIDR` | `10.244.0.0/16` |

# <a name="kubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS サービス IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster)。

| パラメーター | 既定値 |
|---|---|
| `-KubeDnsServiceIP` | `10.96.0.10` |

# <a name="logdir"></a>[LogDir](#tab/LogDir)
Kubelet および kube ログがそれぞれの出力ファイルにリダイレクトされるディレクトリ。

| パラメーター | 既定値 |
|---|---|
| `-LogDir` | `C:\k` |

---


> [!TIP]
> 何か問題が発生した場合は、 [「トラブルシューティング」セクション](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)を参照してください。

## <a name="manual-approach"></a>手動によるアプローチ
上記の [参照スクリプト](#registering-windows-services) が機能しないようにするために、このセクションでは、これらのサービスを手動で登録するために使用できる *サンプルコマンド* をいくつか紹介します。

> [!TIP]
> を[Kubelet and kube-proxy can now run as Windows services](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) `kubelet` 介して `kube-proxy` ネイティブ Windows サービスとしてを構成および実行する方法の詳細については、「Kubelet and kube」を参照してください `sc` 。

### <a name="register-flanneldexe"></a>flanneld.exe の登録
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>kubelet.exe の登録
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>kube-proxy.exe の登録 (l2bridge/ホスト-gw)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>kube-proxy.exe の登録 (オーバーレイ/vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```