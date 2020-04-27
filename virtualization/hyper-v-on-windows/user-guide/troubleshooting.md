---
title: Windows 10 の Hyper-V のトラブルシューティング
description: Windows 10 の Hyper-V のトラブルシューティング
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
ms.openlocfilehash: bdb9feeb2452f2784a3b814e85dc72f3b967a9d3
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439329"
---
# <a name="troubleshoot-hyper-v-on-windows-10"></a>Windows 10 の Hyper-V のトラブルシューティング

## <a name="i-updated-to-windows-10-and-now-i-cant-connect-to-my-downlevel-windows-81-or-server-2012-r2-host"></a>Windows 10 に更新しましたが、下位レベル (Windows 8.1 または Server 2012 R2) のホストに接続できません
Windows 10 では、Hyper-V マネージャーがリモート管理の WinRM に移動されました。  つまり、Hyper-V マネージャーを使用して管理するには、リモート管理をリモート ホスト上で有効にする必要があります。

詳細については、「[Hyper-V ホストの管理](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Remotely-manage-Hyper-V-hosts)」をご覧ください。

## <a name="i-changed-the-checkpoint-type-but-it-is-still-taking-the-wrong-type-of-checkpoint"></a>チェックポイントの種類を変更しましたが、誤った種類のチェックポイントを取得し続けています
VMConnect からチェックポイントを取得していて、Hyper-V マネージャーのチェックポイントの種類を変更した場合、いずれの種類であっても取得されるチェックポイントは、VMConnect を開いたときに指定したチェックポイントの種類になります。

VMConnect を閉じて、もう一度開き、正しいチェックポイントの種類を取得するようにします。

## <a name="when-i-try-to-create-a-virtual-hard-disk-on-a-flash-drive-an-error-message-is-displayed"></a>フラッシュ ドライブ上に仮想ハード ディスクを作成しようとしましたが、エラー メッセージが表示されます
これらのファイル システムはアクセス制御リスト (ACL) を指定しないため、また、4 GB より大きいファイルをサポートしないため、Hyper-V では FAT または FAT32 でフォーマットされたディスク ドライブをサポートしません。 ExFAT でフォーマットされたディスクは、ACL 機能を制限付きで提供するため、セキュリティ上の理由により、サポートもされません。
PowerShell に表示されるエラー メッセージは、"システムで '\[VHD へのパス\]' を作成できませんでした:ファイル システム制限のため、要求された操作を完了できませんでした (0x80070299)" です。

代わりに、NTFS フォーマットのドライブを使用します。 

## <a name="i-get-this-message-when-i-try-to-install-hyper-v-cannot-be-installed-the-processor-does-not-support-second-level-address-translation-slat"></a>インストールしようとすると、このメッセージが表示されます。"Hyper-V をインストールできません:プロセッサは、第 2 レベルのアドレス変換 (SLAT) をサポートしていません。"
Hyper-V では、仮想マシンを実行するために SLAT が必要です。 コンピューターが SLAT をサポートしていない場合、仮想マシンのホストにすることはできません。

管理ツールのインストールのみを行う場合は、 **[プログラムと機能]**  >  **[Windows の機能の有効化または無効化]** で、 **[Hyper-V プラットフォーム]** の選択を解除します。
