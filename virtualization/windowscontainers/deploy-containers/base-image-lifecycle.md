---
title: 基本イメージのサービス ライフサイクル
description: Windows コンテナーの基本イメージのライフサイクルに関する情報。
keywords: windows コンテナー, コンテナー, ライフサイクル, リリース情報, 基本イメージ, コンテナー基本イメージ
author: Heidilohr
ms.author: helohr
ms.date: 05/12/2020
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: c6276db89f093b62a01cadc095f5357d2e5a8eba
ms.sourcegitcommit: dd80813679df2de89fe523a26600cfe58a2d39a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/27/2020
ms.locfileid: "84023147"
---
# <a name="base-image-servicing-lifecycles"></a>基本イメージのサービス ライフサイクル

> [!Note]  
> 個人や組織のお客様がビジネス継続性の維持に集中できるように、Microsoft は、多くの製品で予定されていたサポートおよびサービスの終了日を延期しました。 詳細については、2020 年 4 月 14 日の「[サポート終了日とサービスの終了日にライフサイクルを変更](https://support.microsoft.com/en-us/help/4557164/lifecycle-changes-to-end-of-support-and-servicing-dates)」エントリを参照してください。

Windows コンテナーの基本イメージは、Windows Server の半期チャネル リリースまたは長期サービス チャネル リリースに基づいています。 この記事では、両方のチャネルの基本イメージのさまざまなバージョンについてサポート予定期間を説明します。

半期チャネルでは、機能更新プログラムが年 2 回リリースされます。各リリースのサービス タイムラインは 18 か月です。 これにより、お客様は、アプリケーション (特にコンテナーとマイクロサービスに基づいて構築されたもの) とソフトウェア定義のハイブリッド データセンターの両方で、新しいオペレーティング システムの機能をより迅速に利用できます。 詳細については、「[Windows Server の半期チャネルの概要](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)」を参照してください。

Server Core イメージの場合、2、3 年ごとに Windows Server の新しいメジャー バージョンがリリースされる長期サービス チャネルを使用することもできます。 長期サービス チャネルのリリースには、5 年間のメインストリーム サポートと 5 年間の延長サポートが提供されます。 このチャネルは、より長いサービス オプションと機能の安定性が必要なシステムで機能します。

各種類の基本イメージ、サービス チャネル、サポートの継続期間を次の表に示します。

|TestVM                       |サービス チャネル|バージョン|OS ビルド|可用性|メインストリーム サポートの終了日|延長サポート日|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core、Nano Server、Windows|半期      |2004   |19041   |2020 年 5 月 27 日  |2021 年 12 月 14 日                 |なし                  |
|Server Core、Nano Server、Windows|半期      |1909   |18363   |2019 年 11 月 12 日  |2021 年 5 月 11 日                 |なし                  |
|Server Core、Nano Server、Windows|半期      |1903   |18362   |2019 年 5 月 21 日  |2020 年 12 月 8 日                 |なし                  |
|Server Core                      |長期        |2019   |17763   |2018 年 11 月 13 日  |2024 年 1 月 9 日                 |2029 年 1 月 9 日           |
|Server Core、Nano Server、Windows|半期      |1809   |17763   |2018 年 11 月 13 日  |2020 年 11 月 10 日                 |なし                  |
|Server Core、Nano Server         |半期      |1803   |17134   |2018 年 4 月 30 日  |2019 年 11 月 12 日                 |なし                  |
|Server Core、Nano Server         |半期      |1709   |16299   |2017 年 10 月 17 日  |2019 年 4 月 9日                 |なし                  |
|Server Core                      |長期        |2016   |14393   |2016 年 10 月 15 日  |2022 年 1 月 11 日                 |2027 年 1 月 11 日           |
|Nano Server                      |半期      |1607   |14393   |2016 年 10 月 15 日  |2018 年 10 月 9 日                 |なし                  |

サービスの要件とその他の追加情報については、[Windows のライフサイクルの FAQ](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products) に関するページ、「[Windows Server のリリース情報](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)」、および [Windows 基本 OS イメージの Docker ハブ リポジトリ](https://hub.docker.com/_/microsoft-windows-base-os-images)に関するページを参照してください。
