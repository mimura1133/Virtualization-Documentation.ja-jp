---
title: 分離モード
description: Hyper-V による分離とプロセスによる分離コンテナーの違いについて説明します。
keywords: Docker, コンテナー
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 362805fa230f461414ccc53643644f6c1b3474a8
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853956"
---
# <a name="isolation-modes"></a>分離モード

Windows コンテナーには、2 種類のランタイム分離モードがあります。`process` による分離と `Hyper-V` による分離です。 両方の分離モードで実行されるコンテナーは、同様に作成、管理され、機能します。 また、作成および使用するコンテナー イメージも同じです。 分離モードの違いは、コンテナー、ホスト オペレーティング システム、およびそのホストで実行されている他のすべてのコンテナーの間に生じる分離の程度です。

## <a name="process-isolation"></a>プロセスによる分離

これはコンテナーの "従来の" 分離モードであり、[Windows コンテナーの概要](../about/index.md)に関するページで説明されています。 プロセスによる分離を使用すると、名前空間、リソース管理、およびプロセス分離テクノロジを使用して実現する分離により、複数のコンテナー インスタンスを特定のホスト上で同時に実行できます。 このモードで実行する場合、コンテナーとホストは相互に同じカーネルを共有します。  これは、Linux コンテナーの実行方法とほぼ同じです。

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V による分離
この分離モードでは、セキュリティが強化され、ホストとコンテナーのバージョン間の互換性が広がります。 Hyper-V による分離では、ホスト上で複数のコンテナー インスタンスが同時に実行されます。ただし、各コンテナーは高度に最適化された仮想マシン内で実行され、独自のカーネルが効果的に取得されます。 仮想マシンが存在することで、各コンテナーとコンテナー ホストの間にハードウェア レベルの分離が実現します。

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>分離の例

### <a name="create-container"></a>コンテナーの作成

Docker を使用した Hyper-V による分離コンテナーの管理は、プロセスによる分離コンテナーの管理とほぼ同じです。 Docker を介して Hyper-V による分離を使用するコンテナーを作成するには、`--isolation` パラメーターを使用して `--isolation=hyperv` を設定します。

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Docker を介してプロセスによる分離を使用するコンテナーを作成するには、`--isolation` パラメーターを使用して `--isolation=process` を設定します。

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows Server 上で実行される Windows コンテナーは、既定でプロセスによる分離を使用して実行されます。 Windows 10 Pro および Enterprise 上で実行される Windows コンテナーは、既定で Hyper-V による分離を使用して実行されます。 2018 年 10 月の Windows 10 更新プログラム以降、Windows 10 Pro または Enterprise ホストを実行しているユーザーは、プロセスによる分離を使用して Windows コンテナーを実行できるようになりました。 ユーザーは、`--isolation=process` フラグを使用してプロセスによる分離を直接要求する必要があります。

> [!WARNING]
> Windows 10 Pro および Enterprise 上でのプロセスによる分離を使用した実行は、開発およびテストを目的としています。 ホストでは Windows 10 ビルド 17763 以降を実行している必要があり、Docker バージョンは Engine 18.09 以降である必要があります。
> 
> 運用環境のデプロイでは、引き続き Windows Server をホストとして使用する必要があります。 Windows 10 Pro および Enterprise 上でこの機能を使用することにより、ホストとコンテナーのバージョン タグが一致するようにする必要があります。そうしないと、コンテナーの起動に失敗したり、未定義の動作が発生したりする可能性があります。

### <a name="isolation-explanation"></a>分離の説明

この例は、プロセスによる分離と Hyper-V による分離の分離機能の違いを示しています。

ここでは、プロセスによる分離コンテナーが配置されており、長時間実行される ping プロセスをホストします。

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

`docker top` コマンドを使用すると、以下のようにコンテナー内の ping プロセスが返されます。 この例のプロセスの ID は 3964 です。

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

コンテナー ホストで `get-process` コマンドを使用して、ホストから実行中のすべての ping プロセスを返すことができます。 この例では、コンテナーの ID と一致するプロセス ID は 1 つです。 コンテナーとホストの両方で表示されるプロセスは同じです。

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

一方、この例では Hyper-V による分離コンテナーを起動して、ping プロセスも使用します。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

同様に、`docker top` を使用して、コンテナーから実行中のプロセスを返すことができます。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

ただし、コンテナー ホストでプロセスを検索しても、ping プロセスは検出されず、エラーはスローされます。

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

最終的に、ホストで `vmwp` プロセスが表示されます。実行中の仮想マシンは、実行中のコンテナーをカプセル化し、ホストのオペレーティング システムから実行中のプロセスを保護します。

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
