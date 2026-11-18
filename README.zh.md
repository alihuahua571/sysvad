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

[English](README.md) | [日本語](README.ja.md) | 中文

# SysVAD 虚拟音频设备驱动程序示例

> [!NOTE]
> 随着 [Windows 10 移动版支持终止](https://support.microsoft.com/help/4484693/windows-10-mobile-end-of-support)，Phone Audio 示例也已从 SysVAD 驱动程序示例中移除。

## 简介

Microsoft SysVAD 虚拟音频设备驱动程序 (SYSVAD) 展示了如何开发一个支持多种音频设备的 WDM 音频驱动程序。

这些音频设备中，有些内置于系统（例如扬声器、麦克风阵列），有些则可插拔（例如耳机、蓝牙耳麦等）。该驱动程序对呈现设备使用 WaveRT 和音频卸载。该驱动程序使用“虚拟音频设备”而非实际的基于硬件的适配器，并重点展示了音频卸载 WDM 音频驱动程序体系结构的各个方面。

驱动程序开发者可以使用此示例中的框架，在不担心硬件依赖关系的情况下为各种音频设备提供支持。该框架包含以下接口的实现：

- *CAdapterCommon* 接口让微型端口可以访问虚拟混音器硬件。它还实现了 *IAdapterPowerManagement* 接口。

- *CMiniportTopologySYSVAD* 接口是所有示例拓扑的基类。它包含非常基本的通用函数。此外，该类还包含通用的拓扑属性处理程序。

下表列出了此示例各子目录中实现的功能。

| 目录 | 说明 |
| --- | --- |
| TabletAudioSample | TabletAudioSample 驱动程序中存在的终结点。 |
| EndpointsCommon | TabletAudioSample 和 PhoneAudioSample 中都存在的终结点。它还包含两个示例驱动程序共享的其他通用代码。 |
| SwapAPO | 示例 APO，安装到 SysVAD 示例驱动程序公开的终结点上，用于交换左右声道。 |
| DelayAPO | 为输入采样添加延迟的示例 APO。 |
| KwsAPO | 示例 APO，它使用 KSPROPERTY_INTERLEAVEDAUDIO_FORMATINFORMATION 判断关键字检测器引脚是否将环回音频与麦克风音频交错，并确定哪些声道包含环回音频。如果音频是交错的，APO 会去除环回音频，仅将麦克风音频向上游传递。由于移除了声道数据，APO 会协商不同于输入格式的输出格式。 |
| AecAPO | 实现声学回声消除的示例捕获 MFX APO。此 APO 演示了 AEC APO 如何使用回声消除接口获取用于消除的参考音频。 |
| KeywordDetectorAdapter | 示例关键字检测器适配器。 |

有关 Windows 音频引擎的详细信息，请参阅[硬件卸载的音频处理](https://docs.microsoft.com/windows-hardware/drivers/audio/hardware-offloaded-audio-processing)，并注意支持卸载的音频硬件会复制该主题中示意图所展示的体系结构。

## 生成示例

如果你只想生成此示例驱动程序，而不打算运行或测试它，则不需要目标计算机（也称为测试计算机）。但是，如果你希望部署、运行和测试此示例驱动程序，则需要第二台计算机作为目标计算机。[运行示例](#运行示例)中提供了相关说明，介绍如何设置目标计算机，这也称为*预配*目标计算机。

执行以下步骤生成此示例驱动程序。

### 设置开发计算机

生成示例驱动程序必须安装以下组件：
- Visual Studio，
- Windows SDK，
- Windows 驱动程序工具包 (WDK)。

请参阅[下载 Windows 驱动程序工具包](https://docs.microsoft.com/windows-hardware/drivers/download-the-wdk)，了解如何下载并安装各个组件。

在 Visual Studio 安装程序中，必须安装以下单个组件才能生成 SysVAD：
- 适用于最新 v142 生成工具的 C++ ATL \<arch\>
- 适用于最新 v142 生成工具并带 Spectre 缓解措施的 C++ ATL \<arch\>
- MSVC v142 - VS 2019 C++ \<arch\> 生成工具（最新）
- MSVC v142 - VS 2019 C++ \<arch\> 生成工具 Spectre 缓解库（最新）

其中“\<arch\>”指你的目标体系结构。

#### 为 Windows 实现库 (WIL) 配置存储库

若要生成项目，必须包含 WIL。Windows 驱动程序示例存储库将 WIL 作为 Git 子模块包含在内。

如果之前尚未执行过此操作（例如在全新克隆时），则应在存储库根目录运行以下命令：

`git submodule update --init`

这会在存储库根目录填充一个名为 *wil* 的文件夹。

### 在 Visual Studio 中打开驱动程序解决方案

在 Microsoft Visual Studio 中，依次单击*文件* \> *打开* \> *项目/解决方案...*，然后导航到包含示例文件的文件夹（例如 *C:\Windows-driver-samples\audio\sysvad*）。双击 *sysvad* 解决方案文件。

在 Visual Studio 中找到解决方案资源管理器。（如果它尚未打开，请从*视图*菜单中选择*解决方案资源管理器*。）在解决方案资源管理器中，可以看到一个包含八个项目的解决方案。

### 设置示例的配置和平台

再次右键单击*解决方案 'sysvad'（8 个项目中的 8 个）*，然后选择*配置管理器*。确保八个项目的配置和平台设置相同。默认情况下，所有项目的配置都设置为 *Debug*，平台都设置为 *Win32*。如果对某个项目进行了任何配置和/或平台更改，则必须对所有其余项目进行相同的更改。

### 使用 Visual Studio 生成示例

在 Visual Studio 中，依次单击*生成* \> *生成解决方案*。

### 找到生成的驱动程序包

在文件资源管理器中，导航到包含示例文件的文件夹。例如，如果前面的第 1 步中指定的是 *C:\\Windows-driver-samples\\audio\\sysvad*，则导航到该文件夹。

在该文件夹中，驱动程序包的位置取决于你在*配置管理器*中选择的配置和平台设置。例如，如果你保留默认设置不变，则生成的驱动程序包将保存到示例文件所在文件夹内名为 *Debug* 的文件夹中。双击生成的驱动程序包文件夹，然后双击名为 *package* 的文件夹。

该包应包含以下文件：

| 文件 | 说明 |
| --- | --- |
| TabletAudioSample.sys | 驱动程序文件。 |
| DelayAPO.dll | 延迟 APO。 |
| KeywordDetectorContosoAdapter.dll | 示例关键字检测器适配器。 |
| AecApo.dll | AEC APO。 |
| KWSApo.dll | KWS APO。 |
| SwapAPO.dll | 交换 APO。 |
| sysvad.cat | 已签名的目录文件，用作整个包的签名。 |
| ComponentizedApoSample.inf | 安装 APO 设备的组件化信息 (INF) 文件。 |
| ComponentizedAudioSample.inf | 包含安装 Tablet Audio Sample 驱动程序所需信息的组件化信息 (INF) 文件。 |
| ComponentizedAudioSampleExtension.inf | 通过将 APO 设备关联到 Tablet Audio Sample 驱动程序来扩展其功能的扩展信息 (INF) 文件。 |
| TabletAudioSample.inf | 包含安装驱动程序所需信息的非组件化信息 (INF) 文件。 |

有关扩展 INF 文件的详细信息，请参阅[使用扩展 INF 文件](https://docs.microsoft.com/windows-hardware/drivers/install/using-an-extension-inf-file)。

## 运行示例

安装驱动程序的计算机称为*目标计算机*或*测试计算机*。通常，它与开发和生成驱动程序包的计算机是分开的。开发和生成驱动程序的计算机称为*主机*。

将驱动程序包移动到目标计算机并安装驱动程序的过程称为*部署*驱动程序。可以自动或手动部署 TabletAudioSample 示例驱动程序。

### 设置目标计算机

首先，在目标计算机上安装最新的 [Windows 驱动程序工具包](https://docs.microsoft.com/windows-hardware/drivers/download-the-wdk) (WDK)。

在手动部署驱动程序之前，必须通过开启测试签名并安装证书来准备目标计算机。还需要在 WDK 安装位置中找到 DevCon 工具。之后，即可运行生成的驱动程序示例。

以管理员身份打开命令提示符窗口。然后输入以下命令：

`bcdedit /set TESTSIGNING ON`

然后重新启动目标计算机。

> [!IMPORTANT]
> 在使用 BCDEdit 更改启动信息之前，你可能需要暂时挂起测试电脑上的 BitLocker 和安全启动等 Windows 安全功能。

测试完成后，请重新启用这些安全功能，并在安全功能处于禁用状态期间妥善管理测试电脑。

重新启动后，导航到 WDK 安装位置中的 Tools 文件夹，找到 DevCon 工具。例如，查看以下文件夹：

C:\\Program Files (x86)\\Windows Kits\\10\\Tools\\x64\\devcon.exe

将 *devcon.exe* 复制到目标计算机上更容易找到的文件夹。例如，创建 *C:\\Tools* 文件夹，并将 *devcon.exe* 复制到该文件夹。

在目标计算机上为生成的驱动程序包创建一个文件夹（例如 *C:\\SysvadDriver*）。将主机上生成的驱动程序包中的所有文件复制并保存到你在目标计算机上创建的文件夹中。

在目标计算机上为生成过程创建的证书创建一个文件夹。例如，可以在目标计算机上创建一个名为 *C:\\Certificates* 的文件夹，然后从主机将 *package.cer* 复制到其中。你可以在主机上与包含生成的驱动程序文件的 *package* 文件夹相同的文件夹中找到此证书。在目标计算机上，右键单击该证书文件，单击*安装*，然后按照提示安装测试证书。

如果需要有关设置目标计算机的更详细说明，请参阅[准备计算机以进行手动驱动程序部署](https://docs.microsoft.com/windows-hardware/drivers/develop/preparing-a-computer-for-manual-driver-deployment)。

#### 关于签名的说明

由于这些二进制文件大多在内核模式下执行，因此对它们进行签名很重要，另外，附加内核调试器也是可选的。

如果没有签名或内核调试器，驱动程序将无法安装到目标计算机上。如果附加了内核调试器，则可以安装驱动程序并加载驱动程序文件（.sys 扩展名），但不会加载任何用户模式文件（.dll 文件）。

安装并执行整个驱动程序示例的唯一方法是，使用受信任的证书对所有文件（.sys、.dll 和 .cat）进行签名。这样，即使未附加内核调试器，也能加载整个驱动程序。

有关此主题的详细信息，请参阅[驱动程序签名](https://docs.microsoft.com/windows-hardware/drivers/install/driver-signing)。

### 安装驱动程序

#### 组件化 INF 文件

TabletAudioSample 驱动程序包包含示例驱动程序、扩展示例和 APO 软件组件。以下说明介绍了如何安装并测试示例驱动程序、APO 组件，以及随后用于将软件组件应用到驱动程序的扩展。

首先，必须安装基础 INF 文件 *ComponentizedAudioSample.inf*。要安装它，请在目标计算机上以管理员身份打开命令提示符窗口，然后导航到驱动程序包文件夹并输入以下命令：

`devcon install ComponentizedAudioSample.inf Root\Sysvad_ComponentizedAudioSample`

如果收到无法识别 *devcon* 的错误消息，请尝试添加 *devcon* 工具的路径。例如，如果你将其复制到名为 *C:\\Tools* 的文件夹，请尝试使用以下命令：

`C:\\tools\\devcon install ComponentizedAudioSample.inf Root\Sysvad_ComponentizedAudioSample`

这会安装基础 Tablet Audio Sample 驱动程序，其硬件 ID 为“Root\Sysvad_ComponentizedAudioSample”。

然后，可以安装扩展 INF 文件 (*ComponentizedAudioSampleExtension.inf*) 或 APO INF 文件 (*ComponentizedApoSample.inf*)。

如果先安装扩展 INF，它将创建一个名为 *Audio Proxy APO Sample* 的新组件。在安装 APO 之前，此组件会显示在设备管理器的“软件组件”部分下，因为它尚未附加驱动程序。安装 APO INF 后会附加驱动程序。

如果先安装 APO，它会安装驱动程序，但不会创建设备，因此在扩展 INF 创建软件组件并随后安装 APO 驱动程序之前，APO 不会显示在设备管理器中。

通过右键单击任一文件并从菜单中选择“安装”，可以安装扩展 INF 和 APO INF。

安装三个组件化 INF 文件后，设备管理器中应出现一个 APO 设备，并且 *Microsoft Virtual Audio Device (WDM) - Tablet Audio Sample* 设备现在应命名为 *SYSVAD (with APO Extensions)*。

有关更详细的说明，请参阅[预配计算机以进行驱动程序部署和测试](https://docs.microsoft.com/windows-hardware/drivers/gettingstarted/provision-a-target-computer-wdk-8-1)。

#### 单一 INF 文件

TabletAudioSample 还包含一个 INF 文件 tabletaudiosample.inf，它使用单一 INF 文件安装示例驱动程序。

从 Windows 10 版本 1809 开始，要求使用组件化驱动程序包。不过，为了向后兼容，下面提供了有关如何安装 TabletAudioSample.inf 的说明。

在目标计算机上，以管理员身份打开命令提示符窗口。导航到驱动程序包文件夹，然后输入以下命令：

`devcon install TabletAudioSample.inf Root\Sysvad_TabletAudioSample`

成功安装示例驱动程序后，即可开始测试它。

### 测试驱动程序

在目标计算机上，在命令提示符窗口中输入 *devmgmt.msc* 以打开设备管理器。在设备管理器中，从*查看*菜单中选择*按类型列出设备*。在设备树中找到 *SYSVAD (with APO Extensions)*。它通常位于*声音、视频和游戏控制器*节点下。

在目标计算机上打开控制面板，导航到*硬件和声音* \> *管理音频设备*。在“声音”对话框中，选择标记为 *SYSVAD (with APO Extensions)* 的扬声器图标，然后单击*设为默认值*，但不要单击*确定*。这样可以使“声音”对话框保持打开状态。

在目标计算机上找到一个 MP3 或其他音频文件，双击播放它。然后在“声音”对话框中，确认与 *SYSVAD (with APO Extensions)* 驱动程序关联的音量级别指示器有活动。

## HLK 测试

此处上传的示例使用可用的最新 HLK 版本进行测试，以确保它通过当前播放列表中的所有音频测试。但是，由于它是虚拟音频驱动程序，因此不实现音频混合，而是通过生成音调来模拟捕获和环回。鉴于这些限制，某些 HLK 测试预计会失败，因为它们依赖于所述功能。

在音频测试方面，其中一项例外是“音频处理的硬件卸载测试”(Hardware Offload of Audio Processing Test)。此测试面向支持卸载功能的设备，并执行检查以确保设备符合相应要求。在 SysVAD 的特定情况下，对于具有卸载和环回的终结点，此测试将失败。

对于具有卸载的终结点，测试将失败，因为驱动程序包含卸载引脚，但没有实现带有音量、静音和峰值计量表节点等的混音器。对于具有环回的终结点，测试将失败，因为驱动程序通过返回正弦音而不是在主机和/或卸载引脚中执行真正的流混合来模拟环回。
