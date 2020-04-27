---
title: Windows 10 上の Windows コンテナーおよび Linux コンテナー
description: コンテナー展開のクイック スタート
keywords: Docker、コンテナー、LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "74909582"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上の Linux コンテナー

この演習では、Windows 10 での Linux コンテナーの作成と実行について説明します。

このクイックスタートでは、次の作業を行います。

1. Docker Desktopのインストール中
2. Windows 上の Linux コンテナー (LCOW) を使用した単純な Linux コンテナーの実行

このクイック スタートは、Windows 10 に固有です。 このページの左側の目次に追加のクイック スタート文書があります。

## <a name="prerequisites"></a>前提条件

次の要件を満たしていることを確認してください。
- Windows 10 Professional、Windows 10 Enterprise、または Windows Server 2019 バージョン1809 以降を実行している 1 台の物理コンピューターシステム
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) は有効にされていることを確認します。

***Hyper-V による分離:*** Windows 上の Linux コンテナーでは、開発者が適切な Linux カーネルを使用してコンテナーを実行するために、Windows 10 で Hyper-v を分離する必要があります。 Hyper-v の分離の詳細については、「[Windows コンテナーの詳細](../about/index.md)」ページを参照してください。

## <a name="install-docker-desktop"></a>Docker Desktop のインストール

[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) をダウンロードし、インストーラーを実行します (ログインする必要があります。 アカウントをお持ちでない場合は作成します)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

> Docker が既にインストールされている場合は、LCOW をサポートするためにバージョン18.02 以降がインストールされていることを確認してください。 `docker -v` を実行して確認するか、*Docker の詳細*を確認してください。

> LCOW コンテナーを実行するには、*Docker 設定 > デーモン* の「試験的な機能」オプションをアクティブにする必要があります。

## <a name="run-your-first-lcow-container"></a>最初の LCOW コンテナーを実行する

この例では、BusyBox コンテナーがデプロイされます。 最初に、' Hello World ' BusyBox イメージを実行しようとします。

```console
docker run --rm busybox echo hello_world
```

Docker がイメージをプルしようとするとエラーが返されることに注意してください。 これは、イメージとホストオペレーティングシステムが適切に一致していることを確認するため、Dockers が`--platform` フラグから命令を要求するため発生します。 Windows コンテナーモードの既定のプラットフォームは Windows であるため、コンテナーをプルして実行するには `--platform linux` フラグを追加します。

```console
docker run --rm --platform linux busybox echo hello_world
```

指定されたプラットフォームでイメージがプルされると、`--platform` フラグは不要になります。 これをテストするためにそれなしでコマンドを実行します。

```console
docker run --rm busybox echo hello_world
```

`docker images` を実行して、インストールされているイメージの一覧を返します。 この場合は、Windows と Linux の両方のイメージです。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus:LCOW の実行については、Docker の対応する [ブログ投稿](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) を参照してください。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [サンプル アプリをビルドする方法について説明します](./building-sample-app.md)
