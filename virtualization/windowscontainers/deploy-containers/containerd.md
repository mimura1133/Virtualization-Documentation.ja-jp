---
title: Windows コンテナー プラットフォーム
description: Windows で使用できるようになった新しいコンテナーの構成要素について詳しく説明します。
keywords: LCOW, Linux コンテナー, Docker, コンテナー, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439279"
---
# <a name="container-platform-tools-on-windows"></a>Windows 上のコンテナー プラットフォーム ツール

Windows コンテナー プラットフォームは拡大し続けています。 Docker はコンテナーの最初の段階でした。現在は、他のコンテナー プラットフォーム ツールを構築しています。

* [containerd/cri](https://github.com/containerd/cri) - Windows Server 2019 と Windows 10 1809 の新機能。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - runc に対応する Windows コンテナー ホスト。
* [hcs](https://docs.microsoft.com/virtualization/api/) - Host Compute Service と、それを使いやすくする便利なシム。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

この記事では、Windows および Linux のコンテナー プラットフォームと各コンテナー プラットフォーム ツールについて説明します。

## <a name="windows-and-linux-container-platform"></a>Windows と Linux のコンテナー プラットフォーム

Linux 環境では、Docker などのコンテナー管理ツールは、より細かいコンテナー ツールセットである [runc](https://github.com/opencontainers/runc) と [containerd](https://containerd.io/) に基づいて構築されています。

![Linux 上の Docker アーキテクチャ](media/docker-on-linux.png)

`runc` は、[OCI コンテナー ランタイム仕様](https://github.com/opencontainers/runtime-spec)に従ってコンテナーを作成および実行するための Linux コマンドライン ツールです。

`containerd` は、コンテナー イメージのダウンロードと展開からコンテナーの実行と監視まで、コンテナーのライフ サイクルを管理するデーモンです。

Windows では、別のアプローチを採用しました。  Windows コンテナーをサポートするために Docker を使い始めたときに、HCS (ホスト コンピューティング サービス) 上に直接構築しました。  [このブログ投稿](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)には、HCS を構築した理由と、このアプローチを最初にコンテナーに採用した理由に関する情報が詳しく書かれています。

![Windows 上の初期の Docker エンジン アーキテクチャ](media/hcs.png)

この時点でも、Docker から HCS が直接呼び出されています。 ただし、その後は、コンテナー管理ツールが拡張されて Windows コンテナーと Windows コンテナー ホストが含まれ、Linux 上で containerd と runc を呼び出す方法で containerd と runhcs を呼び出すことができるようになりました。

## <a name="runhcs"></a>runhcs

`runhcs` は `runc` のフォークです。  `runc` と同様に、`runhcs` は Open Container Initiative (OCI) 形式に従ってパッケージ化されたアプリケーションを実行するためのコマンド ライン クライアントであり、Open Container Initiative 仕様に準拠した実装です。

runc と runhcs の機能上の違いは次のとおりです。

* `runhcs` は Windows 上で動作します。  コンテナーを作成および管理するために [HCS](containerd.md#hcs) と通信します。
* `runhcs` では、さまざまな種類のコンテナーを実行できます。

  * Windows と Linux の [Hyper-V による分離](../manage-containers/hyperv-container.md)
  * Windows プロセス コンテナー (コンテナー イメージはコンテナー ホストと一致する必要があります)

**使用法:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` は、開始するコンテナー インスタンスの名前です。 この名前は、コンテナー ホスト上で一意である必要があります。

バンドル ディレクトリ (`-b bundle` を使用) は省略可能です。  
runc と同様に、コンテナーはバンドルを使用して構成されます。 コンテナーのバンドルは、コンテナーの OCI 仕様ファイル "config.json" のあるディレクトリです。  "バンドル" の既定値は現在のディレクトリです。

正常に実行するには、OCI 仕様ファイル "config.json" に 2 つのフィールドが必要です。

* コンテナーのスクラッチ領域のパス
* コンテナーのレイヤー ディレクトリのパス

runhcs で使用できるコンテナー コマンドは次のとおりです。

* コンテナーを作成して実行するためのツール
  * **run** コンテナーを作成して実行します
  * **create** コンテナーを作成します

* コンテナーで実行されているプロセスを管理するためのツール:
  * **start** 作成されたコンテナー内でユーザー定義プロセスを実行します
  * **exec** コンテナー内で新しいプロセスを実行します
  * **pause** pause を使用して、コンテナー内のすべてのプロセスを一時停止します
  * **resume** 以前に一時停止されたすべてのプロセスを再開します
  * **ps** ps を使用して、コンテナー内で実行されているプロセスを表示します

* コンテナーの状態を管理するためのツール
  * **state** コンテナーの状態を出力します
  * **kill** 指定されたシグナル (既定値: SIGTERM) をコンテナーの init プロセスに送信します
  * **delete** デタッチされたコンテナーでよく使用されているコンテナーが保持するすべてのリソースを削除します

マルチコンテナーと見なすことができる唯一のコマンドは、**list** です。  指定されたルートで runhcs によって開始された実行中または一時停止中のコンテナーを一覧表示します。

### <a name="hcs"></a>HCS

GitHub には、HCS とのインターフェイスとして使用できる 2 つのラッパーがあります。 HCS は C API であるため、ラッパーを使用すると、高水準言語から HCS を簡単に呼び出すことができます。  

* [hcsshim](https://github.com/microsoft/hcsshim) - HCSShim は Go で記述されており、runhcs のベースです。
AppVeyor から最新のものを入手するか、自分で構築してください。
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) - dotnet-computevirtualization は、HCS の C# ラッパーです。

HCS を (直接またはラッパー経由で) 使用する場合、または HCS に Rust/Haskell/InsertYourLanguage ラッパーを作成する場合は、コメントを残してください。

HCS の詳細については、[John Stark の DockerCon プレゼンテーション](https://www.youtube.com/watch?v=85nCF5S8Qok)を参照してください。

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI サポートは、サーバー 2019 以降と Windows 10 1809 以降でのみ使用できます。  また、Windows 用の Containerd の開発も積極的に行っています。
> 開発およびテスト専用です。

OCI 仕様では 1 つのコンテナーが定義されていますが、[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (コンテナー ランタイム インターフェイス) には、コンテナーをポッドという共有サンドボックス環境のワークロードとして記述されています。  ポッドには、1 つ以上のコンテナー ワークロードを含めることができます。  ポッドを使用すると、Kubernetes や Service Fabric Mesh などのコンテナー オーケストレーターによって、同じホスト上にある、メモリや vNET などの共有リソースを持つグループ化されたワークロードを処理できるようになります。

containerd/cri を使用すると、ポッドの次の互換性マトリクスが有効になります。

| ホスト OS | Container OS | 分離 | ポッドのサポート |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | あり - 真のマルチコンテナー ポッドをサポートします。 |
|  | Windows Server 2019/1809 | `process`* または `hyperv` | あり - 各ワークロード コンテナー OS がユーティリティ VM OS と一致する場合、真のマルチコンテナー ポッドをサポートします。 |
|  | Windows Server 2016、</br>Windows Server 1709、</br>Windows Server 1803 | `hyperv` | 部分的 - コンテナー OS がユーティリティ VM OS と一致する場合、ユーティリティ VM ごとに 1 つのプロセス分離コンテナーをサポートできるポッド サンドボックスをサポートします。 |

\*Windows 10 ホストは Hyper-V による分離のみをサポートします

CRI 仕様へのリンク:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - ポッドの仕様
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - ワークロードの仕様

![Containerd ベースのコンテナー環境](media/containerd-platform.png)

runHCS と containerd はどちらも任意の Windows システム サーバー 2016 以降で管理できますが、ポッド (コンテナーのグループ) をサポートするには、Windows のコンテナー ツールに重大な変更を加える必要がありました。  CRI サポートは、Windows Server 2019 以降と Windows 10 1809 以降で使用できます。
