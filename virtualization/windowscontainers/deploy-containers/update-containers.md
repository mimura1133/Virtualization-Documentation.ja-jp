---
title: Windows Server コンテナーの更新
description: Windows の複数のバージョン間で、ビルドとコンテナーを実行する方法について説明します。
keywords: メタデータ, コンテナー, バージョン
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 84413f27bfce66e7d259c05795a280ed34b582ab
ms.sourcegitcommit: 6f216408434a437da87a72d582500a4ca6c2679c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/21/2020
ms.locfileid: "80112695"
---
# <a name="update-windows-server-containers"></a>Windows Server コンテナーの更新

毎月、Windows Server のサービスの一部として、更新された Windows Server ベース OS コンテナー イメージが定期的に公開されています。 これらの更新プログラムを使用すると、更新されたコンテナー イメージの作成を自動化したり、最新バージョンをプルして手動で更新したりすることができます。 Windows Server コンテナーには、Windows Server のようなサービス スタックがありません。 Windows Server の場合のように、コンテナー内で更新プログラムを取得することはできません。 そのため、更新プログラムを使用した Windows Server ベース OS コンテナー イメージの再構築と、更新されたコンテナー イメージの公開が毎月行われています。

.NET や IIS などの他のコンテナー イメージは、更新されたベース OS コンテナー イメージに基づいて再構築され、毎月公開されます。

## <a name="how-to-get-windows-server-container-updates"></a>Windows Server コンテナーの更新プログラムを取得する方法

Windows Server ベース OS コンテナー イメージは、Windows のサービス サイクルに合わせて更新されます。 更新されたコンテナー イメージは毎月第 2 火曜日に公開されます。これは "B" リリースとも呼ばれ、リリース月に基づくプレフィックス番号が付けられます。 たとえば、2 月の更新プログラムは "2B"、3 月の更新プログラムは "3B" と呼ばれます。 月ごとのこの更新イベントは、新しいセキュリティ修正プログラムを含む唯一の定期リリースです。

これらのコンテナーをホストするサーバーは、コンテナー ホストまたは単に "ホスト" と呼ばれ、"B" リリース以外の追加の更新イベント中にサービスが提供される場合があります。 Windows の更新プログラムのサービス サイクルについて詳しくは、[Windows の更新プログラムのサービス サイクル](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376)に関するブログ記事を参照してください。

新しい Windows Server ベース OS コンテナー イメージは、毎月第 2 火曜日の午前 10 時 (太平洋標準時) 頃に Microsoft Container Registry (MCR) で公開され、おすすめタグが最新の B リリースに付けられます。 次のような例があります。

- ltsc2019 [(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc): docker pull mcr.microsoft.com/windows/servercore:ltsc2019
- 1909 [(SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker pull mcr.microsoft.com/windows/servercore:1909

MCR より Docker Hub に慣れている場合は、[こちらのブログ記事](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/)でさらに詳しい説明をご覧いただけます。  

各リリースでは、それぞれのコンテナー イメージは、特定のコンテナー イメージのリビジョンをターゲットとするリビジョン番号とサポート技術情報の記事番号を表す 2 つの追加タグも付けて公開されます。 たとえば、次のように入力します。

- docker pull mcr.microsoft.com/windows/servercore:10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore:1809-KB4546852

どちらの例も、2 月 18 日のセキュリティ リリース更新プログラムを含む Windows Server 2019 の Server Core コンテナー イメージをプルします。  

Windows Server ベース OS コンテナー イメージ、バージョン、およびそれぞれのタグの完全な一覧については、Docker Hub の [Windows ベース OS コンテナー イメージ](https://hub.docker.com/_/microsoft-windows-base-os-images)に関するこちらのページを参照してください。

Microsoft により Azure Marketplace でリリースされ、月ごとにサービスが提供される Windows Server イメージにも、プレインストールされたベース OS コンテナー イメージが付属します。 詳細については、[Windows Server Azure Marketplace の価格に関するページ](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice)を参照してください。 通常、これらのイメージは、"B" リリースからおよそ 5 営業日後に更新されます。

Windows Server のイメージとバージョンの完全な一覧については、「[Azure Marketplace での Windows Server リリースの更新履歴](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history)」を参照してください。

## <a name="host-and-container-version-compatibility"></a>ホストとコンテナーのバージョンの互換性

Windows コンテナーには 2 種類の分離モードがあります。プロセスの分離と Hyper-V による分離です。 ホストとコンテナーのバージョンの互換性については、Hyper-V による分離の方が柔軟性があります。 詳細については、[バージョンの互換性](version-compatibility.md)と[分離モード](../manage-containers/hyperv-container.md)に関するページを参照してください。 このセクションでは、特に明記されていない限り、プロセス分離コンテナーを中心に説明します。

月ごとの更新プログラムを使用してコンテナー ホストやコンテナー イメージを更新する場合、そのホストとコンテナー イメージが両方ともサポートされている限り (Windows Server バージョン 1809 以降)、ホストとコンテナー イメージのリビジョンが一致していなくてもコンテナーは正常に起動して実行されます。

ただし、2020 年 2 月 11 日のセキュリティ更新プログラム リリース ("2B" とも呼ばれる) またはそれ以降の月次セキュリティ更新プログラム リリースを含む Windows Server コンテナーを使用すると、問題が発生する場合があります。 詳細については、[こちらの Microsoft サポート記事](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t)を参照してください。 これらの問題は、アプリケーションのセキュリティを確保するために、ユーザー モードとカーネル モードの間のインターフェイスを変更する必要があるセキュリティの変更によるものです。 この問題は、プロセス分離コンテナーでのみ発生します。これは、プロセス分離コンテナーでは、カーネル モードがコンテナー ホストと共有されるためです。 このため、更新されたユーザー モード コンポーネントのないコンテナー イメージは、セキュリティで保護されておらず、セキュリティで保護された新しいカーネル インターフェイスと互換性がありませんでした。

2020 年 2 月 18 日に修正プログラムがリリースされました。 この新しいリリースにより、"新しいベースライン" が確立されました。 この新しいベースラインは、次の規則に従います。

- ホストとコンテナーがどちらも 2B より前であれば、これらをどのように組み合わせても機能する。
- ホストとコンテナーがどちらも 2B より後であれば、これらをどのように組み合わせても機能する。
- 新しいベースラインの前後をまたぐホストとコンテナーの組み合わせはいずれも機能しない。 たとえば、3B ホストと 1B コンテナーは機能しません。

2020 年 3 月の月次セキュリティ更新プログラム リリースを例に、これらの新しい互換性規則がどのように機能するかを説明します。 次の表で、2020 年 3 月のセキュリティ更新プログラム リリースは 3B、2020 年 2 月の更新プログラムは 2B、2020 年 1 月の更新プログラムは 1B と表記されています。

| ホスト | Container | 互換性 |
|---|---|---|
| 3B | 3B | はい |
| 3B | 2B | はい |
| 3B | 1B 以前 | いいえ |
| 2B | 3B | はい |
| 2B | 2B | はい |
| 2B | 1B 以前 | いいえ |
| 1B 以前 | 3B | いいえ |
| 1B 以前 | 2B | いいえ |
| 1B 以前 | 1B 以前 | はい |

参照用に、1B、2B、および 3B の月次セキュリティ更新プログラム リリースを含むベース OS コンテナー イメージのバージョン番号を、Windows Server 2016 から最新の Windows Server バージョン 1909 リリースまでのさまざまな主要 OS リリースについて、次の表にまとめて示します。

| Windows Server のバージョン (フローティング タグ) | 2020/1/14 リリース (1B) の更新プログラムのバージョン| 2020/2/18 リリース (2B) の更新プログラムのバージョン | 2020/3/10 リリース (3B) の更新プログラムのバージョン |
|---|---|---|---|
| Windows Server 2016 (ltsc2016) | 10.0.14393.3443 (KB4534271) | 10.0.14393.3506 (KB4546850) | 10.0.14393.3568 (KB4551573) |
| Windows Server バージョン 1803 (1803) | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | このバージョンのサポートは終了しました。 詳細については、「[基本イメージのサービス ライフサイクル](base-image-lifecycle.md)」を参照してください。|
| Windows Server バージョン 1809 (1809)| 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server 2019 (ltsc2019) | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server バージョン 1903 (1903) |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server バージョン 1909 (1909) | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>ホストとコンテナー イメージの不一致のトラブルシューティング

開始する前に、[バージョンの互換性](version-compatibility.md)に関する情報を十分にご確認ください。 この情報は、問題が修正プログラムの不一致によって生じたのかどうかを判断するのに役立ちます。 修正プログラムの不一致が原因であることが判明した場合は、このセクションの手順に従って問題をトラブルシューティングできます。

### <a name="query-the-version-of-your-container-host"></a>コンテナー ホストのバージョンについてクエリを実行する

コンテナー ホストにアクセスできる場合は、`ver` コマンドを実行して、その OS バージョンを取得できます。 たとえば、最新の 2020 年 2 月のセキュリティ更新プログラム リリースが適用された Windows Server 2019 を実行しているシステムで `ver` を実行すると、次のような結果が表示されます。

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>この例では、リビジョン番号が 1040 ではなく 1039 と表示されます。これは、2020 年 2 月のセキュリティ更新プログラム リリースには Windows Server 用のアウトオブバンド 2B リリースがなかったためです。 コンテナー用のアウトオブバンド 2B リリースのみがあり、そのリビジョン番号が 1040 でした。

コンテナー ホストに直接アクセスできない場合は、IT 管理者にご確認ください。クラウドで実行している場合は、クラウド プロバイダーの Web サイトで、実行されているコンテナー ホストの OS バージョンをご確認ください。 たとえば、お客様が Azure Kubernetes Service (AKS) を使用している場合は、[AKS リリース ノート](https://github.com/Azure/AKS/releases)でホスト OS のバージョンを確認できます。

### <a name="query-the-version-of-your-container-image"></a>コンテナー イメージのバージョンについてクエリを実行する

次の手順に従って、コンテナーが実行しているバージョンを確認します。

1. PowerShell で次のコマンドレットを実行します。

    ```powershell
    docker images
    ```

    出力は次のようになります。

     ```powershell
     REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
     mcr.microsoft.com/windows/servercore   ltsc2019            b456290f487c        4 weeks ago         4.84GB
     mcr.microsoft.com/windows              1809                58229ca44fa7        4 weeks ago         12GB
     mcr.microsoft.com/windows/nanoserver   1809                f519d4f3a868        4 weeks ago         251M

2. Run the `docker inspect` command for the Image ID of the container image that isn't working. This will tell you which version the container image is targeting.

   For example, let's say we `run docker inspect` for an ltsc 2019 container image:

   ```powershell
   docker inspect b456290f487c

       "Architecture": "amd64",

        "Os": "windows",

        "OsVersion": "10.0.17763.1039",

        "Size": 4841309825,

        "VirtualSize": 4841309825,
    ```

    この例では、コンテナーの OS バージョンは `10.0.17763.1039` として表示されています。

    コンテナーを既に実行している場合は、コンテナー内で `ver` コマンドを実行してバージョンを取得することもできます。 たとえば、最新の 2020 年 2 月のセキュリティ更新プログラム リリースが適用された Windows Server 2019 の Server Core コンテナー イメージで `ver` を実行すると、次のような結果が表示されます。

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```
