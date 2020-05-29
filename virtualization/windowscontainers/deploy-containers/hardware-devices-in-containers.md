---
title: Windows のコンテナー内のデバイス
description: Windows 上のコンテナーで存在しているデバイス サポート
keywords: Docker, コンテナー, デバイス, ハードウェア
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910602"
---
# <a name="devices-in-containers-on-windows"></a>Windows のコンテナー内のデバイス

既定で、Windows コンテナーには、Linux コンテナーと同様に、ホスト デバイスへの最小限のアクセス権が付与されます。 ホストのハードウェア デバイスにアクセスして通信することが役に立つ (または必須でさえある) 特定のワークロードがあります。 このガイドでは、コンテナーでサポートされているデバイスについてと、開始する方法について説明します。

## <a name="requirements"></a>要件

この機能を動作させるには、お使いの環境が次の要件を満たしている必要があります。
- コンテナー ホストでは、Windows Server 2019 または Windows 10 バージョン 1809 以降が実行されている必要があります。
- コンテナーの基本イメージのバージョンは、1809 以降である必要があります。
- コンテナーは、プロセス分離モードで実行されている Windows コンテナーである必要があります。
- コンテナー ホストでは、Docker エンジン 19.03 以降が実行されている必要があります。

## <a name="run-a-container-with-a-device"></a>デバイスでコンテナーを実行する

デバイスでコンテナーを起動するには、次のコマンドを使用します。

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

`{interface class guid}` は、次のセクションで説明されている適切な[デバイス インターフェイス クラス GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes) に置き換える必要があります。

複数のデバイスでコンテナーを起動するには、次のコマンドを使用して、複数の `--device` 引数を結び合わせます。

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows では、すべてのデバイスで、デバイスが実装するインターフェイス クラスのリストが宣言されます。 このコマンドを Docker に渡すことにより、要求されたクラスを実装中であるとして識別されるすべてのデバイスが、確実にコンテナーに組み込まれます。

したがって、デバイスをホストと切り離して割り当てるのでは**ありません**。 むしろ、ホストはデバイスをコンテナーと共有します。 同様に、ユーザーはクラス GUID を指定しているので、その GUID を実装する_すべての_デバイスがコンテナーと共有されます。

## <a name="what-devices-are-supported"></a>サポートされるデバイス

現在、次のデバイス (およびそのデバイス インターフェイス クラス GUID) がサポートされています。
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>デバイスの種類</center></th>
<th><center>インターフェイス クラス GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C バス</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM ポート</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI バス</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU アクセラレーション</center></td>
<td><center><a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU アクセラレーション</a>に関するドキュメントを参照</center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> デバイスのサポートはドライバーによって異なります。 上の表で定義されていないクラス GUID を渡そうとすると、未定義の動作が発生する可能性があります。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V で分離された Windows コンテナーのサポート

Hyper-V で分離された Windows コンテナーのワークロードのデバイス割り当てやデバイス共有は、現在サポートされていません。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V で分離された Linux コンテナーのサポート

Hyper-V で分離された Linux コンテナーのワークロードのデバイス割り当てやデバイス共有は、現在サポートされていません。
