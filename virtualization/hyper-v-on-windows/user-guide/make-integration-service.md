---
title: 独自の統合サービスを作成する
description: Windows 10 の統合サービス。
keywords: windows 10, hyper-v, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: 89a36ee87bce1da18852f0ebff248e239165eb7d
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/24/2020
ms.locfileid: "74911032"
---
# <a name="make-your-own-integration-services"></a>独自の統合サービスを作成する

Windows 10 Anniversary Update以降、Hyper-V ソケット (新しいアドレス ファミリと仮想マシンを対象とした特殊なエンドポイントを備えた Windows ソケット) を使って Hyper-V ホストと仮想マシン間の通信を行うアプリケーションをだれでも作成できるようになりました。  Hyper-V を介したすべての通信はネットワークを使わずに実行され、すべてのデータは同じ物理メモリにとどまります。 Hyper-V ソケットを使うアプリケーションは、Hyper-V の統合サービスと似ています。

このドキュメントでは、Hyper-V ソケット上に単純なプログラムを構築する手順を説明します。

**サポートされるホスト OS**
* Windows 10 以降
* Windows Server 2016 以降

**サポートされるゲスト OS**
* Windows 10 以降
* Windows Server 2016 以降
* Linux ゲストと Linux 統合サービス (「[Supported Linux and FreeBSD virtual machines for Hyper-V on Windows (Windows 上の Hyper-V 向けにサポートされる Linux と FreeBSD 仮想マシン)](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows)」をご覧ください)
> **注:** サポートされる Linux ゲストでは、以下のコマンドに対応するカーネル サポートが必要です。
> ```bash
> CONFIG_VSOCKET=y
> CONFIG_HYPERV_VSOCKETS=y
> ```

**機能と制限事項**
* カーネル モードまたはユーザー モード操作をサポート
* データ ストリームのみ
* ブロック メモリなし (バックアップ/ビデオに最適でない)

--------------

## <a name="getting-started"></a>はじめに

要件:
* C/C++ コンパイラ。  お持ちではない場合は、[Visual Studio コミュニティ](https://aka.ms/vs)を確認してください。
* [Windows 10 SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk): Visual Studio 2015 with Update 3 以降でプレインストールされています。
* 上記のいずれかのホスト オペレーティング システムと 1 つ以上の仮想マシンが実行されているコンピューター。 これは、アプリケーションのテスト用です。

> **注:** Hyper-V ソケットの API は、Windows 10 Anniversary Update で公開されています。 HVSocket を使うアプリケーションは、あらゆる Widnows 10 ホストとゲストで実行できますが、開発には Windows SDK ビルド 14290 以降が必要です。

## <a name="register-a-new-application"></a>新しいアプリケーションの登録
Hyper-V ソケットを使用するには、アプリケーションを Hyper-V ホストのレジストリに登録する必要があります。

レジストリにサービスを登録することで、次が得られます。
*  使用可能なサービスを有効化、無効化、および一覧表示するための WMI 管理
*  仮想マシンと直接通信するアクセス許可

次の PowerShell は、"HV Socket Demo" という名前の新しいアプリケーションを登録しています。  これは管理者として実行する必要があります。  手動の手順は次のとおりです。

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**レジストリの場所と情報:**
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
このレジストリの場所には、いくつかの GUID が表示されます。  これらは、マイクロソフトのボックスでのサービスです。

サービスごとのレジストリ内の情報。
* `Service GUID`
    * `ElementName (REG_SZ)` - これは、サービスのフレンドリ名です。

独自のサービスを登録するには、独自の GUID とフレンドリ名を使用して新しいレジストリ キーを作成します。

フレンドリ名は、新しいアプリケーションに関連付けられます。  これは、GUID が適切でないパフォーマンス カウンターやその他の場所に表示されます。

レジストリ エントリは、次のようになります。
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName    REG_SZ    VM Session Service
    YourGUID\
        ElementName    REG_SZ    Your Service Friendly Name
```

> **注:** Linux ゲストのサービス GUID には、GUID ではなく `svm_cid` および `svm_port` によるアドレス指定が行われる VSOCK プロトコルが使用されます。 このような Windows との不整合を埋め合わせるために、よく知られている GUID がホスト上のサービス テンプレートとして使用され、ゲスト内のポートに変換されます。 サービス GUID をカスタマイズするには、先頭の "00000000" を必要なポート番号に変更します。 例:"00000ac9" は、ポート 2761 です。
> ```C++
> // Hyper-V Socket Linux guest VSOCK template GUID
> struct __declspec(uuid("00000000-facb-11e6-bd58-64006a7986d3")) VSockTemplate{};
>
>  /*
>   * GUID example = __uuidof(VSockTemplate);
>   * example.Data1 = 2761; // 0x00000AC9
>   */
> ```
>

> **クォータを調整する**PowerShell で GUID を生成し、それをクリップボードにコピーするには、次のコマンドを実行します。
>``` PowerShell
>(New-Guid).Guid | clip.exe
>```

## <a name="create-a-hyper-v-socket"></a>Hyper-V ソケットの作成

最も基本的な例で、ソケットの定義には、アドレス ファミリ、接続の種類、およびプロトコルが必要です。

次に、簡単な[ソケット定義](https://docs.microsoft.com/windows/desktop/api/winsock2/nf-winsock2-socket)を示します。

``` C
// Windows
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);

// Linux guest
int socket(int domain, int type, int protocol);
```

Hyper-V ソケットの場合:
* アドレス ファミリ - `AF_HYPERV` (Windows) または `AF_VSOCK` (Linux ゲスト)
* 種類 - `SOCK_STREAM`
* プロトコル - `HV_PROTOCOL_RAW` (Windows) または `0` (Linux ゲスト)


宣言/インスタンスの例を次に示します。
``` C
// Windows
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);

// Linux guest
int sock = socket(AF_VSOCK, SOCK_STREAM, 0);
```

## <a name="bind-to-a-hyper-v-socket"></a>Hyper-V ソケットへのバインド

バインドは、ソケットと接続情報を関連付けます。

便宜上、下に関数定義をコピーしていますが、バインドの詳細については[こちら](https://docs.microsoft.com/windows/desktop/api/winsock/nf-winsock-bind)をご覧ください。

``` C
// Windows
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);

// Linux guest
int bind(int sockfd, const struct sockaddr *addr,
         socklen_t addrlen);
```

ホスト マシンの IP アドレスとそのホスト上のポート番号から構成される標準インターネット プロトコル アドレス ファミリ (`AF_INET`) のソケット アドレス (sockaddr) と対照的に、`AF_HYPERV` のソケット アドレスでは、仮想マシンの ID と上に定義したアプリケーション ID を使用して、接続を確立します。 Linux ゲストからのバインドの場合、`AF_VSOCK` では `svm_cid` と `svm_port` が使用されます。

Hyper-V ソケットは、ネットワーク スタック、TCP/IP、DNS などに依存しないため、ソケット エンドポイントには、引き続きあいまいに接続を記述する、ホスト名ではない非 IP 形式が必要です。

Hyper-V ソケットのソケット アドレスの定義を次に示します。

``` C
// Windows
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};

// Linux guest
// See include/uapi/linux/vm_sockets.h for more information.
struct sockaddr_vm {
    __kernel_sa_family_t svm_family;
    unsigned short svm_reserved1;
    unsigned int svm_port;
    unsigned int svm_cid;
    unsigned char svm_zero[sizeof(struct sockaddr) -
                   sizeof(sa_family_t) -
                   sizeof(unsigned short) -
                   sizeof(unsigned int) - sizeof(unsigned int)];
};
```

IP またはホスト名の代わりに、AF_HYPERV エンドポイントは 2 つの GUID に大きく依存します。
* VM ID – これは、VM あたりに割り当てられた一意の ID です。  次の PowerShell サンプルを使用して、VM の ID を確認できます。
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* サービス ID - アプリケーションを Hyper-V ホスト レジストリに登録するための[上記の](#register-a-new-application) GUID。

接続が特定の仮想マシンとの接続でない場合は、一連の VMID ワイルドカードを使用することもできます。

### <a name="vmid-wildcards"></a>VMID ワイルドカード

| 名前 | GUID | 説明 |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | リスナーは、すべてのパーティションからの接続を受け入れるために、この VmId にバインドする必要があります。 |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | リスナーは、すべてのパーティションからの接続を受け入れるために、この VmId にバインドする必要があります。 |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | 子のワイルドカード アドレス。 リスナーは、その子からの接続を受け入れるために、この VmId にバインドする必要があります。 |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | ループバック アドレス。 この VmId を使用して、コネクタと同じパーティションに接続します。 |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | 親アドレス。 この VmId を使用して、コネクタの親パーティションに接続します。* |


\* `HV_GUID_PARENT` 仮想マシンの親は、その仮想マシンのホストです。  コンテナーの親は、コンテナーのホストです。
仮想マシンで実行しているコンテナーからの接続は、コンテナーをホストしている仮想マシンに接続します。
この VmId でリッスンし、次からの接続を受け入れます。(コンテナー内):コンテナー ホスト。
(VM 内:コンテナー ホスト/コンテナーなし):VM ホスト。
(VM 内なし:コンテナー ホスト/コンテナーなし):サポートされていません。

## <a name="supported-socket-commands"></a>サポートされているソケット コマンド

Socket() Bind() Connect() Send() Listen() Accept()

## <a name="useful-links"></a>役に立つリンク
[完全な WinSock API](https://docs.microsoft.com/windows/desktop/WinSock/winsock-functions)

[Hyper-V 統合サービスの参照](../reference/integration-services.md)
