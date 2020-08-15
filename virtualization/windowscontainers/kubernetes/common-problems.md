---
title: Kubernetes のトラブルシューティング
author: daschott
ms.author: daschott
ms.date: 08/13/2020
ms.topic: troubleshooting
description: Kubernetes の展開と Windows ノードの参加で発生する一般的な問題の解決方法。
keywords: kubernetes、linux、コンパイル
ms.openlocfilehash: f96d90f2ab4f7cdfea942badab8fb277c40cb621
ms.sourcegitcommit: aa139e6e77a27b8afef903fee5c7ef338e1c79d4
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/15/2020
ms.locfileid: "88251605"
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes のトラブルシューティング #
このページでは、Kubernetes のセットアップ、ネットワーク、および展開に関する一般的な問題について説明します。

> [!tip]
> PR を[ドキュメント リポジトリ](https://github.com/MicrosoftDocs/Virtualization-Documentation/)に投稿することで、よく寄せられる質問の項目をご提案ください。

このページは、次のカテゴリに分類されます。
1. [一般的な質問](#general-questions)
2. [一般的なネットワークエラー](#common-networking-errors)
3. [一般的な Windows エラー](#common-windows-errors)
4. [一般的な Kubernetes マスターエラー](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般的な質問 ##

### <a name="how-do-i-know-kubernetes-on-windows-completed-successfully"></a>Windows で Kubernetes が正常に完了したことを操作方法確認してください。 ###
ノードで実行されている flanneld ホストエージェントプロセスである kubelet、kube、および (ネットワークソリューションとして Flannel を選択した場合) が表示されます。 これに加えて、Windows ノードは Kubernetes クラスターに "Ready" と表示されます。

### <a name="can-i-configure-to-run-all-of-this-in-the-background"></a>これらのすべてをバックグラウンドで実行するようにを構成することはできますか。 ###
Kubernetes バージョン1.11 以降では、kubelet & kube をネイティブ [Windows サービス](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)として実行できます。 また、常に、 [nssm.exe](https://nssm.cc/) などの代替サービスマネージャーを使用して、これらのプロセス (flanneld、kubelet & kube) をバックグラウンドで常に実行することもできます。

## <a name="common-networking-errors"></a>一般的なネットワークエラー ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>ロードバランサーがクラスターノード間で一貫性のない方法で組み込まれる ###
Windows では、kube は、クラスター内のすべての Kubernetes サービスに対して、HNS ロードバランサーを作成します。 (既定値) kube 構成では、多くの (通常100を超える) ロードバランサーを含むクラスター内のノードは、使用可能な一時 TCP ポート ( 動的なポート範囲。既定では、ポート49152から 65535) が対象となります。 これは、(DSR 以外の) ロードバランサーごとに、各ノードに予約されているポートの数が多いためです。 この問題は、次のような kube のエラーによって発生する可能性があります。
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

ユーザーは、 [CollectLogs.ps1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) スクリプトを実行してファイルを調べることで、この問題を特定でき `*portrange.txt` ます。

また、は、 `CollectLogs.ps1` 一時的な TCP ポート範囲でポートプールの割り当ての可用性をテストし、で成功/失敗を報告するために、HNS 割り当てロジックを模倣し `reservedports.txt` ます。 このスクリプトでは、64の TCP 一時ポートの10の範囲 (HNS の動作をエミュレートするため) を予約し、予約の成功 & 失敗数をカウントしてから、割り当てられたポート範囲を解放します。 成功数が10未満の場合は、短期プールの空き領域が不足していることを示します。 では、使用可能な 64-ブロックポート予約の数に関する heuristical の概要もで生成され `reservedports.txt` ます。

この問題を解決するには、いくつかの手順を実行します。
1.  永続的なソリューションの場合は、kube の負荷分散を [DSR モード](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)に設定する必要があります。 DSR モードは完全に実装され、新しい [Windows Server Insider build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (またはそれ以降) でのみ使用できます。
2. 回避策として、ユーザーは、などのコマンドを使用して、使用可能な一時ポートの既定の Windows 構成を増やすこともでき `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>` ます。 *警告:* 既定の動的ポート範囲を上書きすると、非短期の範囲の使用可能な TCP ポートに依存するホスト上の他のプロセスやサービスに影響が生じる可能性があるため、この範囲は慎重に選択する必要があります。
3. 累積的な更新プログラム [KB4551853](https://support.microsoft.com/en-us/help/4551853) (およびすべての新しい累積的な更新プログラム) に含まれるインテリジェントなポートプールの共有を使用した DSR モード以外のロードバランサーには、スケーラビリティが強化されています。

### <a name="hostport-publishing-is-not-working"></a>HostPort の発行が機能していません ###
HostPort 機能を使用するには、CNI プラグインが [0.8.6](https://github.com/containernetworking/plugins/releases/tag/v0.8.6) リリース以降であること、および CNI 構成ファイルに次の機能が設定されていることを確認してください `portMappings` 。
```
"capabilities": {
    "portMappings":  true
}
```

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>"HnsCall が Win32 で失敗しました。ドライブに間違ったディスケットがあります" などのエラーが表示されます。 ###
このエラーは、古い HNS オブジェクトを破棄せずに、hns オブジェクトに対してカスタム変更を行ったり、HNS を変更する新しい Windows Update をインストールしたりするときに発生する可能性があります。 これは、更新前に以前に作成された HNS オブジェクトが、現在インストールされている HNS バージョンと互換性がないことを示します。

Windows Server 2019 (およびそれ以降) では、ユーザーは、HNS ファイルを削除することで、HNS オブジェクトを削除できます。
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

ユーザーは、互換性のない HNS エンドポイントまたはネットワークを直接削除できなければなりません。
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server バージョン1903のユーザーは、次のレジストリの場所に移動し、ネットワーク名で始まる Nic (やなど) を削除でき `vxlan0` `cbr0` ます。
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>Azure 上の Flannel ホスト-gw デプロイのコンテナーがインターネットに接続できない ###
Azure で Flannel をホスト gw モードでデプロイする場合、パケットは Azure 物理ホスト vSwitch を経由する必要があります。 ユーザーは、ノードに割り当てられているサブネットごとに、"仮想アプライアンス" 型の [ユーザー定義ルート](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#user-defined) をプログラムする必要があります。 これを行うには、Azure portal ( [こちら](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal)の例を参照) または Azure CLI を使用 `az` します。 次に示すのは、IP 10.0.0.4 が指定されたノードに対して az コマンドを使用し、それぞれのポッドサブネット 10.244.0.0/24 を使用して、"MyRoute" という名前の UDR の一例です。
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> 他のクラウドプロバイダーから自分で Kubernetes を Azure または IaaS Vm にデプロイしている場合は、代わりにを使用することもでき `overlay networking` ます。

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Windows ポッドが外部リソースに ping を行うことができない ###
Windows ポッドには、現在 ICMP プロトコル用にプログラミングされている送信規則はありません。 ただし、TCP/UDP はサポートされています。 クラスターの外部にあるリソースへの接続をデモンストレーションする場合は、 `ping <IP>` 対応するコマンドに置き換えてください `curl <IP>` 。

まだ問題が発生している場合は、 [cni](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) のネットワーク構成に注意が必要です。 この静的ファイルをいつでも編集できます。構成は、新しく作成された Kubernetes リソースに適用されます。

なぜですか?
Kubernetes のネットワーク要件の1つ (「 [Kubernetes モデル](https://kubernetes.io/docs/concepts/cluster-administration/networking/)」を参照) は、NAT を使用せずにクラスター通信を行う場合に使用します。 この要件を遵守するために、送信 NAT [を使用し](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) ないすべての通信のための [の追加] があります。 ただし、これは、クエリを実行しようとしている外部 IP を除外する必要があることも意味します。 その後、Windows ポッドから送信されたトラフィックは、外部からの応答を受信するために正しく正常に送信されます。 この点を考慮して、の [] には次のように表示される `cni.conf` 必要があります。
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Windows ノードが NodePort サービスにアクセスできない ###
ノード自体からのローカル NodePort アクセスは失敗する可能性があります。 これは、累積的な更新プログラム KB4571748 (またはそれ以降) で対処されている既知の機能のギャップです。 NodePort アクセスは、他のノードや外部クライアントからも機能します。

### <a name="my-windows-node-stops-routing-thourgh-nodeports-after-i-scaled-down-my-pods"></a>Windows ノードがポッドをスケールダウンした後、thourgh NodePorts のルーティングを停止する ###
デザイン上の制限により、NodePort 転送を機能させるには、Windows ノードで少なくとも1つのポッドが実行されている必要があります。

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>しばらくすると、コンテナーの vNICs と HNS のエンドポイントが削除されます。 ###
この問題は、 `hostname-override` パラメーターが [kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)に渡されない場合に発生する可能性があります。 これを解決するには、ユーザーは次のようにホスト名を kube に渡す必要があります。
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Flannel (vxlan) モードでは、ノードの再参加後にポッドに接続の問題が発生しています ###
以前に削除されたノードがクラスターに再度参加している場合、flannelD はノードに新しいポッドサブネットを割り当てようとします。 ユーザーは、次のパスにある古いポッドサブネット構成ファイルを削除する必要があります。
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-kubernetes-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Kubernetes を起動すると、"ネットワークが作成されるのを待機しています" と Flanneld がスタックします。 ###
この問題は、 [Flannel v 0.12.0](https://github.com/coreos/flannel/releases/tag/v0.12.0) (およびそれ以降) で対処する必要があります。 古いバージョンの Flanneld を使用している場合は、flannel ネットワークの管理 IP が設定されていないことが原因で発生する可能性のある競合状態が発生する可能性があります。 回避策としては、FlannelD を手動で再起動するだけです。
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>/Run/flannel/subnet.env がないため、Windows ポッドを起動できません ###
これは、Flannel が正常に起動しなかったことを示します。 flanneld.exe を再起動するか、Kubernetes マスターから Windows ワーカーノードに手動でファイルをコピーして、 `/run/flannel/subnet.env` `C:\run\flannel\subnet.env` 割り当てられた `FLANNEL_SUBNET` サブネットに行を変更することができます。 たとえば、ノードサブネット 10.244.4.1/24 が割り当てられている場合は、次のようになります。
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
多くの場合、このエラーの原因となる可能性がある別の問題があり、最初に調査する必要があります。 このファイルを自動的に生成することをお勧め `flanneld.exe` します。


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>VSphere で実行されている Kubernetes クラスターでは、ホスト間のポッド間の接続が切断される
VSphere と Flannel はどちらも、オーバーレイネットワーク用にポート 4789 (既定 VXLAN ポート) を予約しているため、パケットが傍受される可能性があります。 VSphere がオーバーレイネットワークに使用されている場合は、4789を解放するために別のポートを使用するように構成する必要があります。


### <a name="my-endpointsips-are-leaking"></a>エンドポイント/Ip がリークしています ###
エンドポイントのリークを引き起こす可能性がある既知の問題が2つ存在します。
1.  最初の [既知の問題](https://github.com/kubernetes/kubernetes/issues/68511) は、Kubernetes バージョン1.11 の問題です。 Kubernetes バージョン 1.11.0-1.11.2 を使用しないようにしてください。
2. エンドポイントのリークを引き起こす可能性がある2番目の [既知の問題](https://github.com/docker/libnetwork/issues/1950) は、エンドポイントのストレージでの同時実行の問題です。 修正プログラムを入手するには、Docker EE 18.09 以上を使用する必要があります。


### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>"ネットワーク: 範囲の割り当てに失敗しました" というエラーが発生したため、ポッドを起動できません ###
これは、ノードの IP アドレス空間が使用されていることを示します。 リークした [エンドポイント](#my-endpointsips-are-leaking)をクリーンアップするには、影響を受けたノードのリソースをすべて移行し、次のコマンドを実行 & ます。
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Windows ノードがサービス IP を使用してサービスにアクセスできない ###
これは、Windows の現在のネットワーク スタックの既知の制限です。 ただし、Windows*ポッド***は**サービス IP にアクセスできます。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Kubelet を起動するときにネットワーク アダプターが見つからない ###
Kubernetes ネットワークが機能するには、Windows ネットワーク スタックで仮想アダプターが必要です。 次のコマンドを実行しても結果が返されない場合 (管理シェルの場合)、Kubelet を正常に機能させるには、HNS ネットワークの作成に &mdash; 必要な前提条件 &mdash; が失敗しました。

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-HnsNetwork | ? Name -ieq "vxlan0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

多くの場合、ホストのネットワークアダプターが "イーサネット" ではない場合に、start.ps1 スクリプトの [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) パラメーターを変更することができます。 それ以外の場合は、スクリプトの出力を参照し `start-kubelet.ps1` て、仮想ネットワークの作成中にエラーが発生していないかどうかを確認します。


### <a name="i-am-still-seeing-problems-what-should-i-do"></a>まだ問題が発生しています。 どうすればよいですか。 ###
ネットワークまたはホストに追加の制約があり、ノード間で特定の種類の通信が妨げられている場合があります。 次のことを確認します。
  - 選択したネットワークトポロジ ( `l2bridge` または) が正しく構成されていること `overlay`
  - ポッドからと思われるトラフィックが許可されていること
  - HTTP トラフィックが許可されていること (Web サービスを展開する場合)
  - さまざまなプロトコル (ie ICMP と TCP/UDP) からのパケットが削除されない

>[!TIP]
> セルフヘルプに関するその他のリソースについては、Kubernetes ネットワークのトラブルシューティングガイド[を参照してください。](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)

## <a name="common-windows-errors"></a>一般的な Windows エラー ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>使用している Kubernetes ポッドが "ContainerCreating" で停止する ###
この問題には多くの原因が考えられますが、最も一般的な原因の 1 つは、pause イメージが正しく構成されていないことです。 これは、次の問題の大きな兆候です。


### <a name="when-deploying-docker-containers-keep-restarting"></a>展開するときに、Docker コンテナーが再起動を繰り返す ###
pause イメージが、OS のバージョンと互換性があることを確認します。 Kubernetes は、OS とコンテナーの両方に一致する OS バージョン番号があることを前提としています。 Insider ビルドなど、Windows の実験用ビルドを使用している場合は、それに応じてイメージを調整する必要があります。 イメージについては、マイクロソフトの [Docker リポジトリ](https://hub.docker.com/u/microsoft/)を参照してください。


## <a name="common-kubernetes-master-errors"></a>一般的な Kubernetes マスターエラー ##
Kubernetes マスターのデバッグは、(発生しやすい順で) 次の 3 つのカテゴリに分類されます。

  - Kubernetes システム コンテナーに問題がある。
  - `kubelet` の実行状態に問題がある。
  - システムに問題がある。

Kubernetes によるポッドの作成を確認するには、`kubectl get pods -n kube-system` を実行します。これにより、いずれかがクラッシュしていたり正しく開始していないとわかることがあります。   次に `docker ps -a` を実行して、これらのポッドの背後にあるすべての未加工コンテナーを表示します。 最後に、問題の原因となっている疑いがあるコンテナーに対して `docker logs [ID]` を実行し、プロセスの未加工出力を確認します。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>`https://[address]:[port]` で API サーバーに接続できない ###
このエラーはほとんどの場合、証明書の問題を示しています。 構成ファイルを正しく生成していること、その中の IP アドレスがホストに対応していること、API サーバーによってマウントされたディレクトリにコピー済みであることを確認します。

この構成ファイルは次の場所で見つけることができます。
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 それ以外の場合は、API サーバーのマニフェストファイルを参照して、マウントポイントを確認します。
