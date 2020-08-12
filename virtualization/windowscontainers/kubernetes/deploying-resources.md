---
title: Kubernetes リソースのデプロイ
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: how-to
description: 混合 OS Kubernetes クラスターに Kubernetes resoureces をデプロイします。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: d1f0d836b6af9330acba5599b7f89a4c76baea10
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161881"
---
# <a name="deploying-kubernetes-resources"></a>Kubernetes リソースのデプロイ

Kubernetes クラスターが少なくとも1つのマスターと1ワーカーで構成されている場合は、Kubernetes リソースをデプロイすることができます。

> [!TIP]
> Windows で現在どのような Kubernetes リソースがサポートされていますか。 詳細については、「 [Windows ロードマップで公式に](https://github.com/orgs/kubernetes/projects/8)[サポートされている機能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)」および「Kubernetes」を参照してください。

## <a name="running-a-sample-service"></a>サンプルサービスの実行

ここでは、確実にクラスターへの参加に成功し、ネットワークを正しく構成できるように、非常に単純な [PowerShell ベースの Web サービス](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)を展開します。

これを行う前に、すべてのノードが正常であることを確認することをお勧めします。

```bash
kubectl get nodes
```

すべて問題がなければ、次のサービスをダウンロードして実行できます。

> [!IMPORTANT]
> 前に `kubectl apply` 、 `microsoft/windowsservercore` サンプルファイル内のイメージを、[ノードで](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)実行可能なコンテナーイメージに再確認または変更してください。

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

これにより、デプロイとサービスが作成されます。 最後の watch コマンドは、ポッドを無制限に照会してその状態を追跡します。をクリック `Ctrl+C` するだけで、 `watch` 観察を終えたときにコマンドが終了します。

すべての作業を正しく行った場合は、次のことを確認できます。

  - Windows ノードのコマンドの下にあるポッドあたり2つのコンテナーを参照してください。 `docker ps`
  - Linux マスターで `kubectl get pods` コマンドにより 2 つのポッドが表示されること。
  - `curl`Linux マスターからのポート80の*ポッド*ip では、web サーバー応答を取得します。これは、ネットワーク経由で通信するための適切なノードを示しています。
  - `docker exec` で*ポッド間* (複数の Windows ノードがある場合はホスト間も含めて) の ping を実行できること。これは、ポッド間の通信が適切であることを示します。
  - `curl`Linux マスターと個々のポッドからの仮想*サービス IP* (の下に表示され `kubectl get services` ます)。これは、通信をポッドするための適切なサービスを示します。
  - `curl`サービス*名*と Kubernetes の[既定の DNS サフィックス](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)を使用して、適切なサービス検出をデモンストレーションします。
  - `curl`Linux マスターまたはクラスター外のコンピューターからの*Nodeport* 。これは、受信接続を示しています。
  - `curl`ポッド内部からの外部 Ipこれは、送信接続を示しています。

> [!NOTE]
> Windows*コンテナーホスト*は、スケジュールされているサービスからサービス IP にアクセスすることはでき**ません**。 これは、今後のバージョンの Windows Server で改善されることがわかっている[プラットフォームの制限](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)です。 ただし、Windows*ポッド***は**サービス IP にアクセスできます。

## <a name="next-steps"></a>次のステップ

このセクションでは、Windows ノードで Kubernetes リソースをスケジュールする方法について説明します。 これでガイドは終わりです。 問題が発生した場合は、「トラブルシューティング」セクションを参照してください。

> [!div class="nextstepaction"]
> [トラブルシューティング](./common-problems.md)

それ以外の場合は、Kubernetes コンポーネントを Windows サービスとして実行することにも興味があります。
> [!div class="nextstepaction"]
> [Windows サービス](./kube-windows-services.md)
