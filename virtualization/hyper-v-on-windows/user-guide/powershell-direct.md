---
title: PowerShell ダイレクトでの Windows Virtual Machines の管理
description: PowerShell ダイレクトでの Windows Virtual Machines の管理
keywords: windows 10, hyper-v, powershell, 統合サービス, 統合コンポーネント, 自動化, powershell ダイレクト
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: ed96c7ba30c83906cd3245a279ab078229400d8d
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439549"
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>PowerShell を使用した仮想マシンの自動化と管理

PowerShell ダイレクトを使用すると、ネットワークの構成やリモート管理の設定に関係なく、Hyper-V ホストから、Windows 10 仮想マシンまたは Windows Server 2016 仮想マシンで任意の PowerShell を実行することができます。

以下に、PowerShell ダイレクトを実行する方法をいくつか示します。

* [Enter-PSSession コマンドレットを使用する対話型セッションとして](#create-and-exit-an-interactive-powershell-session)
* [Invoke-Command コマンドレットを使用して単一のコマンドまたはスクリプトを実行する 1 回限りの使用のセクションとして](#run-a-script-or-command-with-invoke-command)
* [New-PSSession、Copy-Item、および Remove-PSSession コマンドレットを使用する永続的なセッションとして (ビルド 14280 以降)](#copy-files-with-new-pssession-and-copy-item)

## <a name="requirements"></a>要件
**オペレーティング システムの要件:**
* ホスト: Windows 10、Windows Server 2016、または Hyper-V を実行するそれ以降のバージョン。
* ゲスト/仮想マシン: Windows 10、Windows Server 2016、またはそれ以降のバージョン。

以前の仮想マシンを管理している場合は、仮想マシン接続 (VMConnect) を使用するか、[仮想マシン用の仮想ネットワークを構成します](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816585(v=ws.10))。 

**構成に必要な条件:**    
* 仮想マシンがホスト上でローカルに実行されている必要があります。
* 仮想マシンが有効であり、少なくとも 1 つの構成済みユーザー プロファイルで実行されている必要があります。
* HYPER-V の管理者として、ホスト コンピューターには、ログインする必要があります。
* 仮想マシンの有効なユーザー資格情報を指定する必要があります。

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>対話型 PowerShell セッションの作成と終了

仮想マシンで PowerShell コマンドを実行するには、対話型セッションを開始するのが最も簡単な方法です。

セッションが始まると、入力したコマンドが、仮想マシン自体の PowerShell セッションに直接入力されたかのように、仮想マシンで実行されます。

**対話型セッションを開始するには:**

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、仮想マシンの名前または GUID を使用して対話型セッションが作成されます。  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  メッセージが表示されたら仮想マシンの資格情報を指定します。

3. 仮想マシンでコマンドを実行します。
  
  PowerShell プロンプトのプレフィックスとして VMName が次のように表示されます。
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  仮想マシンで任意のコマンドが実行されます。 テストのために、`ipconfig` または `hostname` を実行して、これらのコマンドが仮想マシンで実行されていることを確認します。
  
4. 次のように完了したら、セッションを終了するには、次のコマンドを実行します。  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> 注: セッションが接続されない場合は、考えられる原因の「[トラブルシューティング](#troubleshooting)」を参照してください。 

これらのコマンドレットの詳細については、「[Enter-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Enter-PSSession?view=powershell-5.1)」と「[Exit-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Exit-PSSession?view=powershell-5.1)」を参照してください。 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>Invoke-Command でのスクリプトまたはコマンドの実行

PowerShell ダイレクトと Invoke-Command コマンドの組み合わせは、1 つのコマンドまたは 1 つのスクリプトを仮想マシン上で実行する必要があるが、それらの実行以降は仮想マシンとの対話を続行する必要がないという場合に最適です。

**単一のコマンドを実行するには:**

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、仮想マシンの名前または GUID を使用してセッションが作成されます。  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   メッセージが表示されたら仮想マシンの資格情報を指定します。
   
   仮想マシンでコマンドが実行されます。コンソールへの出力がある場合は、その内容がコンソールに出力されます。  コマンドが実行されるとすぐに、接続が自動的に切断されます。
   
   
**スクリプトを実行するには:**

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、仮想マシンの名前または GUID を使用してセッションが作成されます。  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   メッセージが表示されたら仮想マシンの資格情報を指定します。
   
   仮想マシンでスクリプトが実行されます。  コマンドが実行されるとすぐに、接続が自動的に切断されます。

このコマンドレットの詳細については、「[Invoke-Command](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Invoke-Command?view=powershell-5.1)」を参照してください。 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>New-PSSession および Copy-Item でファイルをコピーする

> **注:** PowerShell ダイレクトでは、Windows ビルド 14280 以降でのみ永続的なセッションをサポートしています。

永続的な PowerShell セッションは、1 つまたは複数のリモート コンピューターを対象にした動作を調整するスクリプトを記述する際にとても便利です。  いったん作成された永続的なセッションは、それを削除するまで、バック グラウンドに保持されます。  すなわち、資格情報を渡さなくても`Invoke-Command` または `Enter-PSSession` を使用して、同じセッションを何度でも繰り返し参照することができます。

セッションは同じトークンにより状態を保持します。  永続的なセッションは存続するので、セッションで作成された変数またはセッションに渡された変数はいずれも複数回の呼び出しにわたり保持されます。 永続的なセッションでの作業で使用できるさまざまなツールがあります。  たとえば、[New-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/New-PSSession?view=powershell-5.1) および [Copy-Item](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Management/Copy-Item?view=powershell-5.1) を使用して、ホストから仮想マシンへ、仮想マシンからホストへデータを移動することができます。

**セッションを作成し、ファイルをコピーするには:**  

1. Hyper-V ホストで PowerShell を管理者として開きます。

2. 次のいずれかのコマンドを実行すると、`New-PSSession` を使用して仮想マシンに対する永続的な PowerShell セッションが作成されます。
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  メッセージが表示されたら仮想マシンの資格情報を指定します。
  
  > **警告:**  
   14500 より前のビルドにはバグがあります。  `-Credential` フラグで資格情報が明示的に指定されていない場合、ゲスト内のサービスはクラッシュするので、再起動する必要があります。  この問題が発生した場合、解決策については[こちら](#error-a-remote-session-might-have-ended)を参照してください。
  
3. 仮想マシンにファイルをコピーします。
  
  ホスト マシンから仮想マシンに `C:\host_path\data.txt` をコピーするには、次を実行します。
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  仮想マシンから (ホストに) ファイルをコピーします。 
   
   仮想マシンからホストに `C:\guest_path\data.txt` をコピーするには、次を実行します。
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. `Remove-PSSession` を使用して永続的なセッションを停止します。
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>トラブルシューティング

PowerShell Direct を通じて表示される一般的なエラー メッセージが少しあります。  いくつかの最も一般的な原因、および問題を診断するためのツールは、次のとおりです。

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>-VMName または -VMID パラメーターが存在しない
**問題**:  
`Enter-PSSession`、`Invoke-Command`、または `New-PSSession` に `-VMName` パラメーターまたは `-VMId` パラメーターがありません。

**考えられる原因:**  
最も可能性の高い原因として、PowerShell ダイレクトがご使用のホスト オペレーティング システムでサポートされないことが挙げられます。

Windows ビルドは、次のコマンドを実行して確認できます。

``` PowerShell
[System.Environment]::OSVersion.Version
```

サポートされたビルドを実行していても、ご使用の PowerShell バージョンで PowerShell ダイレクトが実行されない可能もあります。  PowerShell ダイレクトと JEA については、メジャー バージョンが 5 以降である必要があります。

PowerShell バージョンのビルドは、次のコマンドを実行して確認できます。

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>エラー: リモート セッションが終了した可能性がある
> **注:**  
10240 ～ 12400 のホスト ビルドでの Enter-PSSession の場合、以下に示すエラーはすべて、"リモート セッションが終了した可能性がある" とレポートされます。

**エラー メッセージ:**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**考えられる原因:**
* 仮想マシンは存在しますが、まだ実行されていません。
* ゲスト OS が PowerShell Direct をサポートしていない (「[要件](#requirements)」を参照してください)
* PowerShell がまだゲストで使用できない
  * オペレーティング システムが起動を完了していない
  * オペレーティング システムが正しく起動できない
  * ユーザー入力が必要な起動時のイベントがある

[GeT-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) コマンドレットを使用すれば、どの VM がホストで実行されているかを確認することができます。

**エラー メッセージ:**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**考えられる原因:**
* 上の一覧に示したいずれかの理由です。これらはすべて `New-PSSession` にも同様に当てはまります  
* 現在のビルドにバグがあります。`-Credential` で資格情報を明示的に渡す必要があります。  このエラーが発生すると、ゲスト オペレーティング システムでサービス全体がハングし、再起動する必要があります。  Enter-PSSession を使用すれば、セッションが引き続き使用できるかどうかを確認できます。

資格情報の問題を回避するには、VMConnect を使用して仮想マシンにログインし、PowerShell を開き、次の PowerShell を使用して vmicvmsession サービスを再開します。

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>エラー: パラメーター セットを解決できません。
**エラー メッセージ:**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**考えられる原因:**  
* 仮想マシンへの接続時に `-RunAsAdministrator` がサポートされません。
     
  Windows コンテナーに接続するときは、`-RunAsAdministrator` フラグを指定すると、明示的な資格情報がなくても管理者として接続できます。  仮想マシンはホストに暗黙の管理者アクセス権を付与しないため、資格情報を明示的に入力する必要があります。

仮想マシンに管理者の資格情報を渡すには、`-Credential` パラメーターを使用するか、要求されたときに手動で入力します。


### <a name="error-the-credential-is-invalid"></a>エラー: 資格情報が無効です。

**エラー メッセージ:**  
```
Enter-PSSession : The credential is invalid.
```

**考えられる原因:** 
* ゲストの資格情報を検証できない
  * 指定された資格情報が誤っていた。
  * ゲストにユーザー アカウントがない (前もって OS が起動されていない)
  * 管理者として接続する場合: 管理者がアクティブなユーザーとして設定されていない。  [こちら](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh825104(v=win.10)>)をご覧ください。
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>エラー: 入力 VMName パラメーターが仮想マシンに解決されません。

**エラー メッセージ:**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**考えられる原因:**  
* Hyper-V の管理者ではありません。  
* 仮想マシンが存在しません。

[Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) コマンドレットを使用して、使用している資格情報に Hyper-V 管理者の役割があることを確認し、ホスト上でローカルで実行されている、起動した VM を確認することができます。


-------------

## <a name="samples-and-user-guides"></a>サンプルとユーザー ガイド

PowerShell ダイレクトでは JEA (Just Enough Administration) をサポートしています。  これを試すには、このユーザー ガイドをご覧ください。

[GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) のサンプルをご確認ください。
