---
title: 入れ子になった仮想化
description: 入れ子になった仮想化
keywords: Windows 10, Hyper-V
author: johncslack
ms.date: 12/18/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: f819ac04773188525af202d370ba271a2d93e259
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439349"
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>入れ子になった仮想化による仮想マシンでの Hyper-V の実行

入れ子になった仮想化は、Hyper-V 仮想マシン (VM) 内での Hyper-V の実行を可能にする機能です。 これは、仮想マシンで Visual Studio 電話エミュレーターを実行する場合や、通常は複数のホストが必要な構成のテストを行う場合に便利です。

![](./media/HyperVNesting.png)

## <a name="prerequisites"></a>前提条件

* Hyper-V ホストとゲストの両方が Windows Server 2016/Windows 10 Anniversary Update 以降であること。
* VM 構成バージョン 8.0 以上。
* VT-x および EPT テクノロジを使用する Intel プロセッサ。入れ子は現在 **Intel のみ**でサポートされています。
* 第 2 レベルの仮想マシンの仮想ネットワークとは、いくつかの違いがあります。 入れ子になった仮想マシン ネットワークに関する記述をご覧ください。


## <a name="configure-nested-virtualization"></a>入れ子になった仮想化の構成

1. 仮想マシンを作成します。 必要な OS と VM のバージョンについては、前述の前提条件をご覧ください。
2. 仮想マシンがオフ状態のときに、次のコマンドを物理的 Hyper-V ホスト上で実行します。 これで、この仮想マシンに対して入れ子になった仮想化が有効になります。

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 仮想マシンを開始します。
4. Hyper-V を仮想マシン内にインストールします。方法は物理サーバーの場合と同様です。 Hyper-V のインストールの詳細については、[Hyper-V のインストール](../quick-start/enable-hyper-v.md)に関するページを参照してください。

## <a name="disable-nested-virtualization"></a>入れ子になった仮想化の無効化
停止している仮想マシンに対して、入れ子になった仮想化を無効にするには、次の PowerShell コマンドを使用します。
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## <a name="dynamic-memory-and-runtime-memory-resize"></a>動的メモリとランタイム メモリ サイズ変更
Hyper-V が仮想マシンの中で実行されているときに、仮想マシンのメモリを調整するには、仮想マシンをオフにする必要があります。 つまり、動的メモリが有効になっている場合でも、メモリの量が変動することはありません。 動的メモリが有効になっていない仮想マシンでは、仮想マシンがオンの状態でメモリの量を調整しようとすると失敗します。 

なお、入れ子になった仮想化を有効にするだけでは、動的メモリやランタイム メモリ サイズ変更への影響はありません。 これらに対応できなくなるのは、Hyper-V が VM の中で実行中のときに限られます。

## <a name="networking-options"></a>ネットワーク オプション

入れ子になった仮想マシンのネットワーキングには、2 つのオプションがあります。 

1. MAC アドレスのスプーフィング
2. NAT ネットワーク

### <a name="mac-address-spoofing"></a>MAC アドレスのスプーフィング
2 つの仮想スイッチを通じてネットワーク パケットをルーティングするには、仮想スイッチの最初の (L1) レベルで MAC アドレスのスプーフィングを有効にする必要があります。 この処理は、次の PowerShell コマンドを使って完了できます。

``` PowerShell
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="network-address-translation-nat"></a>ネットワーク アドレス変換 (NAT)
2 番目のオプションは、ネットワーク アドレス変換 (NAT) に依存します。 この方法は、MAC アドレスのスプーフィングを利用できない場合 (パブリック クラウド環境など) に最適です。

最初に、仮想 NAT スイッチをホストの仮想マシン ("中間"の VM) に作成する必要があります。 IP アドレスはただの一例で、環境によって異なることに注意してください。

``` PowerShell
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```

次に、IP アドレスをネット アダプターに割り当てます。

``` PowerShell
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```

入れ子になった仮想マシンごとに 1 つの IP アドレスと 1 つのゲートウェイが割り当てられている必要があります。 ゲートウェイ IP は、前の手順で説明した NAT アダプターをポイントする必要があります。 必要に応じて、DNS サーバーも割り当てます。

``` PowerShell
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="how-nested-virtualization-works"></a>入れ子になった仮想化のしくみ

最新のプロセッサには、仮想化をさらに高速かつ安全にするためのハードウェア機能が含まれています。 Hyper-V は、これらのプロセッサ拡張機能に依存して仮想マシンを実行しています (Intel VT-x や AMD-V など)。 通常、Hyper-V が起動すると、他のソフトウェアがこれらのプロセッサ機能を使用できなくなります。  これは、ゲスト仮想マシンによる Hyper-V の実行を回避します。

入れ子になった仮想化では、このハードウェア サポートをゲスト仮想マシンでも利用できます。

次の図は、入れ子になっていない Hyper-V を示しています。  Hyper-V ハイパーバイザーは、ハードウェア仮想化機能 (オレンジ色の矢印) を完全に制御し、ゲスト オペレーティング システムには公開しません。

![](./media/HVNoNesting.PNG)

これに対し、次の図は、入れ子になった仮想化が有効な状態の Hyper-V を示しています。 この場合、Hyper-V は、ハードウェア仮想化拡張機能をその仮想マシンに公開します。 入れ子が有効になると、ゲスト仮想マシンは独自のハイパーバイザーをインストールし、独自のゲスト VM を実行します。

![](./media/HVNesting.png)

## <a name="3rd-party-virtualization-apps"></a>サード パーティの仮想化アプリ

Hyper-V 以外の仮想化アプリケーションは Hyper-V 仮想マシンではサポートされず、ほとんどの場合は実行に失敗します。 これには、ハードウェア仮想化拡張機能を必要とするソフトウェアも含まれます。
