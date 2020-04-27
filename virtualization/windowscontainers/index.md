---
layout: LandingPage
title: Windows 上のコンテナーに関するドキュメント
description: Windows でのコンテナーの実行に関するドキュメント
keywords: Docker, コンテナー
author: cwilhit
ms.date: 09/11/2019
ms.topic: landing-page
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
ms.openlocfilehash: 9a5b08f87983e285418ae333e3a948af9911d73d
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "74909162"
---
<div id="main" class="v2">
    <ul class="cardsY panelContent featuredContent">
        <li>
            <a href="https://docs.microsoft.com/en-us/azure/aks/windows-container-cli" data-linktype="external">
                <div class="cardSize">
                    <div class="cardPadding">
                        <div class="card">
                            <div class="cardImageOuter">
                                <div class="cardImage">
                                    <img src="media/logo_kubernetes.svg" alt="" data-linktype="relative-path">
                                </div>
                            </div>
                            <div class="cardText">
                                <h3>AKS で Windows コンテナーを試してみてください。</h3>
                            </div>
                        </div>
                    </div>
                </div>
            </a>
        </li>
        <li>
            <a href="https://hub.docker.com/_/microsoft-windows-base-os-images" data-linktype="external">
                <div class="cardSize">
                    <div class="cardPadding">
                        <div class="card">
                            <div class="cardImageOuter">
                                <div class="cardImage">
                                    <img src="media/logo_docker.svg" alt="" data-linktype="relative-path">
                                </div>
                            </div>
                            <div class="cardText">
                                <h3>Docker Hub でコンテナー イメージを確認してください。</h3>
                            </div>
                        </div>
                    </div>
                </div>
            </a>
        </li>
        <li>
            <a href="https://techcommunity.microsoft.com/t5/Containers/bg-p/Containers" data-linktype="external">
                <div class="cardSize">
                    <div class="cardPadding">
                        <div class="card">
                            <div class="cardImageOuter">
                                <div class="cardImage">
                                    <img src="media/i_blog.svg" alt="" data-linktype="relative-path">
                                </div>
                            </div>
                            <div class="cardText">
                                <h3>コンテナーのブログをお読みください。</h3>
                            </div>
                        </div>
                    </div>
                </div>
            </a>
        </li>
    </ul>
    <h1>Windows 上のコンテナーに関するドキュメント</h1>
    <br/>
    <div class="abstract">Windows コンテナーを使用すると、ユーザーはアプリケーションを依存関係と共にパッケージ化し、オペレーティング システム レベルの仮想化を利用して、完全に分離された高速な環境を単一のシステム上に実現できます。 Windows コンテナーの使用方法については、クイック スタート ガイド、デプロイ ガイド、およびサンプルを参照してください。</div>
    <ul class="cardsW panelContent featuredContent">
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-about.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">概要</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/about/index" data-linktype="absolute-path">Windows コンテナーについて</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/deploy-containers/system-requirements" data-linktype="absolute-path">システム要件</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/about/faq" data-linktype="absolute-path">FAQ</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-quick-start.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">はじめに</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/quick-start/set-up-environment" data-linktype="external">環境をセットアップする</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/quick-start/run-your-first-container" data-linktype="external">最初のコンテナーを実行する</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/quick-start/building-sample-app" data-linktype="external">サンプル アプリをコンテナー化する</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/container-tutorials.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">チュートリアル</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile" data-linktype="external">Windows コンテナーの構築</a></li>
                                <li><a href="/azure/aks/windows-container-cli" data-linktype="external">AKS で実行する</a></li>
                                <li><a href="/azure/service-fabric/service-fabric-quickstart-containers" data-linktype="external">Service Fabric で実行する</a></li>
                                <li><a href="/azure/app-service/app-service-web-get-started-windows-container" data-linktype="external">Azure App Service で実行する</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-management-tools.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">概念</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon" data-linktype="external">Docker</a></li>
                                <li><a href="/virtualization/windowscontainers/about/overview-container-orchestrators" data-linktype="external">コンテナーのオーケストレーション</a></li>
                                <li><a href="/virtualization/windowscontainers/manage-containers/container-base-images" data-linktype="external">Windows コンテナーの基本事項</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/container-reference.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">リファレンス</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility" data-linktype="external">バージョンの互換性</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/deploy-containers/base-image-lifecycle" data-linktype="external">イメージのサービス ライフサイクル</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/images-eula" data-linktype="external">コンテナー OS イメージの使用許諾契約書 (EULA)</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-community.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">参考資料</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/samples" data-linktype="external">サンプル</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/troubleshooting" data-linktype="external">トラブルシューティング</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/communitylinks" data-linktype="external">コミュニティのビデオとブログ</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
    </ul>
</div>