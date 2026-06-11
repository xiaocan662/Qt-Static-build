# Qt Static Build

使用 GitHub Actions 在云端编译 **Qt 6 静态库**（MSVC 2022 / x64），支持手动触发、自动下载源码、打包发布。

## 使用方法

1. 推送本仓库到 GitHub
2. 打开 **Actions** → **Build Static Qt6 (MSVC 2022)** → **Run workflow**
3. 填写参数后运行，完成后在 **Releases** 或 **Artifacts** 下载 `.7z`

| 参数 | 说明 |
|------|------|
| `qt_version` | Qt 版本，如 `6.11.1`；留空自动使用最新版 |
| `skip_modules` | 要跳过的模块，逗号分隔；留空不跳过 |
| `parallel_jobs` | 并行编译线程数；留空使用 jom 默认 |
| `create_release` | 是否创建 GitHub Release |

安装前缀固定为 `C:\Qt\static-msvc2022`，解压后设置：

```cmake
set(CMAKE_PREFIX_PATH "C:/Qt/static-msvc2022")
```

---

## Qt 6.11.1 推荐模块参数（本地桌面应用）

适用于 **QWidget / QML 本地应用**，**不编译嵌入式浏览器内核**（Chromium / WebEngine），并跳过 IoT、地图、3D 等大型可选模块，以控制编译时间和产物体积。

触发工作流时，**`qt_version` 填 `6.11.1`**，`skip_modules` 填：

```text
qtwebengine,qtwebview,qtwebchannel,qt3d,qtcharts,qtcoap,qtconnectivity,qtdatavis3d,qtdoc,qtgraphs,qtgrpc,qthttpserver,qtlanguageserver,qtlocation,qtpositioning,qtquick3d,qtquick3dphysics,qtremoteobjects,qtscxml,qtsensors,qtserialbus,qtserialport,qtshadertools,qtcanvaspainter,qtmultimedia,qtspeech,qtvirtualkeyboard,qtwayland,qtwebsockets,qtlottie,qtmqtt,qtnetworkauth
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

### 跳过的浏览器相关模块

| 模块 | 说明 |
|------|------|
| `qtwebengine` | 嵌入式 Chromium 浏览器内核（体积大、编译极慢） |
| `qtwebview` | 依赖 WebEngine 的 WebView 封装 |
| `qtwebchannel` | 常与 WebEngine 配合的 JS↔C++ 通道 |

### Qt 6.11.1 依赖说明

跳过 `qtshadertools` 时，须同时跳过依赖它的模块，否则 configure 会失败：

| 已跳过 | 须一并跳过 |
|--------|------------|
| `qtshadertools` | `qtcanvaspainter`、`qtmultimedia` |
| `qtpositioning` | `qtlocation` |

因此上述推荐参数已包含这些依赖项。**跳过 `qtmultimedia` 后无法使用 `QMediaPlayer` 等音视频 API**；若需要多媒体，可从 `skip_modules` 中移除 `qtshadertools`、`qtcanvaspainter`、`qtmultimedia` 三项（编译时间会更长）。

---

## 注意事项

- 完整静态编译通常需要 **2~6 小时**
- GitHub Release 单文件上限 **2 GB**；若上传失败，可从 Artifacts 下载（保留 14 天）
- 模块名须与源码目录一致；工作流会自动忽略当前版本中不存在的模块
