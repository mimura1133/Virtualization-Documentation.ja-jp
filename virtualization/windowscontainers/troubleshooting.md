---
title: Windows コンテナーのトラブルシューティング
description: Windows コンテナーと Docker に関するトラブルシューティングのヒント、自動スクリプト、およびログ情報
keywords: docker, コンテナー, トラブルシューティング, ログ
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: b408582df4aad340a575029d6f7bb8a997ffadbf
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2020
ms.locfileid: "87984706"
---
# <a name="troubleshooting"></a>トラブルシューティング

コンピューターのセットアップやコンテナーの実行で問題が発生したときのために、 一般的な問題を検出する PowerShell スクリプトを用意しました。 最初にこのスクリプトを実行して、結果を調べてみてください。

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
このスクリプトで実行されるすべてのテストの一覧と一般的な解決策が、スクリプトの [Readme ファイル](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md)に記載されています。

問題の原因を特定できない場合は、スクリプトの出力を[コンテナー フォーラム](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)で投稿してください。 Windows Insider 参加者や開発者も集まるこのコミュニティは、手助けを求めるには最適の場所です。


### <a name="finding-logs"></a>ログを見つける
Windows コンテナーの管理には、複数のサービスが使用されています。 次の各セクションで、ログの場所をサービス別に示します。

## <a name="docker-container-logs"></a>Docker コンテナーログ
コマンドは、 `docker logs` Linux アプリケーションの標準のアプリケーションログの保存場所である STDOUT/STDERR からコンテナーのログをフェッチします。 Windows アプリケーションは通常、STDOUT/STDERR には記録されません。代わりに、ETW、イベントログ、またはログファイルに記録されます。

Microsoft がサポートしているオープンソースツールである[Log Monitor](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor)は、github で入手できるようになりました。 ログモニターは、Windows アプリケーションログを STDOUT/STDERR にブリッジします。 ログモニターは、構成ファイルを使用して構成されます。

### <a name="log-monitor-usage"></a>ログモニターの使用状況

LogMonitor.exe と LogMonitorConfig.jsを同じ LogMonitor ディレクトリに含める必要があります。

ログモニターは、シェルの使用パターンで使用できます。

```
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "cmd", "/S", "/C"]
CMD c:\windows\system32\ping.exe -n 20 localhost
```

または ENTRYPOINT の使用パターン:

```
ENTRYPOINT C:\LogMonitor\LogMonitor.exe c:\windows\system32\ping.exe -n 20 localhost
```

どちらの例でも、ping.exe アプリケーションをラップします。 その他のアプリケーション ( [IIS など)。ServiceMonitor]( https://github.com/microsoft/IIS.ServiceMonitor)) は、同様の方法で Log Monitor と入れ子にすることができます。

```
COPY LogMonitor.exe LogMonitorConfig.json C:\LogMonitor\
WORKDIR /LogMonitor
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "powershell.exe"]

# Start IIS Remote Management and monitor IIS
ENTRYPOINT      Start-Service WMSVC; `
                    C:\ServiceMonitor.exe w3svc;
```


ログモニターは、ラップされたアプリケーションを子プロセスとして起動し、アプリケーションの STDOUT 出力を監視します。

シェルの使用パターンでは、CMD/ENTRYPOINT 命令はシェル形式ではなく、exec 形式で指定する必要があることに注意してください。 COMMAND/ENTRYPOINT 命令の exec 形式が使用されている場合、シェルは起動されず、ログモニターツールはコンテナー内で起動されません。

使用方法の詳細については、[ログモニター wiki](https://github.com/microsoft/windows-container-tools/wiki)を参照してください。 主要な Windows コンテナーシナリオ (IIS など) の構成ファイルの例については、 [github リポジトリ](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor/src/LogMonitor/sample-config-files)を参照してください。 その他のコンテキストについては、この[ブログの投稿](https://techcommunity.microsoft.com/t5/Containers/Windows-Containers-Log-Monitor-Opensource-Release/ba-p/973947)を参照してください。

## <a name="docker-engine"></a>Docker エンジン
Docker エンジンは、ファイルではなく Windows 'アプリケーション' イベント ログに記録します。 これらのログは、Windows PowerShell を使用することで、簡単に読み取り、並べ替え、およびフィルター処理することができます。

たとえば、過去 5 分間の Docker エンジン ログを古い順に表示できます。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time
```

また、別のツールで読み取り可能な CSV ファイル、またはスプレッドシートに簡単にパイプすることができます。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

### <a name="enabling-debug-logging"></a>デバッグ ログを有効にする
Docker エンジンのデバッグ レベルのログ出力を有効にすることもできます。 これは、トラブルシューティングに必要な詳細情報が通常のログでは得られない場合に役立つ可能性があります。

管理者特権でのコマンド プロンプトを開いてから、`sc.exe qc docker` を実行して Docker サービスの現在のコマンド ラインを取得します。
例:
```
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

現在の `BINARY_PATH_NAME` を次のとおりに変更します。
- 末尾に -D を追加する
- " をそれぞれ \ でエスケープする
- コマンド全体を " で囲む

変更したら、`sc.exe config docker binpath=` の後に変更後の文字列を付けて実行します。 例:
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


次に、Docker サービスを再起動します。
```
sc.exe stop docker
sc.exe start docker
```

このようにすると、大量のログがアプリケーション イベント ログに出力されるため、トラブルシューティングが完了したら `-D` オプションを削除することをお勧めします。 上記の同じ手順を、`-D` を付けずに実行してからサービスを再起動すると、デバッグ ログは出力されなくなります。

上記の手順の代わりに、管理者特権の PowerShell プロンプトからデバッグ モードで docker デーモンを実行して、ファイルに直接出力をキャプチャすることもできます。
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

### <a name="obtaining-stack-dump"></a>スタックダンプの取得

一般に、これは、Microsoft サポートまたは docker 開発者によって明示的に要求された場合にのみ有効です。 Docker がハングしているように見える状況を診断するために使用できます。

[docker signal.exe](https://github.com/moby/docker-signal) をダウンロードします。

用途:
```PowerShell
docker-signal --pid=$((Get-Process dockerd).Id)
```

出力ファイルは、docker が実行されているデータルートディレクトリにあります。 既定のディレクトリは `C:\ProgramData\Docker` です。 実際のディレクトリは、`docker info -f "{{.DockerRootDir}}"` を実行することによって確認できます。

ファイルはになり `goroutine-stacks-<timestamp>.log` ます。

に `goroutine-stacks*.log` は個人情報が含まれていないことに注意してください。


## <a name="host-compute-service"></a>ホスト コンピューティング サービス
Docker エンジンは、Windows 固有のホスト コンピューティング サービスに依存します。 このサービスには、次のような独立したログがあります。
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

これらはイベントビューアーに表示され、PowerShell を使用してクエリを行うこともできます。

例:
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational
```

### <a name="capturing-hcs-analyticdebug-logs"></a>HCS 分析/デバッグ ログをキャプチャする

Hyper-V Compute の分析/デバッグ ログを有効にし、`hcslog.evtx` に保存します。

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

### <a name="capturing-hcs-verbose-tracing"></a>HCS 詳細トレースをキャプチャする

一般的に、これらが使用されるのは、Microsoft サポートによって要求された場合のみです。

[HcsTraceProfile.wprp](https://github.com/MicrosoftDocs/Virtualization-Documentation/blob/master/windows-server-container-tools/wpr-profiles/HcsTraceProfile.wprp) をダウンロードします。

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

`HcsTrace.etl` をサポート担当者に提供します。
