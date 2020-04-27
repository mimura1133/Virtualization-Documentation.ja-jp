---
title: Hyper-V のプレリリース機能を試す
description: Hyper-V のプレリリース機能を試す
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
ms.openlocfilehash: 725466f657ae8fc4f14813822e90657e12d26fa6
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439559"
---
# <a name="try-pre-release-features-for-hyper-v"></a>Hyper-V のプレリリース機能を試す

> この記事は暫定的な内容であり、変更される可能性があります。  
  プレリリースの仮想マシンは、Microsoft ではサポートされないため、開発またはテスト環境でのみ使用してください。

Windows Server 2016 Technical Preview で Hyper-V のプレリリース機能にいち早くアクセスし、開発またはテスト環境で試すことができます。 最新の Hyper-V 機能を一足先に確認し、早めにフィードバックをお送りいただいて、製品の形成にご協力ください。

プレリリースとして作成する仮想マシンの場合、ビルド間の互換性はなく、今後のサポートは受けられません。  運用環境では使用しないでください。

非運用環境でのみの使用に限られる理由は他にもあります。理由は次のとおりです。

* プレリリース仮想マシンには上位互換性はありません。 これらの仮想マシンを新しい構成バージョンにアップグレードすることはできません。
* プレリリース仮想マシンには、ビルド間で一貫した定義はありません。 ホスト オペレーティング システムを更新すると、既存のプレリリース仮想マシンとホストとの互換性がなくなる可能性があります。 これらの仮想マシンが起動しなくなるか、あるいは、最初は動作しているように見えても後で重大な互換性の問題が発生する場合があります。
* プレリリース仮想マシンをビルドの異なるホストにインポートする場合、結果は予測できません。 プレリリース仮想マシンを別のホストに移動することはできます。 ただし、このシナリオの動作が期待できるのは、両方のホストが同じビルドを実行している場合のみです。

## <a name="create-a-pre-release-virtual-machine"></a>プレリリース仮想マシンの作成

Windows Server 2016 Technical Preview を実行する Hyper-V ホスト上にプレリリース仮想マシンを作成することができます。

1. Windows デスクトップ上で、[スタート] ボタンをクリックし、**Windows PowerShell** という名前の一部を入力します。
2. **[Windows PowerShell]** を右クリックし、 **[管理者として実行]** を選択します。
3. -Prerelease フラグの付いた [New-VM](https://docs.microsoft.com/powershell/module/hyper-v/new-vm?view=win10-ps) コマンドレットを使用して、プレリリース仮想マシンを作成します。 たとえば、VM 名が作成する仮想マシンの名前である場合は、以下のコマンドを実行します。

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
-Prerelease フラグを使用できる他の例を以下に示します。
 - 既存の仮想ハード ディスクまたは新しいハード ディスクを使用する仮想マシンを作成する場合は、[Windows Server 2016 Technical Preview における Hyper-V での仮想マシンの作成](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/Create-a-virtual-machine-in-Hyper-V#BKMK_PowerShell)に関するページの PowerShell の例を参照してください。
 - オペレーティング システム イメージから起動する新しい仮想ハード ディスクを作成する場合は、「[Windows 仮想マシンを Windows 10 の Hyper-V に展開する](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/create-virtual-machine)」の PowerShell の例を参照してください。

 これらの記事に示されている例は、Windows 10 または Windows Server 2016 Technical Preview を実行する Hyper-V ホストで使用できます。 ただし、現時点で -Prerelease フラグを使用できるのは、Windows Server 2016 Technical Preview を実行する Hyper-V ホスト上にプレリリース仮想マシンを作成する場合のみです。

## <a name="see-also"></a>関連項目
-  [仮想化に関するブログ](https://techcommunity.microsoft.com/t5/Virtualization/bg-p/Virtualization) - 使用可能なプレリリース機能とその機能を試す方法を確認できます。
- [サポートされている仮想マシンの構成バージョン](https://docs.microsoft.com/windows-server/virtualization/hyper-v/deploy/Upgrade-virtual-machine-version-in-Hyper-V-on-Windows-or-Windows-Server#BKMK_SupportedConfigVersions) - 仮想マシン構成バージョンと Microsoft でサポートされているバージョンの確認方法を学習します。
