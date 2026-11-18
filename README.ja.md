---
page_type: sample
description: "The Microsoft SysVAD Virtual Audio Device Driver (SYSVAD) shows how to develop a WDM audio driver that exposes support for multiple audio devices."
languages:
- cpp
products:
- windows
- windows-wdk
urlFragment: sysvad-virtual-audio-device-driver-sample
---

[English](README.md) | 日本語 | [中文](README.zh.md)

# SysVAD 仮想オーディオ デバイス ドライバー サンプル

> [!NOTE]
> [Windows 10 Mobile のサポート終了](https://support.microsoft.com/help/4484693/windows-10-mobile-end-of-support)に伴い、Phone Audio サンプルも SysVAD ドライバー サンプルから削除されました。

## はじめに

Microsoft SysVAD 仮想オーディオ デバイス ドライバー (SYSVAD) は、複数のオーディオ デバイスをサポートする WDM オーディオ ドライバーを開発する方法を示すサンプルです。

これらのオーディオ デバイスには、システムに組み込まれているもの (スピーカー、マイク アレイなど) と、接続可能なもの (ヘッドフォン、Bluetooth ヘッドセットなど) があります。このドライバーは、レンダリング デバイスに WaveRT とオーディオ オフロードを使用します。また、実際のハードウェア ベースのアダプターではなく「仮想オーディオ デバイス」を使用し、オーディオ オフロード WDM オーディオ ドライバー アーキテクチャのさまざまな側面を取り上げます。

ドライバー開発者は、このサンプルのフレームワークを使用して、ハードウェアへの依存を気にすることなく、さまざまなオーディオ デバイスをサポートできます。このフレームワークには、次のインターフェイスの実装が含まれています。

- *CAdapterCommon* インターフェイスは、ミニポートに仮想ミキサー ハードウェアへのアクセスを提供します。また、*IAdapterPowerManagement* インターフェイスも実装します。

- *CMiniportTopologySYSVAD* インターフェイスは、すべてのサンプル トポロジの基底クラスです。非常に基本的な共通関数を備えています。さらに、このクラスには共通のトポロジ プロパティ ハンドラーが含まれています。

次の表は、このサンプルの各サブディレクトリに実装されている機能を示しています。

| ディレクトリ | 説明 |
| --- | --- |
| TabletAudioSample | TabletAudioSample ドライバーに存在するエンドポイント。 |
| EndpointsCommon | TabletAudioSample と PhoneAudioSample の両方に存在するエンドポイント。両方のサンプル ドライバーで共有されるその他の共通コードも含まれます。 |
| SwapAPO | SysVAD サンプル ドライバーによって公開されるエンドポイントにインストールされ、左右のチャネルを入れ替えるサンプル APO。 |
| DelayAPO | 入力サンプルに遅延を追加するサンプル APO。 |
| KwsAPO | KSPROPERTY_INTERLEAVEDAUDIO_FORMATINFORMATION を使用して、キーワード スポッターのピンがループバック オーディオとマイク オーディオをインターリーブしているかどうかを判断し、どのチャネルにループバック オーディオが含まれているかを識別するサンプル APO。インターリーブされている場合、APO はループバック オーディオを取り除き、マイク オーディオのみをアップストリームに配信します。チャネル データが削除されるため、APO は入力形式とは異なる出力形式をネゴシエートします。 |
| AecAPO | Acoustic Echo Cancellation (音響エコー除去) を実装するサンプル キャプチャ MFX APO。この APO は、AEC APO がキャンセル用の参照オーディオを取得するために使用するエコー キャンセル インターフェイスの使用方法を示します。 |
| KeywordDetectorAdapter | サンプル キーワード検出アダプター。 |

Windows オーディオ エンジンの詳細については、「[Hardware-Offloaded Audio Processing](https://docs.microsoft.com/windows-hardware/drivers/audio/hardware-offloaded-audio-processing)」を参照してください。オフロード対応のオーディオ ハードウェアは、このトピックに示されている図のアーキテクチャを再現していることに注意してください。

## サンプルのビルド

このサンプル ドライバーをビルドするだけで、実行やテストを行う予定がない場合は、ターゲット コンピューター (テスト コンピューターとも呼ばれます) は必要ありません。ただし、このサンプル ドライバーを展開、実行、テストする場合は、ターゲット コンピューターとして機能する 2 台目のコンピューターが必要です。ターゲット コンピューターをセットアップする方法 (ターゲット コンピューターの*プロビジョニング*とも呼ばれます) は、[サンプルの実行](#サンプルの実行)に記載されています。

このサンプル ドライバーをビルドするには、次の手順を実行します。

### 開発コンピューターをセットアップする

サンプル ドライバーをビルドするには、次のコンポーネントがインストールされている必要があります。
- Visual Studio、
- Windows SDK、
- Windows Driver Kit (WDK)。

それぞれのダウンロードとインストールの手順については、「[Download the Windows Driver Kit](https://docs.microsoft.com/windows-hardware/drivers/download-the-wdk)」を参照してください。

Visual Studio インストーラーから、SysVAD をビルドするために次の個別パッケージをインストールする必要があります。
- C++ ATL for latest v142 build tools \<arch\>
- C++ ATL for latest v142 build tools with Spectre Mitigations \<arch\>
- MSVC v142 - VS 2019 C++ \<arch\> build tools (Latest)
- MSVC v142 - VS 2019 C++ \<arch\> build tools Spectre-mitigated libs (Latest)

ここで、「\<arch\>」はターゲット アーキテクチャを指します。

#### Windows Implementation Library (WIL) 用にリポジトリを構成する

プロジェクトをビルドするには、WIL を含める必要があります。Windows Driver Samples リポジトリには、WIL が Git サブモジュールとして含まれています。

まだ実行していない場合 (新規クローン時など) は、リポジトリのルートで次のコマンドを実行します。

`git submodule update --init`

これにより、リポジトリのルートに *wil* という名前のフォルダーが作成されます。

### Visual Studio でドライバー ソリューションを開く

Microsoft Visual Studio で、*File* \> *Open* \> *Project/Solution...* の順にクリックし、サンプル ファイルを含むフォルダー (例: *C:\Windows-driver-samples\audio\sysvad*) に移動します。*sysvad* ソリューション ファイルをダブルクリックします。

Visual Studio でソリューション エクスプローラーを開きます (まだ開いていない場合は、*View* メニューから *Solution Explorer* を選択します)。ソリューション エクスプローラーには、8 つのプロジェクトを含む 1 つのソリューションが表示されます。

### サンプルの構成とプラットフォームを設定する

*Solution 'sysvad' (8 of 8 projects)* を再度右クリックし、*Configuration Manager* を選択します。8 つのプロジェクトすべてで構成とプラットフォームの設定が同じであることを確認します。既定では、すべてのプロジェクトの構成が *Debug*、プラットフォームが *Win32* に設定されています。1 つのプロジェクトで構成やプラットフォームを変更した場合は、残りのすべてのプロジェクトでも同じ変更を行う必要があります。

### Visual Studio を使用してサンプルをビルドする

Visual Studio で、*Build* \> *Build Solution* の順にクリックします。

### ビルドされたドライバー パッケージを探す

ファイル エクスプローラーで、サンプル ファイルを含むフォルダーに移動します。たとえば、前の手順 1 で指定したフォルダーが *C:\\Windows-driver-samples\\audio\\sysvad* である場合は、そのフォルダーに移動します。

フォルダー内でのドライバー パッケージの場所は、*Configuration Manager* で選択した構成とプラットフォームの設定によって異なります。たとえば、既定の設定を変更しなかった場合、ビルドされたドライバー パッケージは、サンプル ファイルと同じフォルダー内の *Debug* という名前のフォルダーに保存されます。ビルドされたドライバー パッケージのフォルダーをダブルクリックし、次に *package* という名前のフォルダーをダブルクリックします。

パッケージには次のファイルが含まれています。

| ファイル | 説明 |
| --- | --- |
| TabletAudioSample.sys | ドライバー ファイル。 |
| DelayAPO.dll | 遅延 APO。 |
| KeywordDetectorContosoAdapter.dll | サンプル キーワード検出アダプター。 |
| AecApo.dll | AEC APO。 |
| KWSApo.dll | KWS APO。 |
| SwapAPO.dll | スワップ APO。 |
| sysvad.cat | パッケージ全体の署名として機能する、署名済みのカタログ ファイル。 |
| ComponentizedApoSample.inf | APO デバイスをインストールするコンポーネント化情報 (INF) ファイル。 |
| ComponentizedAudioSample.inf | Tablet Audio Sample ドライバーのインストールに必要な情報を含むコンポーネント化情報 (INF) ファイル。 |
| ComponentizedAudioSampleExtension.inf | APO デバイスを関連付けることで Tablet Audio Sample ドライバーの機能を拡張する拡張情報 (INF) ファイル。 |
| TabletAudioSample.inf | ドライバーのインストールに必要な情報を含む非コンポーネント化情報 (INF) ファイル。 |

拡張 INF ファイルの詳細については、「[Using an extension INF file](https://docs.microsoft.com/windows-hardware/drivers/install/using-an-extension-inf-file)」を参照してください。

## サンプルの実行

ドライバーをインストールするコンピューターは、*ターゲット コンピューター*または*テスト コンピューター*と呼ばれます。通常、これはドライバー パッケージを開発およびビルドするコンピューターとは別のコンピューターです。ドライバーを開発およびビルドするコンピューターは、*ホスト コンピューター*と呼ばれます。

ドライバー パッケージをターゲット コンピューターに移動してドライバーをインストールするプロセスは、ドライバーの*展開*と呼ばれます。TabletAudioSample サンプル ドライバーは、自動または手動で展開できます。

### ターゲット コンピューターをセットアップする

まず、最新の [Windows Driver Kit](https://docs.microsoft.com/windows-hardware/drivers/download-the-wdk) (WDK) をターゲット コンピューターにインストールします。

ドライバーを手動で展開する前に、テスト署名を有効にし、証明書をインストールして、ターゲット コンピューターを準備する必要があります。また、WDK のインストール場所で DevCon ツールを見つける必要があります。その後、ビルドしたドライバー サンプルを実行する準備が整います。

管理者としてコマンド プロンプト ウィンドウを開きます。次に、次のコマンドを入力します。

`bcdedit /set TESTSIGNING ON`

その後、ターゲット コンピューターを再起動します。

> [!IMPORTANT]
> BCDEdit を使用して起動情報を変更する前に、テスト PC の BitLocker や Secure Boot などの Windows セキュリティ機能を一時的に無効にする必要がある場合があります。

テストが完了したらこれらのセキュリティ機能を再度有効にし、セキュリティ機能が無効になっている間はテスト PC を適切に管理してください。

再起動後、WDK のインストール場所にある Tools フォルダーに移動し、DevCon ツールを見つけます。たとえば、次のフォルダーを確認します。

C:\\Program Files (x86)\\Windows Kits\\10\\Tools\\x64\\devcon.exe

*devcon.exe* を、ターゲット コンピューター上で見つけやすいフォルダーにコピーします。たとえば、*C:\\Tools* フォルダーを作成し、*devcon.exe* をそのフォルダーにコピーします。

ビルドされたドライバー パッケージ用のフォルダーをターゲット コンピューターに作成します (例: *C:\\SysvadDriver*)。ホスト コンピューター上でビルドされたドライバー パッケージからすべてのファイルをコピーし、ターゲット コンピューターに作成したフォルダーに保存します。

ビルド プロセスで作成された証明書用のフォルダーをターゲット コンピューターに作成します。たとえば、ターゲット コンピューターに *C:\\Certificates* という名前のフォルダーを作成し、ホスト コンピューターから *package.cer* をそのフォルダーにコピーします。この証明書は、ビルドされたドライバー ファイルを含む *package* フォルダーと同じ、ホスト コンピューター上のフォルダーにあります。ターゲット コンピューターで証明書ファイルを右クリックし、*Install* をクリックして、画面の指示に従ってテスト証明書をインストールします。

ターゲット コンピューターのセットアップに関する詳細な手順については、「[Preparing a Computer for Manual Driver Deployment](https://docs.microsoft.com/windows-hardware/drivers/develop/preparing-a-computer-for-manual-driver-deployment)」を参照してください。

#### 署名に関する注意

これらのバイナリ ファイルのほとんどはカーネル モードで実行されるため、署名されていること、およびオプションでカーネル デバッガーがアタッチされていることが重要です。

署名もカーネル デバッガーもない場合、ドライバーはターゲット コンピューターにインストールされません。カーネル デバッガーがアタッチされている場合は、ドライバーをインストールしてドライバー ファイル (.sys 拡張子) を読み込むことができますが、ユーザー モード ファイル (.dll ファイル) は読み込まれません。

ドライバー サンプル全体をインストールして実行する唯一の方法は、すべてのファイル (.sys、.dll、.cat) に信頼された証明書で署名することです。これにより、カーネル デバッガーがアタッチされていなくても、ドライバー全体を読み込むことができます。

この件の詳細については、「[Driver signing](https://docs.microsoft.com/windows-hardware/drivers/install/driver-signing)」を参照してください。

### ドライバーをインストールする

#### コンポーネント化 INF ファイル

TabletAudioSample ドライバー パッケージには、サンプル ドライバー、拡張サンプル、APO ソフトウェア コンポーネントが含まれています。次の手順では、サンプル ドライバー、APO コンポーネント、およびソフトウェア コンポーネントをドライバーに適用する拡張をインストールしてテストする方法を示します。

まず、基本 INF である *ComponentizedAudioSample.inf* をインストールする必要があります。インストールするには、ターゲット コンピューターで管理者としてコマンド プロンプト ウィンドウを開き、ドライバー パッケージ フォルダーに移動して、次のコマンドを入力します。

`devcon install ComponentizedAudioSample.inf Root\Sysvad_ComponentizedAudioSample`

*devcon* が認識されないというエラー メッセージが表示された場合は、*devcon* ツールへのパスを追加してみてください。たとえば、*C:\\Tools* という名前のフォルダーにコピーした場合は、次のコマンドを使用してみてください。

`C:\\tools\\devcon install ComponentizedAudioSample.inf Root\Sysvad_ComponentizedAudioSample`

これにより、ハードウェア ID が "Root\Sysvad_ComponentizedAudioSample" の基本 Tablet Audio Sample ドライバーがインストールされます。

次に、拡張 INF (*ComponentizedAudioSampleExtension.inf*) または APO INF (*ComponentizedApoSample.inf*) のいずれかをインストールできます。

拡張 INF を最初にインストールすると、*Audio Proxy APO Sample* という名前の新しいコンポーネントが作成されます。APO がインストールされるまで、このコンポーネントにはドライバーがまだアタッチされていないため、デバイス マネージャーの [ソフトウェア コンポーネント] セクションに表示されます。ドライバーは APO INF をインストールすることによってアタッチされます。

APO を最初にインストールするとドライバーはインストールされますが、デバイスは作成されないため、拡張 INF によってソフトウェア コンポーネントが作成され、そのコンポーネントに APO ドライバーがインストールされるまで、APO はデバイス マネージャーに表示されません。

拡張 INF と APO INF はどちらも、いずれかのファイルを右クリックしてメニューから [インストール] を選択することでインストールできます。

3 つのコンポーネント化 INF ファイルがインストールされると、APO デバイスがデバイス マネージャーに表示され、*Microsoft Virtual Audio Device (WDM) - Tablet Audio Sample* デバイスの名前が *SYSVAD (with APO Extensions)* に変わります。

詳細な手順については、「[Provision a computer for driver deployment and testing](https://docs.microsoft.com/windows-hardware/drivers/gettingstarted/provision-a-target-computer-wdk-8-1)」を参照してください。

#### 単一の INF ファイル

TabletAudioSample には、単一の INF ファイルを使用してサンプル ドライバーをインストールする tabletaudiosample.inf という名前の INF ファイルも含まれています。

Windows 10 1809 リリース以降、コンポーネント化ドライバー パッケージが必要です。ただし、下位互換性のために、TabletAudioSample.inf をインストールする方法を以下に示します。

ターゲット コンピューターで、管理者としてコマンド プロンプト ウィンドウを開きます。ドライバー パッケージ フォルダーに移動し、次のコマンドを入力します。

`devcon install TabletAudioSample.inf Root\Sysvad_TabletAudioSample`

サンプル ドライバーが正常にインストールされたら、テストする準備が整います。

### ドライバーをテストする

ターゲット コンピューターのコマンド プロンプト ウィンドウで、*devmgmt.msc* を入力してデバイス マネージャーを開きます。デバイス マネージャーの *表示* メニューで、*種類別にデバイス* を選択します。デバイス ツリーで *SYSVAD (with APO Extensions)* を見つけます。通常、これは *サウンド、ビデオ、およびゲーム コントローラー* ノードの下にあります。

ターゲット コンピューターでコントロール パネルを開き、*ハードウェアとサウンド* \> *サウンド デバイスの管理* に移動します。サウンド ダイアログ ボックスで、*SYSVAD (with APO Extensions)* というラベルの付いたスピーカー アイコンを選択し、*既定に設定* をクリックします。ただし、*OK* はクリックしないでください。これにより、サウンド ダイアログ ボックスが開いたままになります。

ターゲット コンピューターで MP3 などのオーディオ ファイルを見つけ、ダブルクリックして再生します。次に、サウンド ダイアログ ボックスで、*SYSVAD (with APO Extensions)* ドライバーに関連付けられている音量レベル インジケーターにアクティビティがあることを確認します。

## HLK テスト

ここにアップロードされているサンプルは、現在のプレイリストのすべてのオーディオ テストに合格することを確認するために、利用可能な最新バージョンの HLK を使用してテストされています。ただし、このドライバーは仮想オーディオ ドライバーであるため、オーディオ ミキシングを実装しておらず、トーンを生成してキャプチャとループバックをシミュレートします。これらの制限により、説明されている機能に依存する一部の HLK テストは失敗することが想定されています。

オーディオ テストの場合、これらの例外の 1 つが Hardware Offload of Audio Processing Test です。このテストはオフロード機能をサポートするデバイスを対象としており、デバイスが適切な要件に準拠していることを確認するためのチェックを実行します。SysVAD の場合、このテストはオフロードとループバックを備えたエンドポイントで失敗します。

オフロードを備えたエンドポイントの場合、ドライバーにはオフロード ピンが含まれていますが、音量、ミュート、ピーク メーター ノードなどを備えたミキサーを実装していないため、テストは失敗します。ループバックを備えたエンドポイントの場合、ドライバーはホストやオフロード ピンでの実際のストリーム ミキシングを実行せずにサイン トーンを返すことでループバックをシミュレートするため、テストは失敗します。
