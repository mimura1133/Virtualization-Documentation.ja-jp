---
title: Windows コンテナーのオーケストレーションの概要
description: Windows コンテナーのオーケストレーションについて説明します。
keywords: Docker, コンテナー
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "78853908"
---
# <a name="windows-container-orchestration-overview"></a>Windows コンテナーのオーケストレーションの概要

コンテナーは、サイズが小さく、アプリケーション指向であるため、アジャイル配信環境やマイクロサービス ベースのアーキテクチャに最適です。 ただし、コンテナーやマイクロサービスを使用する環境には、追跡するコンポーネントが数百または数千存在する可能性があります。 数十台の仮想マシンや物理サーバーは手動で管理することも可能かもしれませんが、実稼働規模のコンテナー環境を、オートメーションを使用せずに適切に管理する方法はありません。 このタスクは Orchestrator に任せる必要があります。これは、多数のコンテナーと、それらが相互に対話する方法を自動化および管理するプロセスです。

Orchestrator は以下のタスクを実行します。

- スケジュール設定:特定のコンテナー イメージとリソース要求が指定されると、そのコンテナーを実行するのに適したマシンが Orchestrator によって見つけられます。
- アフィニティ/アンチアフィニティ:パフォーマンスのために一連のコンテナーを互いに近くで実行するか、可用性のために距離をおいて実行するかを指定します。
- 正常性の監視:コンテナー エラーを監視し、自動的にそれらのスケジュールを再調整します。
- フェールオーバー:各マシンで何が実行されているかを追跡し、エラーが発生したマシンから正常なノードにコンテナーのスケジュールを再調整します。
- 規模の調整:需要に合わせてコンテナー インスタンスを手動または自動で追加または削除します。
- ネットワーク:複数のホスト マシン間で通信するためにコンテナーを調整するオーバーレイ ネットワークを提供します。
- サービスの検出:コンテナーがホスト マシン間で移動され、IP アドレスが変更された場合でも、コンテナーが自動的に互いを見つけられるようにします。
- アプリケーション アップグレード調整:問題が発生した場合にアプリケーションのダウンタイムを回避してロールバックできるように、コンテナーのアップグレードを管理します。

## <a name="orchestrator-types"></a>Orchestrator の種類

Azure では、次の 2 種類のコンテナー用 Orchestrator が提供されています。Azure Kubernetes Service (AKS) と Azure Service Fabric です。

[Azure Kubernetes Service (AKS)](/azure/aks/) を使用すると、コンテナー化されたアプリケーションを実行するように事前に構成された仮想マシンのクラスターを、簡単に作成、構成、管理できます。 これにより、既存のスキルを利用し、発展し続ける膨大なコミュニティの専門知識を活用して、Microsoft Azure でコンテナー ベースのアプリケーションをデプロイし、管理することができます。 AKS を使用することで、Kubernetes と Docker イメージ形式を通じてアプリケーションの移植性を引き続き維持しながら、Azure のエンタープライズ レベルの機能を活用することができます。

[Azure Service Fabric](/azure/service-fabric/) は、スケーラブルで信頼性の高いマイクロサービスとコンテナーを容易にパッケージ化、展開、管理できる分散システム プラットフォームです。 Service Fabric によって、クラウド ネイティブ アプリケーションを開発、管理する際の重要な問題を解決します。 開発者や管理者は、複雑なインフラストラクチャの問題を回避し、ミッション クリティカルで要求の厳しいワークロードを、スケーラブルで、信頼性が高く、管理しやすい方法で実装することに集中できます。 Service Fabric は、このようなエンタープライズ クラス、階層 1、クラウド規模のアプリケーションを構築および管理するための次世代プラットフォームです。

## <a name="getting-started"></a>はじめに

Azure Kubernetes Service のデプロイを開始するには、[Kubernetes のセットアップ ガイド](../kubernetes/getting-started-kubernetes-windows.md)のページを参照してください。

Azure Service Fabric のデプロイを開始するには、[Service Fabric のクイックスタート](/azure/service-fabric/service-fabric-quickstart-containers.md)のページを参照してください。
