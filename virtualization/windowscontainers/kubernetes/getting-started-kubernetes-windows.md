---
title: Windows で使用する Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.prod: containers
ms.topic: overview
description: Windows ノードを Kubernetes クラスターに追加する (v 1.14)。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 43525b86e00ad3d855f98394aa96ffe987fb51d6
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192329"
---
# <a name="kubernetes-on-windows"></a>Windows で使用する Kubernetes

このページでは、Windows ノードを Linux ベースのクラスターに参加させることによって、Windows で Kubernetes を開始するための概要を説明します。 Windows Server[バージョン 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)の Kubernetes 1.14 のリリースでは、ユーザーは windows 上の Kubernetes で次の機能を利用できます。

- **オーバーレイネットワーク**: vxlan モードで Flannel を使用して仮想オーバーレイネットワークを構成する
    - Windows Server 2019 with [KB4489899](https://support.microsoft.com/help/4489899)がインストールされているか、 [Windows Server Vnext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Build 18317 + が必要です
    - 機能ゲートが有効になっている Kubernetes v1.0 (以上) が必要です `WinOverlay`
    - Flannel v 0.11.0 (またはそれ以上) が必要です
- **ネットワーク管理の簡素化**: ノード間の自動ルート管理には、ホストゲートウェイモードで Flannel を使用します。
- **スケーラビリティの向上**: [Windows Server コンテナーの deviceless vnics](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716)により、より高速で信頼性の高いコンテナーの起動時間を実現できます。
- **Hyper-v 分離 (アルファ)**: セキュリティ強化のために、カーネルモード分離を使用して[hyper-v の分離](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)を調整します。 詳細については、「 [Windows コンテナーの種類](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)」を参照してください。
    - 機能ゲートが有効になっている Kubernetes v 1.10 以上が必要です `HyperVContainer` 。
- **ストレージプラグイン**: Windows コンテナーに対しては、SMB および iSCSI でサポートされている[flexvolume storage プラグイン](https://github.com/Microsoft/K8s-Storage-Plugins)を使用します。

>[!TIP]
>クラスターを Azure にデプロイする場合は、オープンソースの AKS ツールを使用すると簡単にできます。 詳細については、ステップバイステップ[チュートリアル](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)を参照してください。

## <a name="prerequisites"></a>前提条件

### <a name="plan-ip-addressing-for-your-cluster"></a>クラスターの IP アドレス指定を計画する

<a name="definitions"></a>Kubernetes クラスターはポッドとサービスの新しいサブネットを導入するため、環境内の他の既存のネットワークと競合しないようにすることが重要です。 Kubernetes を正常に展開するために解放する必要があるすべてのアドレス空間を次に示します。

| サブネット/アドレス範囲 | 説明 | Default value |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**サービスサブネット** | ルーティング不可能な、純粋な仮想サブネット。ネットワークトポロジに関する気なしで uniformally access services にアクセスするためにポッドによって使用されます。 ノードで実行されている `kube-proxy` によって、ルーティング可能なアドレス空間との間で変換が行われます。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**クラスターサブネット** |  これは、クラスター内のすべてのポッドによって使用されるグローバルサブネットです。 各ノードには、ポッドが使用できるように、これより小さい/24 のサブネットが割り当てられます。 クラスターで使用されているすべてのポッドを格納するのに十分な大きさである必要があります。 サブネットの*最小*サイズの計算式は、`(number of nodes) + (number of nodes * maximum pods per node that you configure)` です。 <p/>ノードあたり100ポッドの5ノードクラスターの例: `(5) + (5 *  100) = 505` 。  | "10.244.0.0/16" |
| **Kubernetes DNS サービス IP** | クラスターサービス検出 & DNS 解決に使用される "kube" サービスの IP アドレス。 | "10.96.0.10" |

> [!NOTE]
> Docker をインストールすると、既定で作成される別の Docker ネットワーク (NAT) があります。 代わりにクラスターサブネットから Ip を割り当てているため、Windows で Kubernetes を操作する必要はありません。

## <a name="what-you-will-accomplish"></a>作業内容

このガイドでは、次のことを行います。

> [!div class="checklist"]
> * [Kubernetes マスター](./creating-a-linux-master.md)ノードが作成されました。
> * [ネットワークソリューション](./network-topologies.md)を選択しました。
> * [Windows ワーカーノード](./joining-windows-workers.md)または[Linux ワーカーノード](./joining-linux-workers.md)を参加させます。
> * [サンプル Kubernetes リソース](./deploying-resources.md)をデプロイしました。
> * [一般的な問題とよくある誤り](./common-problems.md)を理解する。

## <a name="next-steps"></a>次の手順

このセクションでは、現在、Windows で Kubernetes を正常に展開するために必要な前提条件 & 重要な前提条件について説明しました。 Kubernetes マスターのセットアップ方法については、「」を参照してください。

>[!div class="nextstepaction"]
>[Kubernetes マスターを作成する](./creating-a-linux-master.md)