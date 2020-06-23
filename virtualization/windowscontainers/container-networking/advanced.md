---
title: Windows コンテナー ネットワーク
description: Windows コンテナー用の高度なネットワーク。
keywords: Docker, コンテナー
author: jmesser81
ms.date: 03/27/2018
ms.topic: how-to
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 5278d1d0aefadf81905c843609e1c922f5ca446a
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192689"
---
# <a name="advanced-network-options-in-windows"></a>Windows での高度なネットワーク オプション

Windows に固有の機能を活用するために、いくつかのネットワーク ドライバーのオプションがサポートされています。

## <a name="switch-embedded-teaming-with-docker-networks"></a>Docker ネットワークを使用したスイッチ埋め込みチーミング

> すべてのネットワーク ドライバーに適用されます。

Docker で使用するコンテナー ホスト ネットワークを作成するときに、`-o com.docker.network.windowsshim.interface` オプションで複数のネットワーク アダプターを (コンマで区切って) 指定することにより、[スイッチ埋め込みチーミング](https://docs.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/RDMA-and-Switch-Embedded-Teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set)を活用できます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>ネットワークの VLAN ID の設定

> transparent および l2bridge ネットワーク ドライバーに適用されます。

ネットワークの VLAN ID を設定するには、`docker network create` コマンドに `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` オプションを使用します。 たとえば、次のコマンドを使用して、VLAN ID が 11 である透過ネットワークを作成できます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
ネットワークの VLAN ID を設定すると、そのネットワークに接続されるあらゆるコンテナー エンドポイントの VLAN 分離が設定されます。

> タグ付けされたすべてのトラフィックを正しい VLAN 上でアクセス モードの vNIC (コンテナー エンドポイント) ポートを持つ vSwitch によって処理するためには、(物理的) ホスト ネットワーク アダプターがトランク モードであることを確認します。

## <a name="specify-outboundnat-policy-for-a-network"></a>ネットワークの OutboundNAT ポリシーを指定する

> L2bridge ネットワークに適用されます

通常、 `l2bridge` を使用してコンテナーネットワークを作成すると、 `docker network create` コンテナーエンドポイントには、HNS outboundnat ポリシーが適用されないため、コンテナーが外部に届かなくなります。 ネットワークを作成する場合は、オプションを使用して、 `-o com.docker.network.windowsshim.enable_outboundnat=<true|false>` コンテナーに外部へのアクセスを許可するために OutboundNAT HNS ポリシーを適用することができます。

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true MyL2BridgeNetwork
```

NAT'ing を発生させたくない場所に、一連の宛先 (コンテナーからコンテナーへの接続が必要な場合など) がある場合は、次のようにして、追加の設定を指定する必要があります。

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true -o com.docker.network.windowsshim.outboundnat_exceptions=10.244.10.0/24
```

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>HNS サービスへのネットワーク名の指定

> すべてのネットワーク ドライバーに適用されます。

通常、`docker network create` を使ってコンテナー ネットワークを作成した場合、使用したネットワーク名は Docker サービスでは使われますが、HNS サービスでは使われません。 ネットワークを作成する場合、`docker network create` コマンドで `-o com.docker.network.windowsshim.networkname=<network name>` オプションを使うと、HNS サービスから提供された名前を指定できます。 たとえば、次のコマンドを使用すると、HNS サービスに対して指定された名前を使って透過ネットワークを作成できます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>特定のネットワーク インターフェイスへのネットワークのバインド

> 'nat' を除くすべてのネットワーク ドライバーに適用されます。

ネットワーク (Hyper-V 仮想スイッチを通じて接続された) を特定のネットワーク インターフェイスにバインドするには、`docker network create` コマンドで `-o com.docker.network.windowsshim.interface=<Interface>` オプションを使います。 たとえば、次のコマンドを使うと、"Ethernet 2" ネットワーク インターフェイスに接続された透過ネットワークを作成できます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 注: *com.docker.network.windowsshim.interface* の値は、ネットワーク アダプターの*名前*です。これは次のコマンドを使って確認できます。

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>ネットワークの DNS サフィックスや DNS サーバーの指定

> すべてのネットワーク ドライバーに適用されます。

ネットワークの DNS サフィックスを指定するには、オプション `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` を使い、ネットワークの DNS サーバーを指定するには、オプション `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` を使います。 たとえば、次のコマンドを使用すると、ネットワークの DNS サフィックスが "example.com" に設定され、ネットワークの DNS サーバーが 4.4.4.4 と 8.8.8.8 に設定されます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

詳細については、[この記事](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/)を参照してください。

## <a name="tips--insights"></a>ヒントとインサイト
ここでは、Windows コンテナーのネットワークに関してコミュニティに寄せられたよくある質問をまとめた、便利なヒントと情報の一覧を示します。

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS では、コンテナーのホスト コンピューターで IPv6 が有効になっている必要がある
[KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217) でも説明されているように、HNSでは、Windows コンテナー ホストで IPv6 が有効になっている必要があります。 次のようなエラーが発生した場合は、ホスト コンピューターで IPv6 が無効になっている可能性があります。
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
この問題を自動的に検出/防止するために、プラットフォームの変更に取り組んでいます。 現在、次の回避策によって、ホスト コンピューターで確実に IPv6 を有効にすることができます。

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Linux Containers on Windows

**最新情報:** 現在、_Moby Linux VM を使用せずに_ Linux と Windows のコンテナーをサイド バイ サイドで実行できるよう取り組んでいます。 詳しくは、[Linux Containers on Windows (LCOW) に関するこちらのブログ記事](https://blog.docker.com/2017/11/docker-for-windows-17-11/)をご覧ください。 [開始](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux)方法を次に示します。
> 注意: LCOW は Moby Linux VM に代わるものであり、既定の HNS "nat" 内部 vSwitch を使用します。

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-ce"></a>Moby Linux VM では、Docker for Windows ([Docker CE](https://www.docker.com/community-edition) の製品) と共に DockerNAT スイッチを使用しています。

Windows 10 上の Docker for Windows (Docker CE エンジン用の Windows ドライバー) は、DockerNAT という内部 vSwitch を使って Moby Linux VM をコンテナー ホストに接続します。 Windows で Moby Linux VM を使用している開発者は、ホストが、HNS サービスによって作成される "nat" vSwitch (Windows コンテナーで使用される既定のスイッチ) ではなく、DockerNAT vSwitch を使用していることに注意する必要があります。



#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>仮想コンテナー ホストでの IP の割り当てに DHCP を使用するには、MACAddressSpoofing を有効にする

コンテナーのホストが仮想化されており、IP の割り当てに DHCP を使用したい場合は、仮想マシンのネットワーク アダプターで MACAddressSpoofing を有効にする必要があります。 それ以外の場合、HYPER-V ホストでは、複数の MAC アドレスを持つ VM 内のコンテナーからのネットワーク トラフィックをブロックします。 次の PowerShell コマンドを使用して、MACAddressSpoofing を有効にすることができます。
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
VMware をハイパーバイザーとして実行する場合は、正しく実行できるように無作為検出モードを有効にする必要があります。 詳しくは、[こちら](https://kb.vmware.com/s/article/1004099)をご覧ください。


#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>1 つのコンテナー ホストで複数の透過ネットワークを作成する
複数の透過ネットワークを作成する場合は、外部 Hyper-V 仮想スイッチをバインドする (仮想) ネットワーク アダプターを指定する必要があります。 ネットワークのインターフェイスを指定するには、次の構文を使います。
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME>

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>静的 IP 割り当てを使用する場合は、必ず *--subnet* と *--gateway* を指定する
IP を静的に割り当てる場合、ネットワークの作成時に、*--subnet* および *--gateway* パラメーターが指定されていることを最初に確認する必要があります。 サブネットとゲートウェイの IP アドレスは、コンテナー ホスト (つまり、物理ネットワーク) のネットワーク設定と同じにする必要があります。 たとえば、静的 IP の割り当てを使用して、透過ネットワークを作成し、そのネットワーク上でエンドポイントを実行する方法は次のようになります。

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>DHCP による IP 割り当ては L2Bridge ネットワークでサポートされない
l2bridge ドライバーを使って作成されたコンテナー ネットワークでは、静的 IP の割り当てのみがサポートされます。 前述のように、静的 IP の割り当て用に構成されたネットワークを作成するには、必ず *--subnet* と *--gateway* パラメーターを使用してください。

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>外部 vSwitch を活用するネットワークでは独自のネットワーク アダプターが必要である
同じコンテナー ホストで、接続に外部 vSwitch を使用する複数のネットワーク (透過、L2 ブリッジ、L2 透過など) が作成されている場合、それぞれに独自のネットワーク アダプターが必要であることに注意してください。

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>停止しているコンテナーと実行中のコンテナーでの IP の割り当て
静的な IP の割り当ては、コンテナーのネットワーク アダプターに対して直接行い、コンテナーが停止状態にある場合にのみ実行する必要があります。 コンテナーの実行中は、コンテナーのネットワーク アダプターの "ホット アド" またはネットワーク スタックに対する変更は実行できません (Windows Server 2016 の場合)。

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>既存の vSwitch (Docker には表示されない) が透過ネットワークの作成をブロックする場合がある
透過ネットワークの作成中にエラーが発生した場合、Docker で自動検出されなかった外部 vSwitch がシステムに存在し、それが透過ネットワークとコンテナー ホストの外部ネットワーク アダプターの関連付けを阻んでいる可能性があります。

透過ネットワークを作成するとき、Docker はネットワークの外部 vSwitch を作成し、スイッチと (外部) ネットワーク アダプターの関連付けを試行します。アダプターは VM ネットワーク アダプターまたは物理ネットワーク アダプターになります。 vSwitch が既にコンテナー ホストで作成されており、*Docker に表示される*場合、Windows Docker エンジンは新しいスイッチを作成せず、そのスイッチを使用します。 ただし、vSwitch が帯域外で作成され (すなわち、Hyper-V Manager または PowerShell を利用し、コンテナー ホストで作成され)、Docker にはまだ表示されていない場合、Windows Docker エンジンは新しい vSwitch の作成を試みます。新しいスイッチをコンテナー ホストの外部ネットワーク アダプターに接続することはできません (ネットワーク アダプターが帯域外で作成されたスイッチに接続されるため)。

たとえば、最初に Docker サービスの実行中にホストで新しい vSwitch を作成し、次に透過ネットワークを作成しようとすると、この問題が発生します。 その場合、Docker はユーザーが作成したスイッチを認識せず、透過ネットワークのために新しい vSwitch を作成します。

この問題は 3 つの方法で解決できます。

* 帯域外で作成された vSwitch はもちろん削除できます。削除後、Docker は新しい vSwitch を作成し、それを問題なくホスト ネットワーク アダプターに接続できます。 この方法を選択する前に、帯域外 vSwitch が他のサービス (Hyper-V など) により利用されていないことを確認します。
* あるいは、帯域外で作成された外部 vSwitch を使用する場合、Docker と HNS サービスを再起動し、*Docker にスイッチが表示される*ようにします。
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* もう 1 つの選択肢は、'-o com.docker.network.windowsshim.interface' オプションを利用し、透過ネットワークの外部 vSwitch をコンテナー ホストでまだ使用されていない特定のネットワーク アダプター (すなわち、帯域外で作成された vSwitch で利用されているネットワーク アダプター以外のネットワーク アダプター) に関連付けることです。 「-O」オプションについては、このドキュメントの「 [1 つのコンテナーホストに複数の透過的ネットワークを作成する](advanced.md#creating-multiple-transparent-networks-on-a-single-container-host)」で詳しく説明します。


## <a name="windows-server-2016-work-arounds"></a>Windows Server 2016 の回避策

マイクロソフトでは新しい機能の追加と開発の促進を継続していますが、これらの機能の一部は以前のプラットフォームには移植されません。 最善の策として、Windows 10 および Windows Server の最新の更新プログラムを適用することをお勧めします。  以下のセクションでは、Windows Server 2016 と以前のバージョンの Windows 10 (つまり 1704 Creators Update より前のバージョン) に適用されるいくつかの回避策と注意事項を示します。

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>WS2016 コンテナー ホスト上の複数の NAT ネットワーク

新しい NAT ネットワークのパーティションは、大規模な内部 NAT ネットワーク プレフィックスの下で作成する必要があります。 PowerShell から次のコマンドを実行し、"InternalIPInterfaceAddressPrefix" フィールドを参照すると、プレフィックスが見つかります。

```
PS C:\> Get-NetNAT
```

たとえば、ホストの NAT ネットワークの内部プレフィックスを 172.16.0.0/16 にします。 その場合、*172.16.0.0/16 プレフィックスのサブセットである限り*、Docker を利用して追加 NAT ネットワークを作成できます。 たとえば、IP プレフィックス 172.16.1.0/24 (ゲートウェイ、172.16.1.1) と 172.16.2.0/24 (ゲートウェイ、172.16.2.1) で 2 つの NAT ネットワークを作成できます。

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

新しく作成されたネットワークは次を利用して一覧表示できます。
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) を利用し、コンテナー ネットワークをそれらのネットワークを使用するコンテナー/サービスと一緒に定義し、構成できます。 コンテナーの接続先となるネットワークを定義するとき、Compose 'ネットワーク' キーが最上位キーとして使用されます。 たとえば、下の構文では、Docker により作成された既存の NAT ネットワークが特定の Compose ファイルに定義されているすべてのコンテナー/サービスの '既定の' ネットワークになります。

```
networks:
 default:
  external:
   name: "nat"
```

同様に、次の構文を利用し、カスタム NAT ネットワークを定義できます。

> 注: 下の例で定義されている 'カスタム NAT ネットワーク' はコンテナー ホストの既存の NAT 内部プレフィックスのパーティションとして定義されています。 詳細については、上記の「複数の NAT ネットワーク」セクションを参照してください。

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Docker Compose を利用し、コンテナー ネットワークを定義/構成する方法については、「[Compose ファイル参照](https://docs.docker.com/compose/compose-file/)」を参照してください。
