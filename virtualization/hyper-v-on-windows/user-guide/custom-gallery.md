---
title: カスタム仮想マシン ギャラリーの作成
description: Windows 10 Creators Update 以降では、仮想マシン ギャラリーに独自のエントリを構築できます。
keywords: Windows 10, Hyper-V, クイック作成, 仮想マシン, ギャラリー
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: 1348b9923d9de1314818f13414abdacee2cb9735
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439716"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>カスタム仮想マシン ギャラリーの作成

> Windows 10 Fall Creators Update 以降。

Fall Creators Update ではクイック作成が拡張され、仮想マシン ギャラリーが追加されています。

![カスタム イメージが表示された、クイック作成の VM ギャラリー](media/vmgallery.png)

ギャラリーには、Microsoft および Microsoft パートナーによって提供されたイメージが用意されていますが、独自のイメージを追加することもできます。

この記事の内容:

* ギャラリーと互換性のある仮想マシンを構築する。
* 新しいギャラリー ソースを作成する。
* カスタムのギャラリー ソースをギャラリーに追加する。

## <a name="gallery-architecture"></a>ギャラリーのアーキテクチャ

仮想マシン ギャラリーは、Windows レジストリで定義されている仮想マシン ソースのセットをグラフィカル表示したものです。  各仮想マシン ソースは、一覧項目としての仮想マシンと JSON ファイルへのパス (ローカル パスまたは URI) です。

ギャラリーに表示される仮想マシンの一覧では、1 つ目のソースの詳細内容に 2 番目以降のソースの内容が続く形で、使用可能な仮想マシンがすべて表示されます。  この一覧は、ギャラリーを起動するたびに動的に作成されます。

![ギャラリーのアーキテクチャ](media/vmgallery-architecture.png)

レジストリ キー: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

値の名前: `GalleryLocations`

型: `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>ギャラリーと互換性がある仮想マシンを作成する

ギャラリー内の仮想マシンは、ディスク イメージ (.iso) または仮想ハード ドライブ (.vhdx) のいずれかです。

仮想ハード ディスク ドライブからの仮想マシンには、いくつかの構成要件があります。

1. UEFI ファームウェアをサポートするように構築されていること。 Hyper-V を使用して作成された場合は、第 2 世代 VM になります。
1. 仮想ハード ドライブは 20 GB 以上にすること。これは最大サイズです。  Hyper-V では、VM がアクティブに使用していない領域が占有されません。

### <a name="testing-a-new-vm-image"></a>新しい VM イメージのテスト

仮想マシン ギャラリーでは、ローカル インストール ソースからインストールする場合と同じメカニズムを使用して仮想マシンが作成されます。

仮想マシンイメージの起動と実行を確認するには:

1. VM ギャラリー (Hyper-V クイック作成) を開き、 **[ローカル インストール ソース]** を選択します。
  ![ローカル インストール ソースを使用するためのボタン](media/use-local-source.png)
1. **[インストール元の変更]** を選択します。
  ![ローカル インストール ソースを使用するためのボタン](media/change-source.png)
1. ギャラリーで使用する .iso または .vhdx を選択します。
1. イメージが Linux イメージの場合は、セキュア ブート オプションを選択解除します。
  ![ローカル インストール ソースを使用するためのボタン](media/toggle-secure-boot.png)
1. 仮想マシンを作成します。  仮想マシンが正しく起動する場合は、ギャラリーに追加する準備が完了です。

## <a name="build-a-new-gallery-source"></a>新しいギャラリー ソースを作成する

次の手順では、新しいギャラリー ソースを作成します。  これは、ギャラリーに仮想マシンを一覧表示して詳細情報を追加する JSON ファイルです。

テキスト情報:

![ギャラリー テキストの場所のラベル表示](media/gallery-text.png)

* **name**: 必須。左の列に表示される名前です。仮想マシン ビューの最上部にも表示されます。
* **publisher**: (必須)
* **description**: 必須。VM を説明する文字列の一覧。
* **version**: 必須
* lastUpdated: 既定値は 0001 年 1 月 1 日月曜日です。

  yyyy-mm-ddThh:mm:ssZ の形式にする必要があります。

  次の PowerShell コマンドでは、現在の日付が適切な形式でクリップボードに置かれます。

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* locale - 既定値は空白です。

画像:

![ギャラリー画像の場所のラベル表示](media/gallery-pictures.png)

* **logo** - 必須
* symbol
* thumbnail

仮想マシン (.iso または .vhdx)。

ハッシュを生成するには、次の PowerShell コマンドを使用できます。

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

以下 JSON テンプレートには、スターター項目とギャラリーのスキーマが含まれています。  これを VSCode で編集すると、自動的に IntelliSense が提供されます。

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>ギャラリーを VM ギャラリー UI に接続する

カスタムのギャラリー ソースを VM ギャラリーに追加する最も簡単な方法は、regedit で追加することです。

1. **regedit.exe** を開きます。
1. 次に移動します: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`
1. `GalleryLocations` という項目を探します。

    既に存在する場合は、 **[編集]** メニューの **[修正]** に移動します。

    既に存在していない場合は、 **[編集]** メニューに移動し、 **[新規]** 、 **[複数行文字列値]** の順に選択します。

1. ギャラリーを `GalleryLocations` レジストリ キーに追加します。

    ![ギャラリーのレジストリ キーに新しい項目が表示されている](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>トラブルシューティング

### <a name="check-for-errors-loading-gallery"></a>ギャラリーの読み込みエラーをチェックする

仮想マシン ギャラリーでのエラー報告は、Windows イベント ビューアーで確認できます。  エラーをチェックするには:

1. イベント ビューアーを開きます。
1. **[Windows ログ]**  ->  **[Application]** に移動します。
1. ソース VMCreate からのイベントを探します。

## <a name="resources"></a>参考資料

GitHub の[リンク](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery)には、少数のギャラリー スクリプトとヘルパーが掲載されています。

ギャラリー エントリのサンプルについては、[こちら](https://go.microsoft.com/fwlink/?linkid=851584)をご覧ください。  これは、受信トレイ ギャラリーを定義する JSON ファイルです。
