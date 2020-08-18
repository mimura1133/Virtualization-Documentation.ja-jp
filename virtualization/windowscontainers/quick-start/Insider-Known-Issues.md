---
title: Insider ビルドに関する既知の問題
description: Insider ビルドに関する既知の問題。
keywords: Docker, コンテナー
ms.topic: quickstart
author: cwilhit
ms.openlocfilehash: 7bb5567be29d93310e77b28e5718303bcdf39ab3
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161691"
---
# <a name="known-issues-for-insider-builds"></a>Insider ビルドに関する既知の問題

## <a name="build-16237"></a>ビルド 16237

- Hyper-v による分離が正常に機能していません。 ビルド 16237 で Hyper-V による分離を使用するには、この回避策を実行する必要があります。 PowerShell で、以下のコマンドを実行します。

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server がユーザーとして実行されるようになったため、管理者特権を必要とするコマンドが失敗します。 "RUN setx /M PATH" などの行が含まれている場合は、ビルドできません。 このシナリオでは、以下のコマンドを代わりに使用できます。

```dockerfile
RUN setx PATH <path>
```
