---
title: Hyper-V 統合サービス
description: Hyper-V 統合サービスの参照
keywords: windows 10, hyper-v, 統合サービス, 統合コンポーネント
author: scooley
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
ms.openlocfilehash: 6568b68a77fc5506b58249caea44ec78e3e44de2
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439569"
---
# <a name="hyper-v-integration-services"></a>Hyper-V 統合サービス

統合サービス (多くの場合、統合コンポーネントと呼ばれます) は、仮想マシンから Hyper-V ホストに通信できるようにするサービスです。 これらのサービスの多くは便利な機能ですが、その他にも、仮想マシンの機能が正しく動作する上で非常に重要な働きをするものもあります。

この記事は、Windows で利用可能な各統合サービスに関するリファレンスです。  また、特定の統合サービスまたはそれらの履歴に関連する情報の出発点としての役割も果たしています。

**ユーザー ガイド:**  
* [統合サービスの管理](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Manage-Hyper-V-integration-services)


## <a name="quick-reference"></a>クイック リファレンス

| 名前 | Windows サービス名 | Linux デーモン名 |  説明 | 無効にした場合に VM に与える影響 |
|:---------|:---------|:---------|:---------|:---------|
| [Hyper-V ハートビート サービス](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | 仮想マシンが正しく動作していることを報告します。 | 状況により異なる |
| [Hyper-V ゲスト シャットダウン サービス](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  ホストが仮想マシンのシャットダウンをトリガーするのを許可します。 | **High** |
| [Hyper-V 時刻の同期サービス](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | 仮想マシンのクロックをホスト コンピューターのクロックに同期します。 | **High** |
| [Hyper-V データ交換サービス (KVP)](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | 仮想マシンとホストとの間で基本的なメタデータを交換する手段を提供します。 | 中間 |
| [Hyper-V ボリューム シャドウ コピー リクエスター](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | 仮想マシンをシャットダウンすることなく仮想マシンのバックアップを作成することをボリューム シャドウ コピー サービスに許可します。 | 状況により異なる |
| [Hyper-V ゲスト サービス インターフェイス](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | Hyper-V ホストが仮想マシンとの間でファイルをコピーするのに必要なインターフェイスを提供します。 | 低 |
| [Hyper-V PowerShell ダイレクト サービス](#hyper-v-powershell-direct-service) | vmicvmsession | 利用不可 | ネットワークに接続することなく PowerShell を使用して仮想マシンを管理する方法を提供します。 | 低 |  


## <a name="hyper-v-heartbeat-service"></a>Hyper-V ハートビート サービス

**Windows サービス名:** vmicheartbeat  
**Linux デーモン名:** hv_utils  
**説明 :** 仮想マシンにオペレーティング システムがインストールされており、そのオペレーティング システムが適切に起動されたことを Hyper-V ホストに通知します。  
**追加先:** Windows Server 2012、Windows 8  
**影響**: 無効にした場合、仮想マシンは内部のオペレーティング システムが正しく動作していることをレポートできなくなります。  これは、一部の種類のモニタリングおよびホスト側の診断に影響を与える場合があります。  

ハートビート サービスを使用すると、"仮想マシンは起動しましたか?" のような基本的な質問に回答できるようになります。  

仮想マシンの状態が "実行中" である旨が Hyper-V から報告された場合 (次の例を参照)、これは、Hyper-V が仮想マシン用にリソースを確保していることを意味するものであり、インストールされている、または機能しているオペレーティング システムが存在することを意味するものではありません。  この点で、ハートビートは有用です。  ハートビート サービスは、仮想マシン内のオペレーティング システムが起動したことを Hyper-V に通知します。  

### <a name="check-heartbeat-with-powershell"></a>PowerShell を使用してハートビートをチェックする

仮想マシンのハートビートを確認するには、[Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) を管理者として実行します。
``` PowerShell
Get-VM -VMName $VMName | select Name, State, Status
```

出力は以下のようになります。
```
Name    State    Status
----    -----    ------
DemoVM  Running  Operating normally
```

`Status` フィールドは、ハートビート サービスによって決定されます。



## <a name="hyper-v-guest-shutdown-service"></a>Hyper-V ゲスト シャットダウン サービス

**Windows サービス名:** vmicshutdown  
**Linux デーモン名:** hv_utils  
**説明 :** Hyper-V が仮想マシンのシャットダウンを要求できるようにします。  ホストはいつでも仮想マシンを強制的に停止させることができますが、それはシャットダウンを選択するのではなく、電源スイッチをオフにするようなものです。  
**追加先:** Windows Server 2012、Windows 8  
**影響**: **影響: 大**  無効にした場合、ホストは仮想マシンで安全なシャットダウンをトリガーできなくなります。  シャットダウンはすべて、ハード上の電源オフ操作であり、データの損失またはデータの破損を引き起こす可能性があります。  


## <a name="hyper-v-time-synchronization-service"></a>Hyper-V 時刻の同期サービス

**Windows サービス名:** vmictimesync  
**Linux デーモン名:** hv_utils  
**説明 :** 仮想マシンのシステム クロックを物理コンピューターのシステム クロックに同期します。  
**追加先:** Windows Server 2012、Windows 8  
**影響**: **影響: 大**  無効にした場合、仮想マシンのクロックが変動し不安定になります。  


## <a name="hyper-v-data-exchange-service-kvp"></a>Hyper-V データ交換サービス (KVP)

**Windows サービス名:** vmickvpexchange  
**Linux デーモン名:** hv_kvp_daemon  
**説明 :** 仮想マシンとホストとの間で基本的なメタデータを交換する手段を提供します。  
**追加先:** Windows Server 2012、Windows 8  
**影響**: 無効にした場合、Windows 8、Windows Server 2012 またはそれ以前のバージョンを実行する仮想マシンは、Hyper-V 統合サービスに対する更新プログラムを受信しません。  データ交換を無効にすると、一部の種類のモニタリングおよびホスト側の診断に影響を与える場合もあります。  

データ交換サービス (KVP とも呼ばれる) では、仮想マシンと Hyper-V との間で、キー/値ペア (KVP) を使用し Windows レジストリを介して少量のマシン情報を共有します。  同じメカニズムを使用して、仮想マシンとホストとの間でカスタマイズされたデータを共有することもできます。

キー/値ペアは、"キー" と "値" で構成されます。 キーと値は両方とも文字列であり、他のデータ型はサポートされていません。 キー/値ペアが作成または変更されると、ゲストおよびホストからそれを参照できます。 キー/値ペアの情報は、Hyper-V VMbus 経由で転送されるので、ゲストと Hyper-V ホストの間に任意の種類のネットワーク接続は必要ありません。 

データ交換サービスは仮想マシンに関する情報を維持するのに優れたツールです。対話型データの共有またはデータ転送の場合は、[PowerShell ダイレクト](#hyper-v-powershell-direct-service)を使用します。 


**ユーザー ガイド:**  
* [キー/値ペアを使用して Hyper-V 上のホストとゲストの間で情報を共有する](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn798287(v=ws.11))。  


## <a name="hyper-v-volume-shadow-copy-requestor"></a>Hyper-V ボリューム シャドウ コピー リクエスター

**Windows サービス名:** vmicvss  
**Linux デーモン名:** hv_vss_daemon  
**説明 :** 仮想マシン上のアプリケーションとデータをバックアップすることをボリューム シャドウ コピー サービスに許可します。  
**追加先:** Windows Server 2012、Windows 8  
**影響**: 無効にした場合、仮想マシンを実行中にバックアップできなくなります (VSS を使用)。  

ボリューム シャドウ コピー サービス ([VSS](https://docs.microsoft.com/windows/desktop/VSS/overview-of-processing-a-backup-under-vss)) には、ボリューム シャドウ コピー リクエスター統合サービスが必要です。  ボリューム シャドウ コピー サービス (VSS) では、実行中のシステム (特にサーバー) 上で、それらが提供しているパフォーマンスおよびサービスを低下させることなく、バックアップのイメージをキャプチャおよびコピーします。  この統合サービスは、仮想マシンのワークロードとホストのバックアップ プロセスを調整してこれを実現します。

ボリューム シャドウ コピーの詳細については、[こちら](https://docs.microsoft.com/previous-versions/windows/desktop/virtual/backing-up-and-restoring-virtual-machines)を参照してください。


## <a name="hyper-v-guest-service-interface"></a>Hyper-V ゲスト サービス インターフェイス

**Windows サービス名:** vmicguestinterface  
**Linux デーモン名:** hv_fcopy_daemon  
**説明 :** Hyper-V ホストが仮想マシンとの間で双方向のファイル コピーを実行できるようにするためのインターフェイスを提供します。  
**追加先:** Windows Server 2012 R2、Windows 8.1  
**影響**: 無効にした場合、ホストは、`Copy-VMFile` を使用してゲストとの間でファイルをコピーできなくなります。  Copy-VMFile コマンドレットについては、[こちら](https://docs.microsoft.com/powershell/module/hyper-v/copy-vmfile?view=win10-ps)を参照してください。  

**注:**  
既定では無効になっています。  [Copy-Item を使用した PowerShell ダイレクト](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)に関するページを参照してください。 


## <a name="hyper-v-powershell-direct-service"></a>Hyper-V PowerShell ダイレクト サービス

**Windows サービス名:** vmicvmsession  
**Linux デーモン名:** なし  
**説明 :** 仮想ネットワークを使わずに PowerShell を使用して VM セッション経由で仮想マシンを管理するためのメカニズムを提供します。    
**追加先:** Windows Server TP3、Windows 10  
**影響**: このサービスを無効にすると、ホストは PowerShell ダイレクトを使用して仮想マシンに接続することができなくなります。  

**注:**  
このサービスの元の名前は、Hyper-V VM セッション サービスでした。  
PowerShell ダイレクトは現在開発中であり、Windows 10/Windows Server Technical Preview 3 またはそれ以降のホスト/ゲストでのみ使用できます。

PowerShell ダイレクトでは、Hyper-V ホストまたは仮想マシンのいずれかにおけるネットワーク構成またはリモート管理設定に関係なく、Hyper-V ホストから仮想マシン内の PowerShell の管理ができます。 これにより、Hyper-V 管理者は管理タスクと構成タスクを簡単に自動化およびスクリプト化できるようになります。

PowerShell ダイレクトの詳細については、[こちら](../user-guide/powershell-direct.md)を参照してください。  

**ユーザー ガイド:**  
* [仮想マシンでのスクリプトの実行](../user-guide/powershell-direct.md#run-a-script-or-command-with-invoke-command)
* [仮想マシンとの間でのファイルのコピー](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)
