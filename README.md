# QT Static Build

使用 GitHub Actions 在云端编译 **QT 静态库**（MSVC 2022 / x64），支持任意 QT 版本，手动触发、自动下载源码、打包发布。

## 使用方法

1. 推送本仓库到 GitHub
2. 打开 **Actions** → **Build Static QT (MSVC 2022)** → **Run workflow**
3. **`qt_version` 填写目标版本**（如 `6.11.1`；留空自动使用最新版）
4. 复制下方 **推荐编译参数**，粘贴到 **编译参数** 输入框
5. 点击 **Run workflow**，完成后在 **Releases** 或 **Artifacts** 下载 `.7z`

| 参数 | 说明 |
|------|------|
| `qt_version` | QT 版本，如 `6.11.1`、`6.8.0`；留空自动使用最新版 |
| **编译参数** | **必填**，完整 `configure.bat` 参数，直接复制下方推荐配置 |
| `parallel_jobs` | 并行编译线程数；留空使用 jom 默认 |
| `create_release` | 是否创建 GitHub Release（标签与标题按本次编译版本自动生成） |

解压安装包到 `C:\Qt\static-msvc2022` 后，在 CMake 中设置：

```cmake
set(CMAKE_PREFIX_PATH "C:/Qt/static-msvc2022")
```

---

## 推荐编译参数（以 QT 6.11.1 为例，复制粘贴）

适用于 **QWidget / QML 本地桌面应用**，**不含嵌入式浏览器内核**（Chromium / WebEngine），并排除 IoT、地图、3D 等大型可选模块。其他版本请自行调整 `-skip` 列表。

**整段复制，粘贴到「编译参数」即可：**

```text
-prefix C:/Qt/static-msvc2022 -static -static-runtime -release -opensource -confirm-license -nomake examples -nomake tests -platform win32-msvc -cmake-generator "NMake Makefiles" -skip qtwebengine -skip qtwebview -skip qtwebchannel -skip qt3d -skip qtcharts -skip qtcoap -skip qtconnectivity -skip qtdatavis3d -skip qtdoc -skip qtgraphs -skip qtgrpc -skip qthttpserver -skip qtlanguageserver -skip qtlocation -skip qtpositioning -skip qtquick3d -skip qtquick3dphysics -skip qtremoteobjects -skip qtscxml -skip qtsensors -skip qtserialbus -skip qtserialport -skip qtshadertools -skip qtcanvaspainter -skip qtmultimedia -skip qtspeech -skip qtvirtualkeyboard -skip qtwayland -skip qtwebsockets -skip qtlottie -skip qtmqtt -skip qtnetworkauth
```

### 保留的主要模块

| 模块 | 用途 |
|------|------|
| `qtbase` | 核心：窗口、控件、网络、文件等 |
| `qtdeclarative` | QML / Qt Quick |
| `qtsvg` | SVG 图标 |
| `qtimageformats` | PNG、JPEG 等图片格式 |
| `qt5compat` | Qt5 兼容 API |
| `qtactiveqt` | Windows COM / ActiveX |
| `qttools` | moc、uic、rcc 等构建工具 |

### 已排除的浏览器相关模块

| 模块 | 说明 |
|------|------|
| `qtwebengine` | 嵌入式 Chromium 浏览器内核 |
| `qtwebview` | 依赖 WebEngine 的 WebView |
| `qtwebchannel` | 常与 WebEngine 配合使用 |

### 自定义说明

- 编译其他 QT 版本：修改 `qt_version`，并按该版本模块依赖调整 `-skip` 参数
- 不需要排除某模块：从参数中删除对应的 `-skip 模块名`
- 需要额外排除：追加 `-skip 模块名`
- 跳过 `qtshadertools` 时须同时 `-skip qtcanvaspainter`、`-skip qtmultimedia`（QT 6.11 推荐参数已包含）
- 跳过 `qtpositioning` 时须同时 `-skip qtlocation`（推荐参数已包含）

---

## 注意事项

- **编译参数为必填**，须包含 `-prefix` 等完整 configure 选项
- `-prefix` 须为 `C:/Qt/static-msvc2022`，与打包安装路径一致
- 完整静态编译通常需要 **2~6 小时**
- GitHub Release 单文件上限 **2 GB**；若上传失败，可从 Artifacts 下载（保留 14 天）
- Release 标签按版本自动生成：`qt-static-msvc-<版本>`；若该标签已存在则自动追加 `-2`、`-3`…（如 `qt-static-msvc-6.11.1-2`）
