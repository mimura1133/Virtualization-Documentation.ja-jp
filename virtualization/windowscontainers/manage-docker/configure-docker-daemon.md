---
title: Windows で Docker を構成する
description: Windows で Docker を構成する
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 05/03/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: db794a0151b6554bd4f8a4a4e494bfab034d2da0
ms.sourcegitcommit: c585bd8b70d0681d8deaf5fefd3f5f93c1ec4316
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/01/2020
ms.locfileid: "85613434"
---
# <a name="docker-engine-on-windows"></a>Windows 上の Docker エンジン

Docker エンジンとクライアントは Windows に含まれていないため、個別にインポートして構成する必要があります。 また、Docker エンジンは多くのカスタム構成に対応できます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントでは、Docker エンジンのインストールおよび構成方法について説明します。また、一般的に使用される構成例も紹介します。

## <a name="install-docker"></a>Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジン (dockerd.exe) と Docker クライアント (docker.exe) で構成されます。 すべてをインストールするための最も簡単な方法は、クイックスタート ガイドを参照してください。このガイドは、すべての設定と最初のコンテナーの実行に役立ちます。

- [Docker のインストール](../quick-start/set-up-environment.md)

スクリプト化したインストールについては、「[Use a script to install Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)」 (スクリプトを使用して Docker EE をインストールする) をご覧ください。

Docker を使用する前に、コンテナー イメージをインストールする必要があります。 詳細については、[コンテナーの基本イメージに関するドキュメント](../manage-containers/container-base-images.md)を参照してください。

## <a name="configure-docker-with-a-configuration-file"></a>構成ファイルを使用して Docker を構成する

Windows で Docker エンジンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'C:\ProgramData\Docker\config\daemon.json' にあります。 このファイルがまだ存在しない場合は、作成できます。

>[!NOTE]
>使用可能なすべての Docker 構成オプションが Windows 上の Docker に適用されるわけではありません。 次の例では、適用する構成オプションを示しています。 Docker エンジン構成の詳細なについては、[Docker デーモン構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)をご覧ください。

```json
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "",
    "mtu": 0,
    "pidfile": "",
    "data-root": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

目的の構成の変更を構成ファイルに追加するだけです。 たとえば、次のサンプルでは、ポート 2375 で受信接続を受け入れるように Docker エンジンを構成しています。 その他のすべての構成オプションには、既定値が使用されます。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同様に、次のサンプルでは、代替パスにイメージとコンテナーを保持するように Docker デーモンが構成されます。 指定されていない場合は、既定値は `c:\programdata\docker` です。

```json
{   
    "data-root": "d:\\docker"
}
```

このサンプルでは、セキュリティで保護された接続をポート 2376 でのみ受け入れるように Docker デーモンを構成しています。

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Docker サービスで Docker を構成する

`sc config` を使用して Docker サービスを変更することで、Docker エンジンを構成することもできます。 この方法の場合、Docker エンジンのフラグは Docker サービスに直接設定されます。 コマンド プロンプト (PowerShell ではなく cmd.exe) で、次のコマンドを実行します。

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>daemon.json ファイルに `"hosts": ["tcp://0.0.0.0:2375"]` というエントリが既に含まれている場合は、このコマンドを実行する必要はありません。

## <a name="common-configuration"></a>一般的な構成

次に、一般的な Docker 構成の構成ファイル例を示します。 これらの構成を 1 つの構成ファイルにまとめることができます。

### <a name="default-network-creation"></a>既定のネットワークの作成

既定の NAT ネットワークが作成されないように Docker エンジンを構成するには、次の構成を使用します。

```json
{
    "bridge" : "none"
}
```

詳細については、「[コンテナーのネットワーク](../container-networking/network-drivers-topologies.md)」を参照してください。

### <a name="set-docker-security-group"></a>Docker セキュリティ グループの設定

Docker ホストにログインし、Docker コマンドをローカルで実行すると、名前付きパイプ経由でこれらのコマンドが実行されます。 既定では、Administrators グループのメンバーのみが、名前付きパイプ経由で Docker エンジンにアクセスできます。 このアクセス権を持つセキュリティ グループを指定するには、`group` フラグを使用します。

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>プロキシの構成

`docker search` と `docker pull` のプロキシ情報を設定するには、`HTTP_PROXY` または `HTTPS_PROXY` という名前と、プロキシ情報の値を使用して Windows 環境変数を作成します。 そのためには、PowerShell で次のようなコマンドを実行します。

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

変数が設定されたら、Docker サービスを再起動します。

```powershell
Restart-Service docker
```

詳細については、[Docker.com の Windows 構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)をご覧ください。

## <a name="how-to-uninstall-docker"></a>Docker をアンインストールする方法

このセクションでは、Docker をアンインストールし、Windows 10 または Windows Server 2016 のシステムから Docker システム コンポーネントの完全クリーンアップを行う方法について説明します。

>[!NOTE]
>以下の手順のコマンドはすべて、管理者特権による PowerShell セッションで実行する必要があります。

### <a name="prepare-your-system-for-dockers-removal"></a>お使いのシステムで Docker 削除の準備を行う

Docker をアンインストールする前に、コンテナーがシステム上で実行されていないことを確認してください。

次のコマンドレットを実行して、実行中のコンテナーを確認します。

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

また、Docker を削除する前に、お使いのシステムからすべてのコンテナー、コンテナー イメージ、ネットワーク、およびボリュームを削除するようお勧めします。 これを行うには、次のコマンドレットを使用します。

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Docker のアンインストール

次に、Docker を実際にアンインストールする必要があります。

Windows 10 で Docker をアンインストールするには

- Windows 10 のコンピューターで、 **[設定]**  >  **[アプリ]** の順に移動します。
- **[アプリと機能]** で、 **[Docker for Windows]** を見つけます。
- **Docker for Windows** > **アンインストール** にアクセスします

Windows Server 2016 で Docker をアンインストールするには:

次の例に示されるように、管理者特権による PowerShell セッションで、**パッケージのアンインストール** および **モジュールのアンインストール** コマンドレットを使用して、Docker モジュールおよび対応するパッケージ管理プロバイダーをシステムから削除します。

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Docker をインストールするために使用したパッケージ プロバイダーを見つけるには、`PS C:\> Get-PackageProvider -Name *Docker*` を使用します。

### <a name="clean-up-docker-data-and-system-components"></a>Docker のデータおよびシステム コンポーネントをクリーンアップする

Docker をアンインストールした後は、Docker の既定のネットワークを削除する必要があります。これにより、Docker が使用されなくなった後も、その構成がシステムに残りません。 これを行うには、次のコマンドレットを使用します。

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Windows Server 2016 で Docker の既定のネットワークを削除するには、次のようにします。
```powershell
Get-ContainerNetwork | Remove-ContainerNetwork
```

次のコマンドレットを実行して、Docker のプログラム データをお使いのシステムから削除します。

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Windows 上の Docker/コンテナーに関連付けられている Windows オプション機能も必要に応じて削除します。

これには少なくとも、Windows 10 または Windows Server 2016 で Docker をインストールすると自動的に有効になる "コンテナー" 機能が含まれます。 また、"Hyper-V" 機能も含まれる可能性があります。これは、Windows 10 では Docker のインストール時に自動的に有効になりますが、Windows Server 2016 では明示的に有効にする必要がある機能です。

>[!IMPORTANT]
>[Hyper-V の機能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)には、単なるコンテナーを大幅に超える全般的な仮想化機能が含まれます。 Hyper-V 機能を無効にする前に、この機能を必要とするその他の仮想化コンポーネントがお使いのシステム上にないことを確認してください。

Windows 10 で Windows 機能を削除するには:

- **[コントロール パネル]**  >  **[プログラム]**  >  **[プログラムと機能]**  >  **[Windows の機能の有効化または無効化]** に移動します。
- 無効にする機能の名前または機能として、この場合は **[コンテナー]** および (必要に応じて) **[Hyper-V]** を見つけます。
- 無効にする機能の名前の横にあるチェック ボックスをオフにします。
- **「OK」** を選択します。

Windows Server 2016 で Windows 機能を削除するには:

管理者特権による PowerShell セッションで以下のコマンドレットを使用して、お使いのシステムから **[コンテナー]** および (必要に応じて) **[Hyper-V]** 機能を無効にします。

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>システムを再起動する

アンインストールとクリーンアップを完了するには、管理者特権の PowerShell セッションから次のコマンドレットを実行して、お使いのシステムを再起動します。

```powershell
Restart-Computer -Force
```
