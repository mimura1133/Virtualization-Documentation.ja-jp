# <a name="build-and-run-an-application-with-or-without-net-core-20-or-powershell-core-6"></a>.NET Core 2.0 または PowerShell Core 6 を使用した、または使用しないアプリケーションの構築と実行

このリリースの Nano Server の基本 OS コンテナー イメージでは、.NET Core と PowerShell が削除されていますが、NET Core と PowerShell はいずれも、基本 Nano Server コンテナーの上のアドオン複数層コンテナとしてサポートされています。  

コンテナーがネイティブ コードまたは Node.js、Python、Ruby などのオープン フレームワークを実行する場合は、基本 Nano Server コンテナーで十分です。  1 つの小さな相違点として、このリリースは、Windows Server 2016 リリースと比較して、[フット プリントが削減](https://docs.microsoft.com/windows-server/get-started/nano-in-semi-annual-channel)されているため、特定のネイティブ コードが実行できないことがあります。 以前のバージョンにはなかった不具合が発生した場合は、[フォーラム](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)を通じてマイクロソフトにお知らせください。 

Dockerfile からコンテナーを構築するには docker build を使用し、それを実行するには docker run を使用します。  次のコマンドを実行すると、Nano Server コンテナーの基本 OS イメージがダウンロードされ (数分かかることがあります)、"Hello World!" というメッセージがホストのコンソールに表示されます。

```
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

[Windows 上の Dockerfile](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile) で、FROM、RUN、COPY、ADD、CMD などの Dockerfile 構文を使用すると、さらに複雑なアプリケーションを構築できます。この基本イメージでは、一部のコマンドが即座に実行できなくなりますが、アプリケーションの機能に必要な要素のみを含むコンテナ イメージを作成できるようになります。

基本 Nano Server コンテナー OS イメージでは、.NET Core と PowerShell がいずれも利用できないため、圧縮 zip 形式のコンテンツを含むコンテナーの作成方法が 1 つの課題となります。 Docker 17.05 に搭載されている[多段階ビルド](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)機能を使用すると、別のコンテナの PowerShell を利用して、コンテンツを解凍し、Nano コンテナーにコピーできます。 このアプローチを使用すると、.NET Core コンテナーと PowerShell コンテナーを作成することができます。 

PowerShell コンテナー イメージのプルには、次のコマンドを使用できます。

```
docker pull microsoft/nanoserver-insider-powershell
```

.NET Core コンテナー イメージのプルには、次のコマンドを使用できます。

```
docker pull microsoft/nanoserver-insider-dotnet
```

以下では、多段階ビルドを使用して、これらのコンテナー イメージを作成した方法の例を示します。

## <a name="deploy-apps-based-on-net-core-20"></a>.NET Core 2.0 に基づくアプリの展開
.NET Core アプリケーションが別の場所に構築されていて、それをコンテナ内で実行しようとする場合は、Insider リリースで .NET Core 2.0 コンテナ イメージを利用して .NET Core アプリを実行できます。  .NET Core コンテナ イメージを使って .NET Core アプリケーションを実行する方法について詳しくは、[.NET Core GitHub](https://github.com/dotnet/dotnet-docker-nightly) をご覧ください。  コンテナー内部にアプリケーションを開発する場合は、.NET Core SDK を代わりに使用する必要があります。  上級ユーザー向けの方法として、.NET Core 2.0 バージョン、Dockerfile、[dotnet docker 夜間](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0) で指定された URL を使用して、独自の .NET Core 2.0 コンテナーを構築することができます。 これには、Windows Server Core コンテナーを使用して、機能をダウンロードして解凍します。  Dockerfile のサンプルについては、[.NET Core Runtime Dockerfile](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/amd64/Dockerfile) をご覧ください。


この Dockerfile では、次のコマンドを使用して .NET Core 2.0 のコンテナーを作成できます。

```
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## <a name="run-powershell-core-6-in-a-container"></a>コンテナーでの PowerShell Core 6 の実行
同じ[多段階ビルド](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)の手法を使用して、[この PowerShell Dockerfile](https://github.com/PowerShell/PowerShell-Docker/blob/master/release/stable/nanoserver/docker/Dockerfile) で PowerShell Core 6 コンテナーを作成できます。


その後、docker build コマンドを発行して、PowerShell コンテナー イメージを作成します。

``` 
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

詳しくは、[PowerShell GitHub](https://github.com/PowerShell/PowerShell-Docker/tree/master/release) をご覧ください。  PowerShell の zip には、PowerShell Core 6 の構築に必要な .NET Core 2.0 のサブセットが含まれていることに注意してください。  使用する PowerShell モジュールが .NET Core 2.0 に依存している場合は、基本 Nano コンテナーではなく、Nano .NET Core コンテナー上に PowerShell コンテナーを構築するのが安全です。これには、Dockerfile で、FROM microsoft/nanoserver-insider-dotnet を使用します。 

## <a name="next-steps"></a>次のステップ
- Docker Hub にある Nano Server ベースの新しいコンテナー イメージ (基本 Nano Server イメージ、Nano/.NET Core 2.0、Nano/PowerShell Core 6) のいずれかを使用する
- このガイドに含まれている Dockerfile コンテンツのサンプルを使用し、新しい Nano Server コンテナー基本 OS イメージに基づいて、独自のコンテナー イメージを構築する 
