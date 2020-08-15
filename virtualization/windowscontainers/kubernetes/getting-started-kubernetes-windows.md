---
title: Windows で使用する Kubernetes
author: daschott
ms.author: daschott
ms.date: 08/13/2020
ms.topic: overview
description: Windows での Kubernetes の概要
keywords: kubernetes、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e5e468d3c0092752e13f0dbbb4d9c87bd0328e9
ms.sourcegitcommit: aa139e6e77a27b8afef903fee5c7ef338e1c79d4
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/15/2020
ms.locfileid: "88251545"
---
# <a name="kubernetes-on-windows"></a>Windows で使用する Kubernetes

> [!TIP]
> Windows でどの Kubernetes 機能がサポートされているかを知りたい方は、 詳細については、 [公式にサポートされている機能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) と [Windows の Kubernetes ロードマップ](https://github.com/orgs/kubernetes/projects/8) を参照してください。

このページでは、Windows で Kubernetes を使用するための概要を説明します。


### <a name="try-out-kubernetes-on-windows"></a>Windows で Kubernetes を試す

任意の環境で Windows に Kubernetes を手動でインストールする方法については、「 [windows での Kubernetes の展開](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/) 」を参照してください。


### <a name="scheduling-windows-containers"></a>Windows コンテナーのスケジュール設定

Kubernetes での Windows コンテナーのスケジュール設定に関するベストプラクティスと推奨事項については、「 [Kubernetes での windows コンテナーのスケジュール設定](https://kubernetes.io/docs/setup/production-environment/windows/user-guide-windows-containers/) 」を参照してください。


### <a name="deploying-kubernetes-on-windows-in-azure"></a>Azure での Kubernetes のデプロイ

[Azure Kubernetes Service の Windows コンテナーで](/azure/aks/windows-container-cli)は、これを簡単に行うことができます。 すべての Kubernetes コンポーネントを自分でデプロイして管理する場合は、オープンソースツールを使用した [手順に関するチュートリアル](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md) を参照してください `AKS-Engine` 。

### <a name="troubleshooting"></a>トラブルシューティング
既知の問題の回避策と解決策の一覧については、「 [Kubernetes のトラブルシューティング](./common-problems.md) 」を参照してください。
>[!TIP]
> セルフヘルプに関するその他のリソースについては、Kubernetes ネットワークのトラブルシューティングガイド[を参照してください。](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)