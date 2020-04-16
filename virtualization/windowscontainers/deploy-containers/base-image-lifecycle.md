---
title: 基本イメージのサービスライフサイクル
description: Windows コンテナーの基本イメージライフサイクルに関する情報です。
keywords: windows コンテナー、コンテナー、ライフサイクル、リリース情報、基本イメージ、コンテナー基本イメージ
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 2dcd228af0984b55162894555fa21f9e02dd1934
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/15/2020
ms.locfileid: "81395745"
---
# <a name="base-image-servicing-lifecycles"></a>基本イメージのサービスライフサイクル

> [!Note]  
> Microsoft は、ユーザーや組織がビジネス継続性を維持することに重点を置いていることを支援するために、スケジュールされたサポートの終了とサービス開始日をいくつかの製品に対して遅らせました。 詳細については、2020年4月14日の「[サポート終了までのライフサイクルの変更」と「サービスの日付」を](https://support.microsoft.com/en-us/help/4557164/lifecycle-changes-to-end-of-support-and-servicing-dates)参照してください。

Windows コンテナーの基本イメージは、半期チャネルリリースまたは Windows Server の長期的なサービスチャネルリリースに基づいています。 この記事では、両方のチャネルの基本イメージのさまざまなバージョンについて、どのくらいの期間サポートが終了するかを説明します。

半期チャネルは、各リリースの18か月のサービスタイムラインを使用した、年1回の機能更新リリースです。 これにより、お客様は、アプリケーション (特にコンテナーとマイクロサービスに組み込まれているもの) とソフトウェア定義のハイブリッドデータセンターの両方で、新しいオペレーティングシステム機能を迅速に利用できます。 詳細については、「 [Windows Server 半期チャネルの概要](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)」を参照してください。

Server Core イメージの場合、お客様は、Windows Server の新しいメジャーバージョンを 2 ~ 3 年ごとにリリースする、長期的なサービスチャネルを使用することもできます。 長期的なサービスチャネルのリリースでは、5年間のメインストリームサポートと5年間の拡張サポートが提供されます。 このチャネルは、より長いサービスオプションと機能の安定性を必要とするシステムで動作します。

次の表に、基本イメージの種類、サービスチャネル、およびサポートが最後に実行される期間を示します。

|TestVM                       |サービス チャネル|バージョン|OS ビルド|対象|メインストリーム サポートの終了日|延長サポート日|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core、Nano Server、Windows|半期      |1909   |18363   |2019 年 11 月 12 日  |2021 年 5 月 11 日                 |N/A                  |
|Server Core、Nano Server、Windows|半期      |1903   |18362   |05/21/2019  |2020 年 12 月 8 日                 |N/A                  |
|Server Core                      |長期的        |2019   |17763   |2018 年 11 月 13 日  |2024 年 1 月 9 日                 |2029 年 1 月 9 日           |
|Server Core、Nano Server、Windows|半期      |1809   |17763   |2018 年 11 月 13 日  |11/10/2020                 |N/A                  |
|Server Core、Nano Server         |半期      |1803   |17134   |2018 年 4 月 30 日  |2019 年 11 月 12 日                 |N/A                  |
|Server Core、Nano Server         |半期      |1709   |16299   |2017 年 10 月 17 日  |2019 年 4 月 9日                 |N/A                  |
|Server Core                      |長期的        |1607   |14393   |2016 年 10 月 15 日  |2022 年 1 月 11 日                 |2027 年 1 月 11 日           |
|Nano Server                      |半期      |1607   |14393   |2016 年 10 月 15 日  |10/09/2018                 |N/A                  |

サービスの要件などの追加情報については、「 [windows のライフサイクル](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)に関する FAQ」、「 [windows Server リリース情報](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)」、および「 [Windows ベース OS イメージの Docker hub リポジトリ](https://hub.docker.com/_/microsoft-windows-base-os-images)」を参照してください。
