---
title: Docker について
description: Docker の詳細
keywords: Docker, コンテナー
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: a2a1f89d7a62479dfae073c7f28fa4a86c693597
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192879"
---
# <a name="about-docker"></a>Docker について

コンテナーについての資料で、必ず言及されるのが Docker です。 Docker Engineは、コンテナー イメージのパッケージ化と配信を行うコンテナー管理ツールセットです。 生成されたイメージは、コンテナーとして、オンプレミス、クラウド、または個人用マシンのいずれでも実行できます。

![](media/docker.png)

Windows Server コンテナーは、他のコンテナーと同様、[Docker](https://www.docker.com) によって管理できます。

コンテナーを通じた名前空間の分離とリソース管理の概念は、ずっと以前からあり、BSD Jail、Solaris ゾーン、UNIX の基本的なルート変更 (chroot) メカニズムにまでさかのぼります。 Docker は、アプリケーションのコンテナー化と配信を簡略化する共通ツールセット、パッケージングモデル、およびデプロイ メカニズムを使用して、開発用の堅固な基盤となります。 これらのアプリケーションは、任意の Linux ホストおよび Windows 上の任意の場所で実行できます。

一般的なパッケージング モデルおよびデプロイ テクノロジは、任意のホストに対して同じ管理コマンドを提供することで管理を合理化するだけでなく、シームレスな DevOps にまたとない機会をもたらします。 開発者のデスクトップ、テスト マシン、または一連の実稼働コンピューターのいずれであっても、秒単位で任意の環境全体に同じように展開される Docker のイメージを作成することができます。 これにより、Docker コンテナーにパッケージ化された、大規模で拡大するエコシステムが作成されてきました。このエコシステムには、DockerHub と、Docker が保持するパブリック コンテナー化されたアプリケーション レジストリがあります。

次に、アプリケーションのエコシステムについて説明し、ニーズに合った開発およびデプロイのワークフローを作成するための Docker 概念を構築する方法について説明しましょう。

## <a name="get-started-with-docker"></a>Docker の概要

Docker を使用してコンテナーを構築する方法については、「[Docker Engine on Windows](../manage-docker/configure-docker-daemon.md)」を参照してください。 また、Docker の使用方法の詳細については、[Docker の Web サイト](https://www.docker.com)にアクセスすることもできます。