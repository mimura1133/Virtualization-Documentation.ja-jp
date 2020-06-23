---
title: Kubernetes マスターの新規作成
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
ms.prod: containers
description: Kubernetes クラスターマスターを作成しています。
keywords: kubernetes、1.14、master、linux
ms.openlocfilehash: a46c8e996162891cc596946d8601bcb590b2b8eb
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192389"
---
# <a name="creating-a-kubernetes-master"></a>Kubernetes マスターの作成 #
> [!NOTE]
> このガイドは Kubernetes v 1.14 で検証されました。 このセクションでは、バージョン間の Kubernetes の変動によって、将来のすべてのバージョンに対して真であるとは限りません。 Kubeadm を使用した Kubernetes マスターの初期化に関する公式ドキュメントについては、[こちら](https://kubernetes.io/docs/setup/independent/install-kubeadm/)を参照してください。 その上で[、混合 OS スケジューリングセクション](#enable-mixed-os-scheduling)を有効にするだけです。

> [!NOTE]
> 最近更新された Linux マシンは、次の手順で実行する必要があります。[Kube](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)、 [kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)、 [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)などの Kubernetes マスターリソースが Windows にまだ移植されていません。

> [!tip]
> Linux の手順は、 **Ubuntu 16.04**に合わせて調整されています。 Kubernetes を実行するために認定された他の Linux ディストリビューションにも、代替として使用できる同等のコマンドが用意されています。 また、Windows との相互運用も正常に行われます。


## <a name="initialization-using-kubeadm"></a>Kubeadm を使用した初期化 ##
明示的に指定しない限り、以下のコマンドを**root**として実行します。

まず、管理者特権のルートシェルにアクセスします。

```bash
sudo –s
```

コンピューターが最新の状態であることを確認します。

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Docker をインストールする ###
コンテナーを使用できるようにするには、Docker などのコンテナーエンジンが必要です。 最新バージョンを入手するには、Docker のインストールに関する[次の手順](https://docs.docker.com/install/linux/docker-ce/ubuntu/)を使用します。 コンテナーを実行することで、docker が正しくインストールされていることを確認でき `hello-world` ます。

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Kubeadm のインストール ###
`kubeadm`Linux ディストリビューションのバイナリをダウンロードし、クラスターを初期化します。

> [!Important]
> Linux ディストリビューションによっては、以下を正しいコードネームに置き換える必要がある場合があり `kubernetes-xenial` ます。 [codename](https://wiki.ubuntu.com/Releases)

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl
```

### <a name="prepare-the-master-node"></a>マスターノードを準備する ###
Linux 上の Kubernetes では、スワップ領域を無効にする必要があります。

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

### <a name="initialize-master"></a>マスターの初期化 ###
クラスターサブネット (例: 10.244.0.0/16) とサービスサブネット (例: 10.96.0.0/12) を書き留め、kubeadm を使用してマスターを初期化します。

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

これには数分かかることがあります。 完了すると、マスターが初期化されたことを確認する次のような画面が表示されます。

![テキスト](media/kubeadm-init.png)

> [!tip]
> この kubeadm join コマンドを書き留めておきます。 長さ kubeadm トークンの有効期限が切れると、を使用して `kubeadm token create --print-join-command` 新しいトークンを作成できます。

> [!tip]
> 使用する必要な Kubernetes のバージョンがある場合は、フラグを kubeadm に渡すことができ `--kubernetes-version` ます。

まだ完了していません。 を通常のユーザーとして使用するには、 `kubectl` 管理者__**特権ではないユーザーシェルで**次のように実行します。__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Kubectl を使用して、クラスターに関する情報を編集または表示できるようになりました。

### <a name="enable-mixed-os-scheduling"></a>混合 OS スケジューリングを有効にする ###
既定では、特定の Kubernetes リソースは、すべてのノードでスケジュールされているように書き込まれます。 ただし、マルチ OS 環境では、Linux リソースが Windows ノード上で相互に干渉またはダブルスケジューリングされることはありません。その逆も同様です。 このため、 [Nodeselector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)ラベルを適用する必要があります。

この点については、linux のみを対象とする linux kube [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)にパッチを適用します。

まず、ディレクトリを作成して、次のように格納します。 yaml マニフェストファイル:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

DaemonSet の更新方法 `kube-proxy` が[rol更新](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)に設定されていることを確認します。

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

次に、[この nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)をダウンロードして DaemonSet にパッチを適用し、ターゲット Linux のみに適用します。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

成功すると、の "ノードセレクター" と、 `kube-proxy` その他のすべてのデーモンセットがに設定されていることを確認します。`beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![テキスト](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>クラスター情報の収集 ###
将来のノードをマスターに正常に参加させるには、次の情報を記録しておく必要があります。
  1. `kubeadm join`出力からのコマンド ([ここ](#initialize-master))
    * 例: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. `kubeadm init`([ここ) で](#initialize-master)定義されたクラスターサブネット
    * 例: `10.244.0.0/16`
  3. `kubeadm init`([ここ) で](#initialize-master)定義されているサービスサブネット
    * 例: `10.96.0.0/12`
    * を使用することもできます。`kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube-dns サービス IP
    * 例: `10.96.0.10`
    * "Cluster IP" フィールドには、`kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` ファイル `kubeadm init` [は (ここ](#initialize-master)) の後に生成されます。 手順に従っている場合は、次のパスに記載されています。
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>マスターを確認しています ##
数分後、システムは次の状態になります。

  - では `kubectl get pods -n kube-system` 、 [Kubernetes マスターコンポーネント](https://kubernetes.io/docs/concepts/overview/components/#master-components)のポッドが状態になり `Running` ます。
  - を呼び出す `kubectl cluster-info` と、DNS のアドオンに加えて、Kubernetes マスター API サーバーに関する情報が表示されます。

> [!tip]
> Kubeadm ではネットワークがセットアップされないため、DNS ポッドがまたは状態のままになる場合があり `ContainerCreating` `Pending` ます。 `Running`[ネットワークソリューションを選択](./network-topologies.md)した後で、状態に切り替わります。

## <a name="next-steps"></a>次の手順 ##
このセクションでは、kubeadm を使用して Kubernetes マスターを設定する方法について説明しました。 これで、手順3の準備ができました。

> [!div class="nextstepaction"]
> [ネットワークソリューションの選択](./network-topologies.md)