---
title: Windows 10 上の Windows コンテナーおよび Linux コンテナー
description: コンテナー展開のクイック スタート
keywords: Docker、コンテナー、LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: tutorial
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: df02dada3e5cf759f003999b38270dd1bf3131fe
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192189"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上の Linux コンテナー

この演習では、Windows 10 での Linux コンテナーの作成と実行について説明します。

このクイックスタートでは、次の作業を行います。

1. Docker Desktopのインストール
2. 単純な Linux コンテナーの実行

このクイック スタートは、Windows 10 に固有です。 このページの左側の目次に追加のクイック スタート文書があります。

## <a name="prerequisites"></a>前提条件

次の要件を満たしていることを確認してください。
- Windows 10 Professional、Windows 10 Enterprise、または Windows Server 2019 バージョン1809 以降を実行している 1 台の物理コンピューターシステム
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) は有効にされていることを確認します。

## <a name="install-docker-desktop"></a>Docker Desktop のインストール

[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) をダウンロードし、インストーラーを実行します (ログインする必要があります。 アカウントをお持ちでない場合は作成します)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

## <a name="run-your-first-linux-container"></a>最初の Linux コンテナーの実行

Linux コンテナーを実行するには、Docker が適切なデーモンをターゲットにしていることを確認する必要があります。 これを切り替えるには、システム トレイの Docker のクジラのアイコンをクリックして、操作メニューから `Switch to Linux Containers` (Linux コンテナーに切り替え) を選択します。 `Switch to Windows Containers` (Windows コンテナーに切り替え) が表示された場合、既に Linux デーモンをターゲットにしています。

![[Switch to Windows containers] (Windows コンテナーに切り替え) コマンドが表示されている Docker システム トレイ メニュー。](./media/switchDaemon.png)

適切なデーモンをターゲットにしていることを確認したら、次のコマンドを使用してコンテナーを実行します。

```console
docker run --rm busybox echo hello_world
```

コンテナーが実行され、"hello_world" と出力されて、終了します。

`docker images` に対してクエリを実行すると、プルして実行した Linux コンテナー イメージが表示されます。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [サンプル アプリをビルドする方法について説明します](./building-sample-app.md)
