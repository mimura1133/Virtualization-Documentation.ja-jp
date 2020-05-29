---
title: Windows 10 上の Linux コンテナー
description: Hyper-V を使用することにより、Linux コンテナーを Windows 10 上でネイティブであるかのように実行するさまざまな方法について説明します。
keywords: LCOW, Linux コンテナー, Docker, コンテナー, Windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 843bd0ab7ccf3a227482ba3a3d2677e36b395b29
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854016"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上の Linux コンテナー

Linux コンテナーは、コンテナー エコシステム全体の大部分を占めており、開発者エクスペリエンスと運用環境の両方の基盤となるものです。  しかし、コンテナーはコンテナー ホストとカーネルを共有するため、Windows で Linux コンテナーを直接実行することはできません[*](linux-containers.md#other-options-we-considered)。  ここで、仮想化が役立ちます。

現時点では、Docker for Windows と Hyper-V を使用して、次の 2 つの方法で Linux コンテナーを実行できます。

- Linux コンテナーを完全な Linux VM で実行する。これは、現在の Docker の一般的な機能です。
- Linux コンテナーを [Hyper-V による分離](../manage-containers/hyperv-container.md) (LCOW) で実行する。これは、Docker for Windows の新しいオプションです。

> _Windows Server OS 上で Linux コンテナーを実行することは、現在はまだ実験段階です。これを試すには、Docker EE プログラムの追加ライセンスが必要になります。**この記事のこれ以降の部分は、Windows 10 にのみ関係します**。_

この記事では、それぞれのアプローチのしくみを説明し、どのような場合にどちらのソリューションを選択すべきかに関するガイダンスを提供します。また、進行中の作業についても説明します。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM の Linux コンテナー

Linux VM で Linux コンテナーを実行するには、[Docker のスターティング ガイド](https://docs.docker.com/docker-for-windows/)の指示に従います。

Docker では、Hyper-V で動作する [LinuxKit](https://github.com/linuxkit/linuxkit) ベースの仮想マシンを使用することにより、2016 年の最初のリリースの時点 (Windows で Hyper-V による分離や Linux コンテナーが使用可能になる前) から、Windows デスクトップ上で Linux コンテナーを実行できました。

このモデルでは、Docker クライアントは Windows デスクトップ上で実行されますが、Linux VM 上の Docker デーモンを呼び出します。

![コンテナー ホストとしての Moby VM](media/MobyVM.png)

このモデルでは、すべての Linux コンテナーが 1 つの Linux ベース コンテナー ホストを共有しており、すべての Linux コンテナーに次の特徴があります。

* カーネルを相互に、また Moby VM とは共有するが、Windows ホストとは共有しない。
* Linux 上で実行されている Linux コンテナーと、ストレージおよびネットワークのプロパティが一貫している (Linux VM 上で実行されているため)。

これは、Linux コンテナー ホスト (Moby VM) が、Docker デーモンと Docker デーモンのすべての依存関係を実行している必要があることも意味しています。

Moby VM で実行しているかどうかを確認するには、Hyper-V マネージャー UI を使用するか、管理者特権での PowerShell ウィンドウで `Get-VM` を実行して、Moby VM の Hyper-V マネージャーを確認します。

## <a name="linux-containers-with-hyper-v-isolation"></a>Hyper-V による分離を使用した Linux コンテナー

Windows 10 (LCOW10) 上の Linux コンテナーを試すには、「[Windows 10 上の Linux コンテナー](../quick-start/quick-start-windows-10-linux.md)」にある Linux コンテナーに関する手順に従います。 

Hyper-V による分離を使用した Linux コンテナーは、コンテナーを実行するのに十分な OS を使用し、最適化された Linux VM 内で各 Linux コンテナーを実行します。 Moby VM のアプローチとは異なり、各 Linux コンテナーには独自のカーネルと独自の VM サンドボックスがあります。 また、Windows 上の Docker によって直接管理されます。

![Hyper-V による分離を使用した Linux コンテナー (LCOW)](media/lcow-approach.png)

Moby VM のアプローチと LCOW とのコンテナーの管理方法の違いをさらに詳しく見てみると、LCOW モデルでは、コンテナーの管理は Windows 上で引き続き行われ、それぞれの LCOW 管理は GRPC と containerd を介して行われます。  これは、LCOW の Linux ディストリビューション コンテナーの使用ではインベントリがはるかに少なくて済むことを意味します。  現時点では、ディストリビューション コンテナーの使用を最適化するため LinuxKit が使用されていますが、Kata などの他のプロジェクトでも同様の高度に調整された Linux ディストリビューション (Clear Linux) が構築されています。

各 LCOW をさらに詳しく見ると、次のようになっています。

![LCOW アーキテクチャ](media/lcow.png)

LCOW を実行しているかどうかを確認するには、`C:\Program Files\Linux Containers` に移動します。 Docker が LCOW を使用するように構成されている場合は、こちらに最小限の LinuxKit ディストリビューションを含むファイルがいくつか存在します。これらは、Hyper-V による分離下で実行される各コンテナーで実行されます。  最適化された VM コンポーネントは、100 MB 未満であり、Moby VM の LinuxKit イメージよりもはるかに小さいことにご注意ください。

### <a name="work-in-progress"></a>進行中の作業

LCOW は活発に開発が進められています。 Moby プロジェクトで進行中の作業の進捗状況は、[GitHub](https://github.com/moby/moby/issues/33850) で追跡できます。

#### <a name="bind-mounts"></a>バインド マウント

`docker run -v ...` を使ってボリュームをバインド マウントした場合、Windows NTFS ファイルシステムにファイルが格納されるため、POSIX 操作のための変換が必要になります。 しかしファイル システム操作の中には、現在部分的に、またはまったく実装されていない操作があるため、一部のアプリは互換性がありません。

以下の操作は、現在、バインド マウントされたボリュームには機能しません。

* MkNod
* XAttrWalk
* XAttrCreate
* ロック
* Getlock
* Auth
* フラッシュ
* INotify

完全に実装されていない操作も少数ですが存在します。

* GetAttr: Nlink カウントは、常に 2 と報告されます。
* Open: ReadWrite フラグ、WriteOnly フラグ、および ReadOnly フラグのみが実装されています。

これらのアプリケーションはすべてボリューム マッピングを必要とするため、正常に起動または動作しません。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>追加情報

[LCOW に関する Docker ブログ](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux コンテナーのビデオ](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW - カーネルおよびビルドの手順](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby VM と LCOW のどちらを使用すべき状況か

### <a name="when-to-use-moby-vm"></a>Moby VM を使用すべき場合

現時点で、次のようなユーザーには、Linux コンテナーを実行する Moby VM による方法をお勧めします。

- 安定したコンテナー環境を必要とする。  これは Docker for Windows の既定の設定です。
- Windows か Linux のコンテナーを実行するが、両方を同時に実行することはほとんどない。
- Linux コンテナーどうしの間に複雑な、またはカスタム ネットワーク要件がある。
- Linux コンテナーどうしの間でカーネルの分離 (Hyper-V による分離) を必要としない。

### <a name="when-to-use-lcow"></a>LCOW を使用すべき場合

現時点で、次のようなユーザーに LCOW をお勧めします。

- Microsoft の最新のテクノロジをテストしたい。
- Windows と Linux のコンテナーを同時に実行する。
- Linux コンテナーどうしの間でカーネルの分離 (Hyper-V による分離) を必要とする。

## <a name="other-options-we-considered"></a>検討された他のオプション

Linux コンテナーを Windows で実行する方法を検討するにあたり、WSL も考慮されました。 最終的に、Windows 上の Linux コンテナーが Linux 上の Linux コンテナーと整合するように、仮想化ベースのアプローチを選択しました。 また、Hyper-V を使用すると、LCOW のセキュリティが強化されます。 将来的に再評価されることも考えられますが、現時点では、LCOW では引き続き Hyper-V が使用されます。

ご意見がある場合は、GitHub または UserVoice からフィードバックをお寄せください。  実現をご希望の特定のエクスペリエンスに関するフィードバックがあれば、ぜひお知らせください。
