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
| `parallel_jobs` | 并行编译线程数；留空由 `cmake --build` 自动并行 |
| `create_release` | 是否创建 GitHub Release（独立 job 执行，**失败不影响编译与 Artifacts**） |

解压安装包到本地目录（推荐 `C:\Qt\static-msvc2022`）后，在 CMake 中设置：

```cmake
set(CMAKE_PREFIX_PATH "C:/Qt/static-msvc2022")
```

> CI 在 Runner 的 **D 盘**（`D:\Qt\...`）编译安装；下载的 `.7z` 解压到本机任意路径即可，`CMAKE_PREFIX_PATH` 指向解压目录。

---

## 检查 Runner 磁盘空间

编译前可先查看 GitHub Runner 上 C/D 盘各有多少可用空间：

**Actions** → **Check Disk Space** → **Run workflow**

约 **30 秒** 完成，结果在运行日志和 **Summary** 里可看到各盘符总容量、已用、剩余及 `GITHUB_WORKSPACE` / `RUNNER_TEMP` 路径。

---

在改发布逻辑或排查 Release 失败时，可单独运行：

**Actions** → **Test Release Publish** → **Run workflow**

| 参数 | 说明 |
|------|------|
| `qt_version` | 模拟版本号，仅用于标签/标题 |
| `create_release` | 是否创建 **预发布** Release |
| `upload_artifact` | 是否上传 Workflow Artifact |

约 **1~3 分钟** 完成：生成占位目录 → 7z 压缩 → 上传 Artifact → 创建预发布 Release（标签形如 `test-qt-static-msvc-6.11.1-test-run42`）。

---

## QT 6.11.1 推荐编译参数（复制粘贴）

适用于 **QWidget / QML 本地桌面应用**，**不含嵌入式浏览器内核**，并排除 IoT、地图、3D、多媒体等大型可选模块。

> 说明：`-skip` 支持逗号分隔多个模块（见 `qtyil.md` configure 帮助），下列参数已按 **强依赖关系** 一次性补全，避免 configure 因缺依赖模块而失败。

**整段复制，粘贴到「编译参数」即可：**

```text
-prefix D:/Qt/static-msvc2022 -static -static-runtime -release -opensource -confirm-license -nomake examples -nomake tests -platform win32-msvc -cmake-generator "NMake Makefiles" -skip qt3d,qtcanvaspainter,qtcharts,qtcoap,qtconnectivity,qtdatavis3d,qtdoc,qtgraphs,qtgrpc,qthttpserver,qtlanguageserver,qtlocation,qtlottie,qtmqtt,qtmultimedia,qtnetworkauth,qtopcua,qtopenapi,qtpositioning,qtprotobuf,qtquick3d,qtquick3dphysics,qtquickeffectmaker,qtremoteobjects,qtscxml,qtsensors,qtserialbus,qtserialport,qtshadertools,qtspeech,qttasktree,qtvirtualkeyboard,qtwayland,qtwebchannel,qtwebengine,qtwebsockets,qtwebview,qtpdf
```

### 保留的主要模块（未出现在 `-skip` 中）

| 模块 | 用途 |
|------|------|
| `qtbase` | 核心：窗口、控件、网络、文件等 |
| `qtdeclarative` | QML / Qt Quick |
| `qtquicktimeline` | QML 时间线动画 |
| `qtsvg` | SVG 图标 |
| `qtimageformats` | PNG、JPEG 等图片格式 |
| `qt5compat` | Qt5 兼容 API |
| `qtactiveqt` | Windows COM / ActiveX |
| `qttools` | moc、uic、rcc 等构建工具 |

### 已排除模块及依赖说明

| 类别 | 模块 | 说明 |
|------|------|------|
| 浏览器 | `qtwebengine` | Chromium 内核，体积大、编译极慢 |
| 浏览器 | `qtwebview` | 依赖 WebEngine |
| 浏览器 | `qtwebchannel` | 常与 WebEngine 配合 |
| 着色器链 | `qtshadertools` | 着色器工具（跳过后须一并跳过下列 3 项） |
| 着色器链 | `qtcanvaspainter` | 强依赖 `qtshadertools` |
| 着色器链 | `qtmultimedia` | 强依赖 `qtshadertools`（6.11） |
| 着色器链 | `qtquickeffectmaker` | 强依赖 `qtshadertools` |
| 定位链 | `qtpositioning` | 定位服务 |
| 定位链 | `qtlocation` | 强依赖 `qtpositioning` |
| 3D 链 | `qtquick3d` | 3D 渲染 |
| 3D 链 | `qtquick3dphysics` | 依赖 `qtquick3d` |
| 3D 链 | `qt3d` | 3D 引擎 |
| 网络/IoT | `qtgrpc`,`qtprotobuf`,`qthttpserver`,`qtmqtt`,`qtcoap`,`qtnetworkauth` | gRPC / HTTP 服务 / IoT |
| 工业/设备 | `qtserialport`,`qtserialbus`,`qtopcua`,`qtsensors` | 串口 / 工业协议 |
| 其他可选 | `qtcharts`,`qtgraphs`,`qtdatavis3d`,`qtlottie`,`qtpdf`,`qtspeech`,`qtvirtualkeyboard`,`qttasktree`,`qtopenapi` 等 | 图表 / PDF / 语音 / 6.11 新模块 |

### 自定义说明

- **不需要排除某模块**：从 `-skip` 逗号列表中删除对应模块名
- **需要额外排除**：在 `-skip` 列表末尾追加 `,模块名`
- **需要多媒体**（`QMediaPlayer` 等）：从 `-skip` 中移除 `qtmultimedia,qtshadertools,qtcanvaspainter,qtquickeffectmaker`（编译时间显著增加）
- **编译其他 QT 版本**：修改 `qt_version`；若该版本无某模块，configure 会自动忽略

---

## 注意事项

- **编译参数为必填**，须包含 `-prefix` 等完整 configure 选项
- `-prefix` 须为 `D:/Qt/static-msvc2022`，与工作流 D 盘安装路径一致（与 CI `QT_PREFIX` 相同）
- 源码/构建目录也在 D 盘（`D:\Qt\src`、`D:\Qt\build`），避免占满 C 盘
- 完整静态编译通常需要 **2~6 小时**
- GitHub Release 单文件上限 **2 GB**；若 Release 失败，**build job 仍成功**，可从 Artifacts 下载（保留 14 天）
- Release 在独立 **publish** job 中创建（`continue-on-error`），发布失败不会导致整次编译作废
- Release 标签按版本自动生成：`qt-static-msvc-<版本>`；若该标签已存在则自动追加 `-2`、`-3`…
