---
title: Kubernetes バイナリのコンパイル
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: how-to
ms.prod: containers
description: ソースからの Kubernetes バイナリのコンパイルとクロスコンパイル
keywords: kubernetes、1.12、linux、コンパイル
ms.openlocfilehash: a0c9ed6ef0872e19de49fa97f4727b6e0e09ed43
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192349"
---
# <a name="compiling-kubernetes-binaries"></a>Kubernetes バイナリのコンパイル #
Kubernetes のコンパイルには、有効な Go 環境が必要です。 このページでは、Linux バイナリをコンパイルし、Windows バイナリをクロスコンパイルするための複数の方法を確認します。
> [!NOTE]
> このページは完全に自発的であり、Kubernetes の開発者にのみ含まれており、最新の & 最大のソースコードを試してみることをお勧めします。

> [!tip]
> サブスクライブできる最新の開発に関する通知を受け取ることができ [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce) ます。

## <a name="installing-go"></a>Go のインストール ##
ここでは、わかりやすくするために、カスタムの一時的な場所に Go をインストールします。

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]
> これらは、セッション用の環境変数を設定しています。 永続的な設定を行う場合は、`~/.profile` に `export` を追加します。

パスが正しく設定されていることを確認するには、`go env` を実行します。 Kubernetes バイナリを構築するには、いくつかのオプションがあります。

  - [ローカル](#build-locally)でビルドする。
  - [Vagrant](#build-with-vagrant) を使用してバイナリを生成する。
  - Kubernetes プロジェクトの[コンテナー化された標準ビルド スクリプト](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts)を活用する。 これには、「[ローカルでビルドする](#build-locally)」の `make` までの手順に従い、リンクされた手順を使用します。

Windows バイナリを各ノードにコピーするには、[WinSCP](https://winscp.net/eng/download.php) などのビジュアル ツールや、[pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) などのコマンドライン ツールを使用して、`C:\k` ディレクトリに転送します。


## <a name="building-locally"></a>ローカルでのビルド ##
> [!Tip]
> "アクセス許可が拒否されました" というエラーが発生した場合は、次の手順に従って、最初に Linux をビルドすることで回避できます `kubelet` [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176) 。
>
> _Kubernetes Windows ビルドシステムでは、バグのように見えますが、最初に、生成する Linux バイナリをビルドする必要があり `_output/bin/deepcopy-gen` ます。これを実行して Windows をビルドすると、空のが生成され `deepcopy-gen` ます。_

まず、Kubernetes リポジトリを取得します。

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

ここで、ブランチをチェックアウトし、Linux の `kubelet` バイナリをビルドします。 これは、上記の Windows ビルド エラーを回避するために必要です。 ここでは `v1.12.2` を使用します。 `git checkout` の後で、保留中の PR またはパッチを適用することや、カスタム バイナリにその他の変更を行うことができます。

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

最後に、必要な Windows クライアント バイナリをビルドします (後で Windows バイナリを取得する場所によって、最後の手順は異なる場合があります)。

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Linux バイナリをビルドする手順は同じです。コマンドの `KUBE_BUILD_PLATFORMS=windows/amd64` プレフィックスを省略するだけです。 出力ディレクトリは `_output/.../linux/amd64` になります。


## <a name="build-with-vagrant"></a>Vagrant によるビルド ##
Vagrant セットアップは[こちら](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant)にあります。 これを使用して Vagrant VM を準備し、その中でこれらのコマンドを実行します。

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.12.2
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```

