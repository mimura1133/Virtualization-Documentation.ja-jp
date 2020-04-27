---
title: サポートされる Windows ゲスト
description: サポートされる Windows ゲスト
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 25c72b910af15fc0b498a5b2abce72d32e6d1efd
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439599"
---
# <a name="supported-windows-guests"></a>サポートされる Windows ゲスト

この記事では、Windows 上の Hyper-V でサポートされているオペレーティング システムの組み合わせを一覧に示します。  また、サポートされる統合サービスやその他のファクターの概要も紹介します。

Microsoft では、これらのホスト/ゲストの組み合わせをテスト済みです。  これらの組み合わせでの問題については、製品サポート サービスから対応を受けることができます。

Microsoft では、次のようにサポートを提供します。

* Microsoft のオペレーティング システムと統合サービスの問題は、Microsoft サポートによってサポートされます。

* オペレーティング システム ベンダーによって Hyper-V の実行が認定されている、他のオペレーティング システムで検出された問題については、ベンダーによってサポートが提供されます。

* 他のオペレーティング システムで見つかった問題は、Microsoft は、マルチ ベンダー サポート コミュニティに問題を送信する [TSANet](http://www.tsanet.org/)します。

サポートを受けるには、すべてのオペレーティング システム (ゲストとホスト) を最新の状態にしておく必要があります。  Windows Update で緊急更新プログラムがないか確認してください。

## <a name="supported-guest-operating-systems"></a>サポートされているゲスト オペレーティング システム

| ゲスト オペレーティング システム |  仮想プロセッサの最大数 | 説明 |
|:-----|:-----|:-----|
| Windows 10 | 32 |拡張セッション モードは、Windows 10 Home エディションでは機能しません |
| Windows 8.1 | 32 | |
| Windows 8 | 32 ||
| Windows 7 Service Pack 1 (SP 1) | 4 | Ultimate、Enterprise、および Professional Edition (32 ビットおよび 64 ビット)。 |
| Windows 7 | 4 | Ultimate、Enterprise、および Professional Edition (32 ビットおよび 64 ビット)。 |
| Windows Vista Service Pack 2 (SP2) | 2 | Business、Enterprise、および Ultimate (N および KN エディションを含む)。 |
| - | | |
| [Windows Server 半期チャネル](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2019 | 64 | |
| Windows Server 2016 | 64 | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 Service Pack 1 (SP 1) | 64 | Datacenter、Enterprise、Standard、および Web Edition。 |
| Windows Server 2008 Service Pack 2 (SP 2) | 4 | Datacenter、Enterprise、Standard、および Web Edition (32 ビットおよび 64 ビット)。 |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Essentials edition - 2、Standard edition - 4 | |

> Windows 10 は、Windows 8.1 と Windows Server 2012 R2 Hyper-V ホスト上でゲスト オペレーティング システムとして実行できます。

## <a name="supported-linux-and-free-bsd"></a>サポートされる Linux と FreeBSD

| ゲスト オペレーティング システム |  |
|:-----|:------|
| [CentOS と Red Hat Enterprise Linux ](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-CentOS-and-Red-Hat-Enterprise-Linux-virtual-machines-on-Hyper-V) | |
| [Hyper-V の Debian 仮想マシン](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Debian-virtual-machines-on-Hyper-V) | |
| [SUSE](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-SUSE-virtual-machines-on-Hyper-V) | |
| [Oracle Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Oracle-Linux-virtual-machines-on-Hyper-V)  | |
| [Ubuntu](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Ubuntu-virtual-machines-on-Hyper-V) | |
| [FreeBSD](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-FreeBSD-virtual-machines-on-Hyper-V) | |

過去のバージョンの Hyper-V におけるサポート情報などの詳細については、「[Hyper V の Linux と FreeBSD 仮想マシン](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows)」をご覧ください。
