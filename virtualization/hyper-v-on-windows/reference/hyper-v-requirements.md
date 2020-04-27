---
title: Windows 10 Hyper-V のシステム要件
description: Windows 10 Hyper-V のシステム要件
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: ebc9be132f05c20eb8daf9b5e6713b9258012305
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "78853992"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Windows 10 Hyper-V のシステム要件

Hyper-V は、64 ビット版の Windows 10 Pro、Enterprise、および Education で使用できます。 Hyper-V には、第 2 レベルのアドレス変換 (SLAT) 機能が必要です。SLAT とは、Intel と AMD が提供する最新世代の 64 ビット プロセッサに搭載されている機能です。

4 GB の RAM を備えたホストでは基本的な仮想マシンを 3 つまたは 4 つ実行することができますが、仮想マシンが増えれば、より多くのリソースが必要となります。 一方で、物理ハードウェアの内容にもよりますが、32 のプロセッサと 512 GB の RAM を備えた大規模な仮想マシンを作成することもできます。

## <a name="operating-system-requirements"></a>オペレーティング システムの要件

Hyper-V のロールは、次のバージョンの Windows 10 で有効にすることができます。

- Windows 10 Enterprise
- Windows 10 Pro
- Windows 10 Education

Hyper-V のロールは、次のバージョンにはインストール**できません**。

- Windows 10 Home
- Windows 10 Mobile
- Windows 10 Mobile Enterprise

>Windows 10 Home Edition は、Windows 10 Pro にアップグレードすることができます。 アップグレードを行うには、 **[設定]**  >  **[更新とセキュリティ]**  >  **[アクティブ化]** の順に開きます。 ここでストアにアクセスして、アップグレードを購入することができます。

## <a name="hardware-requirements"></a>ハードウェア要件

このドキュメントでは、Hyper-V と互換性のあるハードウェアをすべて一覧していませんが、次のアイテムが必要です。

- 第 2 レベルのアドレス変換 (SLAT) の 64 ビット プロセッサ。
- VM モニター モード拡張機能 (Intel CPU の VT-x) の CPU サポート。
- 最小 4 GB のメモリ。 仮想マシンは Hyper-V ホストとメモリを共有するため、予想される仮想ワークロードを処理するために十分なメモリを提供する必要があります。

次の項目をシステム BIOS で有効にする必要があります。
- 仮想化テクノロジ (マザーボードの製造元によってラベルが異なる場合があります)
- ハードウェアによるデータ実行防止

## <a name="verify-hardware-compatibility"></a>ハードウェアの互換性の検証

上記のオペレーティング システムとハードウェアの要件を確認したら、PowerShell セッションまたはコマンドプロンプト (cmd.exe) のウィンドウを開いて、「**systeminfo**」と入力してから、「Hyper-v の要件」セクションを確認して、Windows でのハードウェアの互換性を確認します。 一覧表示された Hyper-V の要件がすべて **Yes** である場合、使用しているシステムで Hyper-V のロールを実行できます。 いずれかの項目が **No** を返す場合、このドキュメントに一覧された要件を確認して、可能な限り調整を行います。

![](media/SystemInfo-upd.png)

## <a name="final-check"></a>最終確認

OS、ハードウェア、および互換性の要件がすべて満たされている場合は、 **コントロールパネルに **Hyper-v** が表示されます。Windows の機能の有効化または無効化を行うと**、2 つのオプションが表示されます。

1. Hyper-V プラットフォーム
1. Hyper-V 管理ツール

![](media/hyper_v_feature_screenshot.png)

> [!NOTE] **コントロールパネルに、**Hyper-v** ではなく、**Windows ハイパーバイザープラットフォーム** が表示される場合:Windows の機能の有効化または無効化を行うと**、お使いのシステムには Hyper-V との互換性がない可能性があります。その後、上記の要件をクロスチェックします。
>
>既存の Hyper-V ホストで **systeminfo** を実行すると、Hyper-V の要件セクションでは、次の情報を読み取ります。
>
>```
>Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
>```
