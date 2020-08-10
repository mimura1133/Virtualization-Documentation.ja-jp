---
title: gMSA を使用してコンテナーを調整する
description: グループ管理されたサービス アカウント (gMSA) を使用して Windows コンテナーを調整する方法。
keywords: Docker, コンテナー, Active Directory, gMSA, オーケストレーション, Kubernetes, グループ管理されたサービス アカウント, グループ管理されたサービス アカウント
author: rpsqrd
ms.date: 09/10/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 00bc05d3d96407d19b96620b3b26059f4ac9e313
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985256"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>gMSA を使用してコンテナーを調整する

運用環境では、アプリやサービスをデプロイして管理する目的でコンテナー オーケストレーターを使用することがよくあります。 オーケストレーターは、それぞれ独自の管理パラダイムがあり、Windows コンテナー プラットフォームに渡す資格情報の仕様を受け入れる役割を担います。

グループ管理されたサービス アカウント (gMSA) を使用してコンテナーを調整する場合、次のことを確認します。

> [!div class="checklist"]
> * gMSA を使用してコンテナーを実行するようにスケジュールできるすべてのコンテナー ホストがドメインに参加している。
> * コンテナー ホストに、コンテナーが使用するすべての gMSA のパスワードを取得するためのアクセス権がある。
> * 資格情報の仕様ファイルが作成されており、オーケストレーターで選択される処理方法に応じて、オーケストレーターにアップロードされるか、各コンテナー ホストにコピーされている。
> * コンテナー ネットワークにより、コンテナーが Active Directory ドメイン コントローラーと通信して、gMSA チケットを取得できる。

## <a name="how-to-use-gmsa-with-service-fabric"></a>Service Fabric で gMSA を使用する方法

アプリケーション マニフェストで資格情報の仕様の場所を指定すると、Service Fabric により、gMSA を使用した Windows コンテナーの実行がサポートされます。 資格情報の仕様ファイルを作成し、各ホストの Docker データ ディレクトリの **CredentialSpecs** サブディレクトリに配置して、Service Fabric がそのファイルを見つけられるようにする必要があります。 [CredentialSpec PowerShell モジュール](https://aka.ms/credspec)の一部である **Get-CredentialSpec** コマンドレットを実行することにより、資格情報の仕様が正しい場所にあるかどうかを確認できます。

「[クイックスタート:Service Fabric に Windows コンテナーをデプロイする](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)」および「[Service Fabric で実行されている Windows コンテナーに対して gMSA を設定する](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)」を参照して、アプリケーションの構成方法の詳細についてご確認ください。

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Docker Swarm で gMSA を使用する方法

Docker Swarm によって管理されているコンテナーで gMSA を使用するには、`--credential-spec` パラメーターを指定して [docker service create](https://docs.docker.com/engine/reference/commandline/service_create/) コマンドを実行します。

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker サービスで資格情報の仕様を使用する方法の詳細については、[Docker Swarm の例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)に関する記事を参照してください。

## <a name="how-to-use-gmsa-with-kubernetes"></a>Kubernetes で gMSA を使用する方法

Kubernetes での gMSA を使用した Windows コンテナーのスケジュール設定のサポートは、Kubernetes 1.14 のアルファ機能として利用できます。 この機能の最新情報と、この機能をお使いの Kubernetes ディストリビューションでテストする方法については、「[Windows ポッドおよびコンテナー用 gMSA の構成](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)」を参照してください。

## <a name="next-steps"></a>次の手順

コンテナーの調整に加えて、gMSA を使用して次のことも行えます。

- [アプリを構成する](gmsa-configure-app.md)
- [コンテナーを実行する](gmsa-run-container.md)

セットアップ中に問題が発生した場合は、[トラブルシューティング ガイド](gmsa-troubleshooting.md)で考えられる解決策をご確認ください。
