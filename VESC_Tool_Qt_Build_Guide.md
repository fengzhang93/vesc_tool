# VESC Tool 在 Qt 中的编译与发布指南 (基于 Qt 5.15.2 MinGW)

本文档详细介绍了如何在 Windows 环境下，使用 Qt Creator (基于 Qt 5.15.2 MinGW 编译器) 编译 VESC Tool 源码，并打包生成可独立运行的 `.exe` 程序的完整步骤。

## 1. 环境准备

在开始编译之前，请确保你的开发环境满足以下要求：

*   **操作系统**: Windows 10/11
*   **Qt 版本**: Qt 5.15.2
*   **编译器**: MinGW (推荐使用 64-bit 版本，如 MinGW 8.1.0 64-bit，这通常在安装 Qt 时一并勾选)
*   **Qt 组件**: 在安装 Qt 时，除了勾选 `MinGW 8.1.0 64-bit`，还需要勾选以下关键组件：
    *   `Qt Charts`
    *   `Qt Connectivity` (包含 Bluetooth 和 SerialPort，VESC Tool 通信强依赖此组件)
    *   `Qt Quick` 相关组件 (如果涉及到 Mobile 版本的编译，桌面版也可能用到部分 QML 组件)
*   **Git**: 用于克隆或管理 VESC Tool 源码。

## 2. 获取源码与工程导入

1.  **获取源码**: 将 VESC Tool 的源码下载到本地（例如你的工作目录 `C:\Users\zhang\Desktop\Tz_ProjectData\vesc_tool`）。
2.  **打开工程**:
    *   启动 Qt Creator。
    *   点击菜单栏的 `文件(File)` -> `打开文件或项目(Open File or Project...)`。
    *   导航到你的 VESC Tool 源码目录，找到并选中 `vesc_tool.pro` 文件，点击“打开”。
3.  **配置项目 (Configure Project)**:
    *   在弹出的“Configure Project”窗口中，Qt Creator 会列出可用的构建套件 (Kits)。
    *   勾选 **Desktop Qt 5.15.2 MinGW 64-bit**（或你安装的对应 MinGW 版本）。
    *   点击 `Configure Project` 按钮。

## 3. 在 Qt Creator 中编译与运行调试

导入工程后，你可以先在 Qt Creator 内部进行编译和运行测试。

1.  **选择构建配置**: 在 Qt Creator 左下角的计算机图标处，点击选择 `Release` 构建模式。
    *   *说明*: `Debug` 模式包含调试信息，生成的文件大且运行稍慢，通常只在开发查错时使用。最终要打包发布时，**必须选择 `Release` 模式**。
2.  **执行构建**: 点击左下角的“锤子”图标（或按 `Ctrl+B`）开始编译。
    *   编译过程中，可以在下方的“编译输出 (Compile Output)”窗口查看进度。
    *   如果遇到找不到头文件或库的错误，请检查你的 Qt 是否安装了 `Qt Connectivity` 和 `Qt SerialPort` 等必要组件。
3.  **运行测试**: 编译成功后，点击左下角的绿色“播放”按钮（或按 `Ctrl+R`）。VESC Tool 界面应该能正常弹出来，此时可以进行基本的功能测试。

### 重新编译 (修改代码或配置后)
如果你修改了源码（例如更新了 `vesc_tool.pro` 中的 `VT_VERSION = 8.00` 等参数），为了确保所有的更改都能正确生效，你需要进行一次完整的重新编译：
1. 在 Qt Creator 中，点击菜单栏的 `构建(Build)` -> 选择 **“清除项目 (Clean Project)”**（或者在项目树上右键选择“清除 (Clean)”）。
2. 点击 **“qmake”** 按钮，重新生成项目文件。
3. 清除完成后，重新点击左下角的 **“构建 (Build)”** 锤子图标，或直接点击 **“运行 (Run)”** 按钮。
*说明：这样编译出来的 VESC Tool 就会应用最新的配置（例如 `highest_supported` 也会随之变成 8.00，连接新版固件时就不会再弹出警告，受限通信模式也会解除）。*

## 4. 生成可独立运行的 exe 文件 (打包发布)

在 Qt Creator 中点击运行，程序是依赖 Qt 内部环境的。如果直接把生成的 `.exe` 复制到其他没有安装 Qt 的电脑上，会提示缺少各种 `.dll` 文件。因此，需要使用 Qt 提供的工具进行打包部署。

### 步骤 4.1: 找到编译生成的 Release 目录
1. 确保你已经在 Qt Creator 中以 **Release** 模式成功编译了项目。
2. 找到构建输出目录。默认情况下，Qt 会在源码同级目录下生成一个类似于 `build-vesc_tool-Desktop_Qt_5_15_2_MinGW_64_bit-Release` 的文件夹。
3. 进入该构建目录，再进入 `release` 文件夹。
4. 你会看到里面有一个生成的 `vesc_tool.exe` 文件。

### 步骤 4.2: 准备打包目录
1. 在桌面或其他方便的地方，新建一个空白文件夹，例如命名为 `VESC_Tool_Release`。
2. 将上一步找到的 `vesc_tool.exe` 复制到这个新建的 `VESC_Tool_Release` 文件夹中。

### 步骤 4.3: 使用 windeployqt 工具提取依赖 dll
`windeployqt` 是 Qt 官方提供的 Windows 部署工具，它会自动分析 exe 文件，并将所有依赖的 Qt dll 文件、插件复制到 exe 所在目录。

**重要说明**：VESC Tool 的右侧工具栏（以及部分其他界面）使用了 **QML/Qt Quick** 技术。如果不加特定参数，`windeployqt` 默认只会拷贝 C++ Widgets 相关的库，导致打包后运行出现**大片白色空白区域**。

1. **打开 Qt 命令行工具**:
   * 在 Windows 的开始菜单中，找到你的 Qt 安装目录夹（例如 `Qt 5.15.2`）。
   * 找到并运行 **`Qt 5.15.2 (MinGW 8.1.0 64-bit)`** 快捷方式。这会打开一个配置好 Qt 环境变量的命令行窗口。
   * *(重要提示：不要直接用 Windows 默认的 cmd，否则可能会因为环境变量问题找不到 windeployqt 命令)*。

2. **切换到打包目录**:
   在打开的 Qt 命令行窗口中，使用 `cd` 命令切换到你刚才新建的打包目录。
   例如：
   ```cmd
   cd /d C:\Users\zhang\Desktop\VESC_Tool_Release
   ```

3. **执行带有 QML 扫描路径的 windeployqt 命令 (关键步骤)**:
   为了让打包工具能正确提取 QML 运行环境，你需要使用 `--qmldir` 参数，并指向 VESC Tool 源码中包含 QML 文件的目录。在命令行中输入以下命令并回车（请将 `--qmldir` 后的路径替换为你实际的源码路径）：
   ```cmd
   windeployqt --qmldir C:\Users\zhang\Desktop\Tz_ProjectData\vesc_tool vesc_tool.exe
   ```
   *此时，你会看到命令行输出大量拷贝文件的信息。windeployqt 正在将需要的 Qt 核心库（如 Qt5Core, Qt5Gui, Qt5Qml, Qt5Quick 等）以及 QML 插件文件夹自动复制到该目录下。如果成功，你的打包目录下会出现一个 `QtQuick` 和其他相关文件夹。*

### 步骤 4.4: 补充 MinGW 编译器依赖 dll (如果需要)
虽然 `windeployqt` 能解决绝大部分 Qt 库的依赖，但由于我们使用的是 MinGW 编译器，有时程序还会依赖 MinGW 的 C/C++ 运行库（如 `libgcc_s_seh-1.dll`, `libstdc++-6.dll`, `libwinpthread-1.dll` 等）。

*   **测试方法**: 在你的 `VESC_Tool_Release` 文件夹中，双击运行 `vesc_tool.exe`。
*   **如果报错缺少 dll**: 记录下缺少的 dll 名称，然后去你的 Qt 安装目录下的 MinGW bin 文件夹中寻找。
    *   默认路径通常类似：`C:\Qt\5.15.2\mingw81_64\bin`
    *   找到缺失的 dll 文件，手动复制到 `VESC_Tool_Release` 文件夹中，与 `vesc_tool.exe` 放在同一级。
*   **如果能正常运行**: 恭喜，打包基本完成。

*(注：如果你在执行 `windeployqt` 时加上 `--compiler-runtime` 参数，如 `windeployqt --compiler-runtime vesc_tool.exe`，它通常会自动把 MinGW 的依赖库也拷贝过来。)*

## 5. 最终验证与分发

现在，你的 `VESC_Tool_Release` 文件夹中包含了 `vesc_tool.exe` 及其所有必需的依赖文件和文件夹。

*   你可以将这个整个 `VESC_Tool_Release` 文件夹压缩成一个 `.zip` 文件，发送给其他没有安装 Qt 环境的 Windows 用户。
*   他们解压后，直接双击里面的 `vesc_tool.exe` 即可正常运行 VESC Tool。

---
**附：常见问题排查**
*   **编译报错找不到 QSerialPort / QBluetooth**: 确认 Qt 安装时是否勾选了 `Qt Connectivity` 或相关的 SerialPort/Bluetooth 模块。如果没有，需要打开 Qt Maintenance Tool 添加该组件。
*   **打开打包后的 exe 闪退或没有任何反应**: 极少数情况下，可能缺少某些不在常规依赖列表中的动态库，或者某些插件加载失败。可以在命令行中运行 exe，看是否有错误信息输出；或者检查是否有杀毒软件拦截。