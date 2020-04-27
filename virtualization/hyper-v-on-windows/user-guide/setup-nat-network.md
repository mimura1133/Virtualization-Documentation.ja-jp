---
title: NAT ネットワークのセットアップ
description: NAT ネットワークのセットアップ
keywords: Windows 10, Hyper-V
author: jmesser81
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
ms.openlocfilehash: 1652c3bcb32ddbc4e05e8821d0e646a76a2fd4f0
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "78853966"
---
# <a name="set-up-a-nat-network"></a>NAT ネットワークのセットアップ

Windows 10 Hyper-V では、仮想ネットワークのネイティブ ネットワーク アドレス変換 (NAT) を可能です。

このガイドでは、以下について説明します。
* NAT ネットワークを作成する
* 既存の仮想マシンを新しいネットワークに接続する
* 仮想マシンが正しく接続されていることを確認する

要件:
* Windows 10 Anniversary Update 以降
* Hyper-V が有効になっている (詳しくは[こちら](../quick-start/enable-hyper-v.md)をご覧ください)

> **注:** 現時点では、ホストごとに 1 つの NAT ネットワークのみを作成できます。 Windows NAT (WinNAT) の実装、機能、制限事項のについて詳しくは、「[WinNAT 機能と制限事項のブログ (ブログの投稿)](https://techcommunity.microsoft.com/t5/Virtualization/Windows-NAT-WinNAT-Capabilities-and-limitations/ba-p/382303)」をご覧ください

## <a name="nat-overview"></a>NAT 概要
NAT は、ホスト コンピューターの IP アドレスと内部 Hyper-V 仮想スイッチを通じてポートを利用することで、ネットワーク リソースへのアクセスを仮想マシンに与えます。

ネットワーク アドレス変換 (NAT) は IP アドレスを節約するように設計されているネットワーキング モードです。外部の IP アドレスとポートをより大きな内部 IP アドレス セットにマッピングします。  基本的に、NAT はフロー テーブルを利用し、外部 (ホスト) IP アドレスとポート番号からネットワーク上のエンドポイント (仮想マシン、コンピューター、コンテナーなど) に関連付けられている正しい内部 IP アドレスにトラフィックを送ります

また、NAT を利用すれば、同じ (内部) 通信ポートを必要とするアプリケーションを複数の仮想マシンがホストできます。アプリケーションを一意の外部ポートにマッピングします。

以上の理由から、NAT ネットワーキングはコンテナー技術として一般的になっています (「[コンテナーのネットワーク](https://docs.microsoft.com/virtualization/windowscontainers/container-networking/architecture)」参照)。


## <a name="create-a-nat-virtual-network"></a>NAT 仮想ネットワークを作成する
新しい NAT ネットワークの設定方法を段階的に確認しましょう。

1. 管理者として PowerShell コンソールを開きます。  

2. 内部スイッチを作成する

    ```powershell
    New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
    ```

3. 作成した仮想スイッチのインターフェイス インデックスを検索します。

    インターフェイス インデックスを検索するには、`Get-NetAdapter` を実行します。

    出力は以下のようになります。

    ```console
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    内部スイッチには、「`vEthernet (SwitchName)`」のような名前が与えられ、`Hyper-V Virtual Ethernet Adapter` のインターフェイスの説明が付きます。 次の手順で使用できるように、`ifIndex` を書き留めます。

4. [New-NetIPAddress](https://docs.microsoft.com/powershell/module/nettcpip/New-NetIPAddress) を利用して NAT ゲートウェイを構成します。  

    汎用コマンドを次に示します。
    ```powershell
    New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
    ```

    ゲートウェイを構成するためには、ネットワークについていくつかの情報が必要です。  
    * **IPAddress** -- NAT ゲートウェイ IP は、NAT ゲートウェイ IP として使用する IPv4 または IPv6 アドレスを指定するものです。  
      一般的な形式は a.b.c.1 です (例: 172.16.0.1)。  最後の位置を 1 にする必要はありませんが、通常はそうなっています (プレフィックスの長さに基づきます)。

      一般的なゲートウェイ IP は 192.168.0.1 です。  

    * **PrefixLength** -- NAT Subnet Prefix Length により、NAT ローカル サブネット サイズが定義されます (サブネット マスク)。
      サブネット プレフィックスの長さは、0 ～ 32 の整数値になります。

      0 の場合、インターネット全体がマッピングされます。32 の場合、IP を 1 つだけマッピングできます。  一般的な値は 24 ～ 12 であり、NAT に接続する IP の数により決まります。

      一般的な PrefixLength は 24 です。これは 255.255.255.0 のサブネット マスクです。

    * **InterfaceIndex**: ifIndex は、前の手順で指定した、仮想スイッチのインターフェイス インデックスです。

    次を実行して、NAT ゲートウェイを作成します。

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
    ```

5. [New-NetNat](https://docs.microsoft.com/powershell/module/netnat/New-NetNat) を使用し、NAT ネットワークを構成します。  

    汎用コマンドを次に示します。

    ```powershell
    New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
    ```

    ゲートウェイを構成するには、ネットワークと NAT ゲートウェイに関する情報を提供する必要があります。  
    * **Name** -- NATOutsideName は NAT ネットワークの名前です。  NAT ネットワークを削除するとき、これを使用します。

    * **InternalIPInterfaceAddressPrefix** -- NAT サブネット プレフィックスは、上記の NAT ゲートウェイ IP プレフィックスと上記の NAT Subnet Prefix Length の両方を表すものです。

    一般的な形式は a.b.c.0/NAT Subnet Prefix Length です

    上記から、この例では、192.168.0.0/24 を使用します。

    この例では、次を実行して NAT ネットワークを設定します。

    ```powershell
    New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
    ```

お疲れさまでした。  これで仮想 NAT ネットワークができました。  NAT ネットワークに仮想マシンを追加するには、[ここの指示](#connect-a-virtual-machine)に従ってください。

## <a name="connect-a-virtual-machine"></a>仮想マシンを接続する

仮想マシンを新しい NAT ネットワークに接続するには、VM 設定メニューを利用し、[NAT ネットワークの設定](#create-a-nat-virtual-network)の最初の手順で作成した内部スイッチを仮想マシンに接続します。

WinNAT は単独でエンドポイント (例: VM) に IP アドレスを割り当てることがないため、VM 自体から手動でこの作業を行う必要があります。つまり、NAT 内部プレフィックスの範囲内で IP アドレスを設定し、既定のゲートウェイ IP アドレスを設定し、DNS サーバー情報を設定します。 ただし、エンドポイントがコンテナーにアタッチされる場合には、注意が必要です。 この場合、ホスト ネットワーク サービス (HNS) はホスト コンピューティング サービス (HCS) を割り当てて使用し、IP アドレス、ゲートウェイ IP、DNS 情報をコンテナーに直接割り当てます。


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>構成例:NAT ネットワークへの VM とコンテナーのアタッチ

_複数の VM とコンテナーを 1 つの NAT にアタッチする場合は、NAT 内部サブネットのプレフィックスが別のアプリケーションまたはサービス (例: Docker for Windows や Windows コンテナー – HNS) によって割り当てられている IP の範囲を網羅するのに十分な大きさになっている必要があります。この場合、IP とネットワーク構成のアプリケーション レベルの割り当てか、手動構成のいずれかが必要になります。手動構成の場合は、管理者が操作を行い、同一ホスト上の既存の IP 割り当てを再使用しないようにする必要があります。_

### <a name="docker-for-windows-linux-vm-and-windows-containers"></a>Docker for Windows (Linux VM) と Windows コンテナー
次のソリューションでは、Docker for Windows (Linux コンテナーを実行する Linux VM) と Windows コンテナーの両方が、別の内部 vSwitch が使用する同じ WinNAT インスタンスを共有できます。 接続は Linux コンテナーと Windows コンテナーの両方で機能します。

ユーザーは “VMNAT” という名前の内部 vSwitch を通じて VM を NAT ネットワークにしており、次に Docker エンジンを使用して Windows コンテナー機能をインストールしようと考えています
```
PS C:\> Get-NetNat “VMNAT”| Remove-NetNat (this will remove the NAT but keep the internal vSwitch).
Install Windows Container Feature
DO NOT START Docker Service (daemon)
Edit the arguments passed to the docker daemon (dockerd) by adding –fixed-cidr=<container prefix> parameter. This tells docker to create a default nat network with the IP subnet <container prefix> (e.g. 192.168.1.0/24) so that HNS can allocate IPs from this prefix.
PS C:\> Start-Service Docker; Stop-Service Docker
PS C:\> Get-NetNat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> Start-Service docker
```
Docker/HNS は Windows コンテナーに IP アドレスを割り当てます。管理者は、その 2 つの別のセットから IP を VM に割り当てます。

ユーザーは実行中の docker エンジンを使用して Windows コンテナー機能をインストールしました。次に、NAT ネットワークに VM を接続しようと考えています
```
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
PS C:\> Get-NetNat | Remove-NetNat (this will remove the NAT but keep the internal vSwitch)
Edit the arguments passed to the docker daemon (dockerd) by adding -b “none” option to the end of docker daemon (dockerd) command to tell docker not to create a default NAT network.
PS C:\> New-ContainerNetwork –name nat –Mode NAT –subnetprefix <container prefix> (create a new NAT and internal vSwitch – HNS will allocate IPs to container endpoints attached to this network from the <container prefix>)
PS C:\> Get-Netnat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> New-VirtualSwitch -Type internal (attach VMs to this new vSwitch)
PS C:\> Start-Service docker
```
Docker/HNS は Windows コンテナーに IP アドレスを割り当てます。管理者は、その 2 つの別のセットから IP を VM に割り当てます。

最後に、2 つの内部 VM スイッチを設定し、そのスイッチ間で共有する NetNat を 1 つ設定する必要があります。

## <a name="multiple-applications-using-the-same-nat"></a>複数のアプリケーションで同じ NAT を使用する

複数のアプリケーションまたはサービスで同じ NAT を使用することが必要になる場合もあります。 そのような場合、複数のアプリケーションまたはサービスでより大きな NAT 内部サブネット プレフィックスを使えるように、次のワークフローに従う必要があります。

**_例として、同じホストで Docker 4 Windows、Docker Beta、Linux VM を Windows コンテナー機能で共存させる方法について説明します。このワークフローは変更される場合があります。_**

1. C:\> net stop docker
2. Stop Docker4Windows MobyLinux VM
3. PS C:\>Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\>Get-NetNat | Remove-NetNat  
   *既存のコンテナー ネットワークが削除されます (つまり、vSwitch の削除、NetNat の削除、クリーンアップ)*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (このサブネットは Windows コンテナー機能で使用されます) *内部 vSwitch が「nat」という名前で作成されます*  
   *NAT ネットワークが “nat” という名前で作成されます。IP プレフィックスは 10.0.76.0/24 です。*  

6. Remove-NetNAT  
   *DockerNAT と nat NAT ネットワークの両方が削除されます (内部 vSwitches は残ります)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (これは D4W とコンテナーが共有するためのより大きな NAT ネットワークを作成します)  
   *NAT ネットワークが「DockerNAT」という名前で作成されます。より大きなプレフィックスは 10.0.0.0/17 です。*  

8. Run Docker4Windows (MobyLinux.ps1)  
   *内部 vSwitch DockerNAT が作成されます*  
   *NAT ネットワークが「DockerNAT」という名前で作成されます。IP プレフィックスは 10.0.75.0/24 です。*  

9. Net start docker  
   *Docker は、ユーザー定義の NAT ネットワークを Windows コンテナーを接続するための既定として使用します。*  

最終的に、2 つの内部 vSwitches が与えられるはずです。1 つは「DockerNAT」という名前で、もう 1 つは「nat」という名前です。 Get-NetNat を実行すると、NAT ネットワークが 1 つだけ確定します (10.0.0.0/17)。 Windows コンテナーの IP アドレスが Windows Host Network Service (HNS) により 10.0.76.0/24 サブネットから割り当てられます。 既存の MobyLinux.ps1 スクリプトに基づき、Docker 4 Windows の IP アドレスが 10.0.75.0/24 サブネットから割り当てられます。


## <a name="troubleshooting"></a>トラブルシューティング

### <a name="multiple-nat-networks-are-not-supported"></a>サポートされない複数の NAT ネットワーク  
このガイドでは、ホストに他に NAT がないものと想定しています。 ただし、アプリケーションまたはサービスで NAT の使用が必要であり、設定の一環として作成される場合があります。 Windows (WinNAT) でサポートされる内部 NAT サブネット プレフィックスは 1 つだけです。複数の NAT を作成しようとすると、システムが不明状態になります。

この問題があるかを確認するには、NAT が 1 つだけであることを確認します。
```powershell
Get-NetNat
```

NAT が既に存在する場合はそれを削除します。
```powershell
Get-NetNat | Remove-NetNat
```
アプリケーションまたは機能 (例: Windows コンテナー) ごとに「内部」vmSwitch が 1 つだけ設定されていることを確認します。 vSwitch の名前を記録します
```powershell
Get-VMSwitch
```

以前の NAT のプライベート IP アドレス (例: NAT の既定のゲートウェイ IP Address - 通常 _x_._y_._z_.1) がアダプターに割り当てられていないか確認します
```powershell
Get-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)"
```

古いプライベート IP アドレスが使用されている場合、それを削除してください。
```powershell
Remove-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)" -IPAddress <IPAddress>
```

**複数の NAT の削除**  
複数の NAT ネットワークが誤って作成されてしまうと報告されています。 これは、最新のビルド (Windows Server 2016 Technical Preview 5 や Windows 10 Insider Preview ビルドなど) のバグが原因です。 複数の NAT ネットワークがある場合は、docker ネットワーク ls や Get-ContainerNetwork を実行した後に、管理者特権の PowerShell から次を実行してください。

```powershell
$keys = Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
Remove-NetNat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```

次のコマンドを実行する前に、オペレーティング システムを再起動します (`Restart-Computer`)

```powershell
Get-NetNat | Remove-NetNat
Set-Service docker -StartupType Automatic
Start-Service docker 
```

NAT 環境を必要に応じて再構築するには、セットアップ ガイド「[複数のアプリケーションで同じ NAT を使用する](#multiple-applications-using-the-same-nat)」をご覧ください。 

## <a name="references"></a>参考資料
NAT ネットワークの詳細は[ここ](https://en.wikipedia.org/wiki/Network_address_translation)を参照してください。
