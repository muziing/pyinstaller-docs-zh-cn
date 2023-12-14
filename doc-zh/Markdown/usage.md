# 使用 PyInstaller

> 翻译自 [PyInstaller 文档 v6.2.0 - Using PyInstaller](https://pyinstaller.org/en/v6.2.0/usage.html)

`pyinstaller` 命令的语法为：

> `pyinstaller` \[*options*\] *script* \[*script* ...\] \| *specfile*

在最简单的情况下，将当前目录设置为你的程序 `myscript.py` 的位置，然后执行：

```shell
pyinstaller myscript.py
```

PyInstaller 分析 `myscript.py` 并：

- 在与脚本相同的文件夹中写入 `myscript.spec`。
- 如果 `build` 文件夹不存在，则在与脚本相同的文件夹中创建该文件夹。
- 在 `build` 文件夹中写入一些日志文件和工作文件。
- 如果不存在 `dist` 文件夹，则在与脚本相同的文件夹中创建该文件夹。
- 在 `dist` 文件夹中写入 `myscript` 可执行文件夹。

在 `dist` 文件夹中，你可以找到向用户分发的捆绑应用程序。

通常情况下，你只需在命令行中指定一个脚本。如果指定了更多脚本，则所有脚本都会被分析并包含在输出中。不过，指定的第一个脚本将提供 spec 文件和可执行文件夹或文件的名称。它的代码将在运行时首先执行。

对于特定使用，你可以编辑 `myscript.spec` 的内容（参见[*使用 Spec 文件*](./spec-files.md#使用-spec-文件)）。这样做之后，你可以向 PyInstaller 指定 spec 文件替代脚本：

> `pyinstaller myscript.spec`

`myscript.spec` 文件包含以脚本文件为参数运行 `pyinstaller`（或 `pyi-makespec`）时指定的选项所提供的大部分信息。使用 spec 文件运行 `pyinstaller` 时，通常不需要指定任何选项。从 spec 文件构建时，只有[少数命令行选项](./spec-files.md#使用-spec-文件)会产生影响。

你可以给出脚本或 spec 文件的路径，例如

> `pyinstaller` *options...*`~/myproject/source/myscript.py`

或者，在 Windows 上，

> `pyinstaller "C:\Documents and Settings\project\myscript.spec"`

## 命令选项

`pyinstaller` 命令选项的完整列表如下：

### 位置参数

`scriptname`
    要处理的脚本文件名或一个 `.spec` 文件名。如果指定了 `.spec` 文件，则大部分选项都是非必须的，会被忽略。

### 选项

`-h, --help`
    显示此帮助信息并退出。

`-v, --version`
    显示程序版本信息并退出。

`--distpath DIR`
    捆绑应用程序的放置位置（默认值：`./dist`）。

`--workpath WORKPATH`
    放置所有临时工作文件、`.log`、`.pyz` 等的位置（默认值：`./build`）。

`-y, --noconfirm`
    覆盖输出目录中的原有内容（默认值：`SPECPATH/dist/SPECNAME`），不请求确认。

`--upx-dir UPX_DIR`
    UPX 组件的位置（默认值：搜索可执行文件路径，即环境变量中的 `PATH`）。

`--clean`
    在构建之前，清理 PyInstaller 缓存并删除临时文件。

`--log-level LEVEL`
    编译时控制台信息的详细程度。LEVEL 可以是 `TRACE`、`DEBUG`、`INFO`、`WARN`、`DEPRECATION`、`ERROR`、`FATAL` 之一（默认值：`INFO`）。也可以通过 `PYI_LOG_LEVEL` 环境变量进行覆盖设置。

### 生成什么

`-D, --onedir`
    创建包含一个可执行文件的单文件夹捆绑包（默认值）。

`-F, --onefile`
    创建单文件捆绑的可执行文件。

`--specpath DIR`
    存储生成的 spec 文件的文件夹（默认值：当前目录）。

`-n NAME, --name NAME`
    为捆绑的应用程序和 spec 文件指定的名称（默认值：第一个脚本的名称）。

`--contents-directory CONTENTS_DIRECTORY`
    仅适用于单文件夹构建。指定存放所有支持文件（即除可执行文件本身外的所有文件）的目录名称。使用 `.` 来重新启用旧的 onedir 布局，但不包含内容目录。

### 捆绑什么，在何处搜索

`--add-data SOURCE:DEST`
    要添加到应用程序中的附加数据文件或包含数据文件的目录。参数值的形式应为 "source:dest_dir"，其中 source 是要收集的文件（或目录）的路径，dest_dir 是相对于应用程序顶层目录的目标目录，两个路径之间用冒号（`:`）分隔。要将文件放入应用程序顶层目录，使用 `.` 作为 dest_dir。该选项可多次使用。

`--add-binary SOURCE:DEST`
    要添加到可执行文件中的其他二进制文件。格式参考 `--add-data` 选项。该选项可多次使用。

`-p DIR, --paths DIR`
    搜索导入的路径（如使用 PYTHONPATH）。允许使用多个路径，以 `:` 分隔，或多次使用该选项。相当于在 spec 文件中提供 pathex 参数。

`--hidden-import MODULENAME, --hiddenimport MODULENAME`
    指明脚本中不可见的导入关系。该选项可多次使用。

`--collect-submodules MODULENAME`
    收集指定软件包或模块的所有子模块。该选项可多次使用。

`--collect-data MODULENAME, --collect-datas MODULENAME`
    收集指定软件包或模块的所有数据。该选项可多次使用。

`--collect-binaries MODULENAME`
    收集指定软件包或模块的所有二进制文件。该选项可多次使用。

`--collect-all MODULENAME`
    收集指定软件包或模块的所有子模块、数据文件和二进制文件。该选项可多次使用。

`--copy-metadata PACKAGENAME`
    复制指定包的元数据。该选项可多次使用。

`--recursive-copy-metadata PACKAGENAME`
    复制指定包及其所有依赖项的元数据。该选项可多次使用。

`--additional-hooks-dir HOOKSPATH`
    用于搜索钩子的附加路径。该选项可多次使用。

`--runtime-hook RUNTIME_HOOKS`
    自定义运行时钩子文件的路径。运行时钩子是与可执行文件捆绑在一起的代码，在其他代码或模块之前执行，以设置运行时环境的特殊功能。该选项可多次使用。

`--exclude-module EXCLUDES`
    将被忽略（就像没有被找到一样）的可选模块或软件包（Python 名称，不是路径名）。该选项可多次使用。

`--splash IMAGE_FILE`
    （实验性功能）为应用程序添加一个带有 IMAGE_FILE 图像的闪屏。闪屏可以在解压缩时显示进度更新。

### 如何生成

`-d {all,imports,bootloader,noarchive}, --debug {all,imports,bootloader,noarchive}`
    辅助调试冻结应用程序。该参数可以提供多次以选择下面多个选项。- `all`：以下所有三个选项。- `imports`：向底层 Python 解释器指定 `-v` 选项，使其在每次初始化模块时打印一条信息，显示模块的加载位置（文件名或内置模块）。参考[命令行选项 -v](https://docs.python.org/zh-cn/3/using/cmdline.html#cmdoption-1)。- `bootloader`：告知 bootloader 在初始化和启动捆绑应用程序时发布进度消息，用于诊断导入丢失的问题。- `noarchive`：不将所有冻结的 Python 源文件作为压缩归档存储在生成的可执行文件中，而是将它们作为文件存储在生成的输出目录中。

`--python-option PYTHON_OPTION`
    指定运行时传递给 Python 解释器的命令行选项。目前支持 “v”（相当于 “–debug imports”）、“u”、“W \<warning control>”、“X \<xoption>” 与 “hash_seed=\<value>”。详情参阅[*指定 Python 解释器选项*](./spec-files.md#指定-python-解释器选项)。

`-s, --strip`
    对可执行文件和共享库应用符号条带表（symbol-table strip）。不建议在 Windows 环境下使用。

`--noupx`
    即使有可用的 UPX 也不要使用。在 Windows 和 \*nix 下效果有所不同。

`--upx-exclude FILE`
    防止二进制文件在使用 upx 时被压缩。如果 upx 在压缩过程中破坏了某些二进制文件，通常可以使用此功能。FILE 是二进制文件不含路径的文件名。该选项可多次使用。

### Windows 与 macOS 专用选项

`-c, --console, --nowindowed`
    为标准 i/o 打开一个控制台窗口（默认选项）。在 Windows 中，如果第一个脚本是 ‘.pyw’ 文件，则此选项无效。

`-w, --windowed, --noconsole`
    不提供用于标准 i/o 的控制台窗口。在 macOS 上，这也会触发构建一个 .app 捆绑程序。在 Windows 系统中，如果第一个脚本是 ‘.pyw’ 文件，则会自动设置该选项。在 \*NIX 系统上，该选项将被忽略。

`-i <FILE.ico or FILE.exe,ID or FILE.icns or Image or "NONE">, --icon <FILE.ico or FILE.exe,ID or FILE.icns or Image or "NONE">`
    FILE.ico：将图标应用于 Windows 可执行文件。FILE.exe,ID：从一个 exe 文件中提取带有 ID 的图标。FILE.icns：将图标应用到 macOS 的 .app 捆绑程序中。如果输入的图像文件不是对应平台的格式（Windows 为 ico，Mac 为 icns），PyInstaller 会尝试使用 Pillow 将图标翻译成正确的格式（如果安装了 Pillow）。使用 “NONE” 不应用任何图标，从而使操作系统显示默认图标（默认值：使用 PyInstaller 的图标）。该选项可多次使用。

`--disable-windowed-traceback`
    禁用窗口（noconsole）模式下未处理异常的回溯转储，并显示禁用此功能的信息。

### Windows 专用选项

`--version-file FILE`
    将 FILE 中的版本资源添加到 exe 中。

`-m <FILE or XML>, --manifest <FILE or XML>`
    将 manifest FILE 或 XML 添加到 exe 中。

`-r RESOURCE, --resource RESOURCE`
    为 Windows 可执行文件添加或更新资源。RESOURCE 包含一到四个条目：FILE[,TYPE[,NAME[,LANGUAGE]]]。FILE 可以是数据文件或 exe/dll。对于数据文件，则必须指定 TYPE 和 NAME。LANGUAGE 默认为 0，也可以指定为通配符 `*`，以更新给定 TYPE 和 NAME 的所有资源。对于 exe/dll 文件，如果省略 TYPE、NAME 和 LANGUAGE 或将其指定为通配符 `*`，则 FILE 中的所有资源都将添加/更新到最终可执行文件中。该选项可多次使用。

`--uac-admin`
    使用该选项可创建一个 Manifest，在应用程序启动时请求提升权限。

`--uac-uiaccess`
    使用此选项，可让提升后的应用程序与远程桌面协同工作。

`--hide-console {hide-late,minimize-late,hide-early,minimize-early}`
    在启用控制台的可执行文件中，如果程序有控制台窗口（即，不是从一个现有的控制台窗口启动的），bootloader 会自动隐藏或最小化控制台窗口。

### macOS 专用选项

`--argv-emulation`
    启用 macOS 应用程序捆绑包的 argv 仿真。如果启用，初始打开文档/URL 事件将由 bootloader 处理，并将传递的文件路径或 URL 附加到 sys.argv。

`--osx-bundle-identifier BUNDLE_IDENTIFIER`
    macOS `.app` 捆绑标识符用于代码签名的唯一程序名称。通常的形式是以反向 DNS 记法表示的分层名称。例如：com.mycompany.department.appname。（默认值：第一个脚本的名称）

`--target-architecture ARCH, --target-arch ARCH`
    目标架构。有效值：`x86_64`、`arm64`、`universal2`。启用冻结应用程序在 universal2 和 single-arch version 之间的切换（前提是 Python 安装支持目标架构）。如果为指定目标架构，则以当前运行的架构为目标。

`--codesign-identity IDENTITY`
    代码签名身份。使用提供的身份对收集的二进制文件和生成的可执行文件进行签名。如果未提供签名标识，则会执行临时签名。

`--osx-entitlements-file FILENAME`
    在对收集的二进制文件进行代码签名时使用的权限文件。

### 罕用的特殊选项

`--runtime-tmpdir PATH`
    在单文件模式下提取库和支持文件的位置。如果给定此选项，bootloader 将忽略运行时操作系统定义的任何临时文件夹位置。将在此处创建 `_MEIxxxxxx` 文件夹。请仅在你知道自己在做什么的情况下使用该选项。

`--bootloader-ignore-signals`
    告知 bootloader 忽略信号，而不是将其转发给子进程。例如，在监督进程同时向 bootloader 和子进程发出信号（如，通过进程组）以避免向子进程发出两次信号的情况下就很有用。

## 缩短命令

由于选项众多，一条完整的 `pyinstaller` 命令可能会变得非常长。在开发脚本的过程中，你会一遍又一遍地运行相同的命令。可以将该命令放在 shell 脚本或批处理文件中，使用续行符使其可读。例如，在 GNU/Linux 中：

```shell
pyinstaller --noconfirm --log-level=WARN \
    --onefile --nowindow \
    --add-data="README:." \
    --add-data="image1.png:img" \
    --add-binary="libfoo.so:lib" \
    --hidden-import=secret1 \
    --hidden-import=secret2 \
    --upx-dir=/usr/local/share/ \
    myscript.spec
```

或者在 Windows 中，使用鲜为人知的 BAT 文件续行符（PowerShell 的续行符则是 `）：

```bat
pyinstaller --noconfirm --log-level=WARN ^
    --onefile --nowindow ^
    --add-data="README:." ^
    --add-data="image1.png:img" ^
    --add-binary="libfoo.so:lib" ^
    --hidden-import=secret1 ^
    --hidden-import=secret2 ^
    --icon=..\MLNMFLCN.ICO ^
    myscript.spec
```

## 从 Python 代码中运行 PyInstaller

如果想从 Python 代码中运行 PyInstaller，可以使用 `PyInstaller.__main__` 中定义的 `run` 函数。例如，下面的代码：

```python
import PyInstaller.__main__

PyInstaller.__main__.run([
    'my_script.py',
    '--onefile',
    '--windowed'
])
```

等同于：

```shell
pyinstaller my_script.py --onefile --windowed
```

## 使用 UPX

[UPX](https://upx.github.io/) 是一款用于压缩可执行文件和库的免费工具。它适用于大多数操作系统，可用压缩大量可执行文件格式。关于下载和支持的文件格式列表，请参阅 [UPX](https://upx.github.io/) 主页。

当 UPX 可用时，PyInstaller 会用它来单独压缩每个收集的二进制文件（可执行文件、共享库或 Python 扩展）以减小冻结应用程序（单文件夹捆绑的目录或单文件可执行文件）的整体大小。冻结应用程序的可执行文件本身并没有经过 UPX 压缩（不管是单文件夹模式还是单文件模式），因为其文件大小中的大部分都已经是由包含单独压缩文件的嵌入式压缩包构成。

PyInstaller 会在标准可执行路径（由 `PATH` 环境变量定义）或通过 `--upx-dir` 命令行选项指定的路径中查找 UPX。如果找到，就会自动使用。使用 `--noupx` 命令行选项可用完全禁用 UPX。

> Note
>
> UPX 目前仅用于 Windows 系统。在其他操作系统上，即使找到 UPX，也无法处理收集到的二进制文件。现代 Linux 发行版上的共享库（比如 Python 共享库）在使用 UPX 处理时似乎会崩溃，导致应用程序捆绑包失效。在 macOS 上，UPX 目前无法处理 .dylib 共享库；此外，UPX 压缩文件无法通过 `codesign` 组件的验证检查，因此无法进行代码签名（这是苹果 M1 平台的一项要求）。

### 从 UPX 处理中排除有问题的文件

使用 UPX 可能会导致已收集的共享库损坏。这种损坏的已知例子包括启用了 [Control Flow Guard (CFG)](https://github.com/upx/upx/issues/398) 的 Windows DLL，以及 [Qt5 和 Qt6 插件](https://github.com/upx/upx/issues/107)。在这种情况下，可能需要使用 `--upx-exclude` 选项（或在 [.spec 文件](./spec-files.md#使用-spec-文件)中使用 `upx_exclude` 参数）将个别文件排除在 UPX 处理之外。

*在版本 4.2 中修改：* PyInstaller 可检测启用 CFG 的 DLL，并自动将其排除在 UPX 处理之外。

*在版本 4.3 中修改：* PyInstaller 会自动将 Qt5 和 Qt6 插件排除在 UPX 处理之外。

尽管 PyInstaller 尝试自动检测并从 UPX 处理中排除一些有问题的文件，但在某些情况下，需要手动指定 UPX 排除项。例如，`PySide2` 软件包中的 32 位 Windows 二进制文件（Qt5 DLL 和 Python 扩展模块）据[报告](https://github.com/pyinstaller/pyinstaller/issues/4178#issuecomment-868985789)称会被 UPX 破坏。

*在版本 5.0 中修改：* 早期版本将提供的 UPX 排除项的名称与收集的二进制文件的名称进行比较（由于大小写规范化不完善，在 Windows 系统上要求提供的排除项名称必须小写）。现在的 UPX 排除模式匹配使用操作系统默认的大小写敏感性，并支持通配符（`*`）。它还支持指定文件的（全部或部分）父路径。

所提供的 UPX 排除模式与所收集二进制文件的*源*（原始）路径相匹配，匹配从右向左进行。

例如，要从 PySide2 包中排除 Qt5 DLL 文件，使用 `--upx-exclude "Qt*.dll"`；要从 PySide2 包中排除 Python 扩展，使用 `--upx-exclude "PySide2\*.pyd"`。

## 闪屏（*实验性功能*）

> Note
>
> 该功能与 macOS 不兼容。在当前的设计中，闪屏在二级线程中运行，而 macOS 上的 Tcl/Tk（或者说底层 GUI 工具包）不允许这样做。

有些应用程序可能需要在程序（bootloader）启动后立即显示闪屏（splash screen）。因为，特别在是单文件模式下，大型应用程序的提取/启动时间可能会很长，bootloader 在进行准备工作，但用户无法判断应用程序是否已经成功启动。

Bootloader 能够显示单幅（只有一幅图像的）闪屏，该闪屏在实际主提取程序启动前显示。闪屏支持非透明和硬切透明图像作为背景图像，因此也可以显示非矩形闪屏。

该闪屏基于 [Tcl/Tk](http://www.tcl.tk/)，与 Python 模块 [tkinter](http://wiki.python.org/moin/TkInter) 使用的库相同。PyInstaller 在编译时将 tcl 和 tk 的动态库捆绑到应用程序中。这些动态库在解压缩后（如果程序已被打包成单文件压缩包），会在应用程序启动时加载到 bootloader 中。由于必要的动态链接库的文件大小非常小，应用程序启动和闪屏之间几乎没有延迟。闪屏所需文件的压缩大小约为 *1.5MB*。

作为一种附加功能，还可以选择在闪屏上显示文本。这可以在 Python 中更改/更新。这为在 Python 程序较长的启动过程中（如等待网络响应或将大文件加载到内存中）显示闪屏提供了可能。你还可以在闪屏后面启动 GUI，只有在图形界面完全初始化后才关闭闪屏。可以选择设置文字的字体、颜色和大小。不过，因为字体没有捆绑，所以必须预先已经安装在用户系统上。如果字体不可用，则使用后备字体。

如果将闪屏配置为显示文本，它将自动（作为单文件压缩包时）显示当前正在解压缩的文件名与进度条。

## pyi_splash 模块

闪屏由 Python 中的 `pyi_splash` 模块控制，该模块可以在运行时导入。该模块并**不能**通过软件包管理器安装，因为它是 PyInstaller 的一部分，在需要时会被包含进来。该模块必须在 Python 程序中导入。用法如下：

```python
import pyi_splash

# 更新闪屏上的文字
pyi_splash.update_text("PyInstaller is a great software!")
pyi_splash.update_text("Second time's a charm!")

# 关闭闪屏。调用该函数的时间并不重要，在调用
# 该函数或 Python 程序结束之前，闪屏一直是打开的
pyi_splash.close()
```

当然，导入应放在一个 `try ... except` 块中，以防程序作为普通 Python 脚本被外部使用，而没有 bootloader。详细说明参见 [pyi_splash Module (Detailed)](https://pyinstaller.org/en/v6.2.0/advanced-topics.html#pyi-splash-module)。

## 定义提取位置

在极少数情况下，当你捆绑至单个可执行文件（参见[*捆绑至单个文件*](./operating-mode.md#捆绑至单个文件) 与[*单文件程序如何工作*](./operating-mode.md#单文件程序如何工作)）时，你可能希望在编译时控制临时目录的位置。这可以使用 `--runtime-tmpdir` 选项来实现。如果给定了该选项，bootloader 将忽略运行时操作系统定义的任何临时文件夹位置。请仅在你知道自己在做什么的情况下使用该选项。

## 多平台支持

如果你只为一种操作系统和 Python 的组合发布应用程序，则只需像安装其他软件包一样安装 PyInstaller，并在正常开发设置中使用即可。

### 支持多种 Python 环境

当你需要在一个操作系统中为不同版本的 Python 和支持库（例如，一个 Python 3.6 版本和一个 Python 3.7 版本；或一个使用 Qt4 的支持版本和一个使用 Qt5的开发版本）捆绑应用程序时，我们建议你使用 [venv](https://docs.python.org/zh-cn/3/library/venv.html)。使用 *venv*，可以维护 Python 和已安装软件包的不同组合，并轻松地从一种组合切换到另一种组合。这些组合被称之为虚拟环境（*virtual environments*）。

- 使用 *venv* 可以根据需要创建多个不同的开发环境，每个环境都有自己独特的 Python 和已安装软件包的组合。
- 在每个虚拟环境中安装 PyInstaller。
- 在每个虚拟环境中使用 PyInstaller 来构建你的应用程序。

注意，当使用 *venv* 时，PyInstaller 命令的路径为：

- Windows： ENV_ROOT\\Scripts
- 其他平台：ENV_ROOT/bin

或者也可以通过[激活虚拟环境](https://docs.python.org/zh-cn/3/tutorial/venv.html#creating-virtual-environments)来简化命令路径。

参阅 [PEP 405](https://www.python.org/dev/peps/pep-0405) 和[关于虚拟环境和包的 Python 官方教程](https://docs.python.org/zh-cn/3/tutorial/venv.html)来了解更多信息。

### 支持多种操作系统

如果你需要在多个操作系统（例如 Windows 和 macOS）上发布应用程序，你必须在每个平台上安装 PyInstaller，并在每个平台上分别捆绑你的应用程序。

你可以使用虚拟化技术在一台机器上完成这项工作。免费的 [VirtualBox](https://www.virtualbox.org/) 或付费的 [VMWare](https://www.vmware.com/products/workstation-player.html) 和 [Parallels](http://www.parallels.com/) 允许你以"客户"身份运行另一个完整的操作系统。你可以为每个"客户"操作系统设置一个虚拟机，在其中安装 Python、应用程序所需的支持包和 PyInstaller。

像 [NextCloud](https://nextcloud.org/) 或[坚果云](https://www.jianguoyun.com/)这样的[文件同步与共享](https://en.wikipedia.org/wiki/Enterprise_file_synchronization_and_sharing)系统对虚拟机很有用。在每台虚拟机上安装同步客户端，所有虚拟机都链接到你的同步账户。在同步文件夹中保存一份脚本副本。这样，你就可以在任何虚拟机上运行 PyInstaller 了：

```shell
cd ~/NextCloud/project_folder/src # GNU/Linux, Mac -- Windows 类似
rm *.pyc # 删除由另一个 Python 编译的模块
pyinstaller --workpath=path-to-local-temp-folder  \
            --distpath=path-to-local-dist-folder  \
            ...other options as required...       \
            ./myscript.py
```

PyInstaller 从公共同步文件夹中读取脚本，但会将其工作文件夹和捆绑的应用程序写入虚拟机本地的文件夹中。

如果你在多个平台（例如 GNU/Linux 和 macOS）上共享同一个家目录（home），则需要在每个平台上将 PYINSTALLER_CONFIG_DIR 环境变量设置为不同的值，否则 PyInstaller 可能会缓存一个平台的文件，并在另一个平台上使用它们，因为默认情况下，它使用你的家目录的子目录作为缓存位置。

据说可以使用免费的 [Wine](http://www.winehq.org/) 环境在 GNU/Linux 下为 Windows 进行交叉开发。需要更多细节详情，参阅[How to Contribute](https://pyinstaller.org/en/v6.2.0/contributing.html)。

## 捕获 Windows 版本数据

Windows 应用程序可能需要包含版本资源文件（Version resource file）。版本资源文件包含一组数据结构，其中一些包含二进制整数，一些包含字符串，用于描述可执行文件的属性。更多详细信息，参阅 Microsoft [Version Information Structures](https://learn.microsoft.com/zh-cn/windows/win32/menurc/version-information-structures) 页面。

版本资源非常复杂，有些元素是可选的，有些则是必需的。当你查看属性对话框的版本选项卡时，显示的数据与资源的结构之间没有简单的关系。因此，PyInstaller 包含了 `pyi-grab_version` 命令。调用该命令时，需要输入任何具有版本资源的 Windows 可执行文件的完整路径名：

> `pyi-grab_version` *executable_with_version_resource*

该命令将以可读形式，把表示版本资源的文本写入标准输出。你可以将其从控制台窗口复制，或重定向到文件。然后你可以编辑版本信息，使其适应你的程序。

版本文本文件使用 UTF-8 编码，可能包含非 ASCII 字符。（版本资源文件的字符串字段允许使用 Unicode 字符。）除非你确定文本文件只包含 ASCII 字符串值，否则一定要以 UTF-8 编码编辑和保存文本文件。

编辑后的版本文本文件可通过 `--version-file` 选项提供给 `pyinstaller` 或 `pyi-makespec`。文本数据将被转换成版本资源并安装到捆绑的应用程序中。

版本资源中由两个 64 位二进制值，`FileVersion` 与 `ProductVersion`。在版本文本文件中，这些值以四元素元组的形式给出，例如：

```python
filevers=(2, 0, 4, 0),
prodvers=(2, 0, 4, 0),
```

每个元组的元素代表从高位到低位的 16 位数值。例如，值 `(2, 0, 4, 0)` 在十六进制中解析为 `0002000000040000`。

你也可以在创建捆绑应用程序后，使用 `pyi-set_version` 命令从文本文件中安装版本资源：

> `pyi-set_version` *version_text_file* *executable_file*

`pyi-set_version` 组件读取由 `pyi-grab_version` 写入的版本文本文件，将其转换为版本资源，并将该资源安装到指定的 *executable_file* 中。

对于高级用途，请检查由 `pyi-grab_version` 编写的版本文本文件。你会发现它是创建一个 `VSVersionInfo` 对象的 Python 代码。`VSVersionInfo` 类的定义可以在 PyInstaller 发行文件夹中的 `utils/win32/versioninfo.py` 中找到。你可以编写一个导入 `versioninfo` 的程序。在该程序中，可以对版本信息文本文件的内容使用 `eval` 来生成一个 `VSVersionInfo` 对象。你可以使用该对象的 `.toRaw()` 方法生成二进制形式的版本资源。或者对该对象使用 `unicode()` 函数来重新生成版本文本文件。

## 构建 macOS 应用程序捆绑包

在 macOS 下，PyInstaller 总是在 `dist` 中构建一个 UNIX 可执行文件。如果指定 `--onedir`，输出将是名为 `myscript` 的文件夹，其中包含支持文件和名为 `myscript` 的可执行文件。如果指定 `--onefile`，输出将是一个名为 `myscript` 的 UNIX 可执行文件。这两个可执行文件都可以通过终端命令行启动。标准输入和输出通过终端窗口正常工作。

如果你在这二者中的任一模式中指定了 `--windowed`，`dist` 文件夹中还会包含一个名为 `myscript.app` 的 macOS 应用程序。

你可能知道，应用程序是一种特殊类型的文件夹。PyInstaller 构建的应用程序包含一个始终名为 `Contents` 的文件夹，其中包含：

- 一个空的 `Frameworks` 文件夹。
- 包含图标文件的文件夹 `Resources`。
- 描述应用程序的文件 `Info.plist`。
- 一个 `MacOS` 文件夹，包含可执行文件和支持文件，与 `--onedir` 文件夹一样。

使用 `--icon` 参数来为应用程序指定一个自定义图标。它将被复制到 `Resources` 文件夹中。（如果没有指定图标文件，PyInstaller 会提供一个带有 PyInstaller logo 的文件 `icon-windowed.icns`）。

使用 `--osx-bundle-identifier` 参数来添加捆绑标识符。这将成为代码签名中使用的 `CFBundleIdentifier`（详见 [PyInstaller code signing recipe](https://github.com/pyinstaller/pyinstaller/wiki/Recipe-OSX-Code-Signing) 和  [Apple code signing overview](https://developer.apple.com/library/mac/technotes/tn2206/_index.html) 技术说明）。

你可以通过编辑 spec 文件向 `Info.plist` 添加其他项目；参阅[*用于 macOS 捆绑的 spec 文件选项*](./spec-files.md#用于-macos-捆绑的-spec-文件选项)。

## 特定平台注意事项

### GNU/Linux

#### 让 GNU/Linux 应用程序向前兼容

在 GNU/Linux 下，PyInstaller 不会将 `libc` （C 标准库，通常是 `glibc`，Gnu 版本）与应用程序捆绑在一起。取而代之的是，应用程序希望从其运行的本地操作系统动态链接到 `libc`。任何应用程序与 `libc` 之间的接口都能向前兼容较新的版本，但不能向后兼容较旧的版本。

因此，如果在当前版本的 GNU/Linux 上捆绑应用程序，那么在旧版本的 GNU/Linux 上可能无法执行（通常会出现运行时动态链接错误）。

解决方法是始终在你打算支持的*最旧*版本的 GNU/Linux 上构建应用程序。在较新版本中，程序应能继续使用 `libc`。

GNU/Linux 标准库（比如 `glibc`）分为 64 位和 32 位两个版本，且二者并不兼容。因此，不能将应用程序捆绑到 32 位系统上，然后在 64 位系统上运行，反之亦然。你必须为支持的每种字长制作一个独特的应用程序版本。

注意，PyInstaller 会捆绑通过依赖关系分析找到的其他共享库，如 libstdc++.so.6、libfontconfig.so.1、libfreetype.so.6。如果系统中存在这些库的旧版本（因此不兼容），则可能需要这些库。另一方面，如果试图加载系统提供的共享库，而该共享库又与系统提供的较新版本的库链接，捆绑的库可能会导致问题。

例如，系统安装的 mesa DRI 驱动（如，radeonsi_dri.so）依赖于系统提供的 libstdc++.so.6 版本。如果冻结的应用程序捆绑了旧版本的 libstdc++.so.6（从构建系统中收集），很可能会导致符号缺失错误，并阻止 DRI 驱动加载。在这种情况下，应删除捆绑的 libstdc++.so.6。不过，在当前系统提供的 libstdc++.so.6 版本旧于构建系统提供的版本时，这可能不起作用；在这种情况下，应保留捆绑的版本，因为系统提供的版本可能缺少其他依赖于 libstdc++.so.6 的已收集二进制文件所需的符号。

### Windows

开发者需要特别注意包含 Visual C++ 运行时 .dll：Python 3.5+ 使用 Visual Studio 2015 运行时，它已更名为 [“Universal CRT“](https://devblogs.microsoft.com/cppblog/introducing-the-universal-crt/)，并成为 Windows 10 的一部分。从 Windows Vista 到 Windows 8.1，都有 Windows 更新包，目标系统可能已安装，也可能未安装。因此，你有以下选择：

1. 在 *Windows 7* 上构建，据报告可以正常运行。
2. 在应用程序的安装程序中包含一个 VCRedist 包（可重新发布的软件包文件）。这是Microsoft 推荐的方法，参阅上述链接中的 “Distributing Software that uses the Universal CRT“，第 2 和第 3 条。
3. 安装 [Windows 软件开发工具包（SDK）](https://developer.microsoft.com/zh-cn/windows/downloads/windows-sdk/)，并扩展 .spec 文件以包含所需的 DLL，参阅上述链接第 6 条中的 "Distributing Software that uses the Universal CRT"。

    如果你认为 PyInstaller 应该自己完成这项工作，请[帮助改进](https://pyinstaller.org/en/v6.2.0/contributing.html#how-to-contribute) PyInstaller。

### macOS

#### 让 macOS 应用程序向前兼容

在 macOS 上，某一版操作系统的系统组件通常与后续版本兼容，但可能无法与早期版本兼容。虽然 PyInstaller 不会收集操作系统的系统组件，但收集的第三方二进制文件（如 Python 扩展模块）是根据特定版本的操作系统构建的，可能支持也可能不支持旧版本的操作系统。

因此，要确保冻结的应用程序支持旧版本操作系统，唯一的办法就是将其冻结在希望支持的最旧版本的操作系统上。这一点在使用 [Homebrew](http://brew.sh/) Python 构建时尤其适用，因为其二进制文件通常会明确针对正在运行的操作系统。

例如，为确保与 "Mojave" (10.14) 及更高版本的兼容性，应在 macOS 10.14 的副本中设置一个完整的环境（安装 Python、PyInstaller、你的应用程序代码及其所有依赖项），必要时可使用虚拟机。然后使用 PyInstaller 在该环境中冻结应用程序；生成的冻结应用程序应与该版本及后续版本的 macOS 兼容。

#### 在 macOS 上构建 32-bit 应用程序

> Note
>
> 由于 macOS 10.15 Catalina 取消了对 32 位应用程序的支持，本节内容已基本过时。关于现代版本 macOS 的64位多架构操作系统支持，请参阅[此处](https://pyinstaller.org/en/v6.2.0/feature-notes.html#macos-multi-arch-support)。不过，PyInstaller 然支持构建 32 位的 bootloader，而且 python.org 上仍提供（部分）Python 3.7 的 32 位/64 位安装包。PyInstaller 从 v6.0 其结束了对 Python 3.7 的支持。

旧版本的 macOS 同时支持 32 位和 64位可执行文件。PyInstaller 使用用于执行它的 Python 解释器的字长构建应用程序。这通常是一个 64 位版本的 Python，从而产生一个 64 位可执行文件。要创建 32 位可执行文件，在 32 位 Python 下运行 PyInstaller。

要验证已安装的 Python 版本是否支持在 64 位或 32 位模式下执行，请在 Python 可执行文件上使用 file 命令：

```shell
$ file /usr/local/bin/python3
/usr/local/bin/python3: Mach-O universal binary with 2 architectures
/usr/local/bin/python3 (for architecture i386):     Mach-O executable i386
/usr/local/bin/python3 (for architecture x86_64):   Mach-O 64-bit executable x86_64
```

操作系统会选择运行哪种架构，通常默认为 64 位。你可以使用 `arch` 命令按名称强行使用任一架构：

```shell
$ /usr/local/bin/python3
Python 3.7.6 (v3.7.6:43364a7ae0, Dec 18 2019, 14:12:53)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys; sys.maxsize
9223372036854775807

$ arch -i386 /usr/local/bin/python3
Python 3.7.6 (v3.7.6:43364a7ae0, Dec 18 2019, 14:12:53)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys; sys.maxsize
2147483647
```

> Note
>
> PyInstaller 不再为 macOS 提供预构建的 32 位 bootloader。为了在 32 位 Python 中使用 PyInstaller，你需要使用仍然支持编译 32 位的 XCode 版本，自己构建  bootloader。根据编译器/工具链的不同，你可能还需要向 `waf` 命令明确传递 `--target-arch=32bit`。

#### 获取已打开的文档名称

当用户双击已在你的应用程序中注册的文档类型时，或当用户拖拽文档并将其放置到你的应用程序图标上时，macOS 会启动你的应用程序，并以 OpenDocument AppleEvent 的形式提供已打开文档的名称。

这些事件通常通过应用程序中已安装的事件处理程序来处理（比如，通过 `ctypes` 使用 `Carbon`，或使用 `tkinter` `PyQt5` 这些 UI 工具包提供的功能）。

另外，PyInstaller 还支持将打开文档/URL 事件转换为参数，附加到 `sys.argv` 中。这只适用于在应用程序启动期间接收到的事件，即，在你的冻结代码启动之前。要处理应用程序已运行时派发的事件，你需要设置相应的事件处理程序。

更多细节，参阅[此节](https://pyinstaller.org/en/v6.2.0/feature-notes.html#macos-event-forwarding-and-argv-emulation)。

### AIX

根据 Python 是 32 位或 64 位可执行文件构建的，你可能需要设置或取消设置环境变量 `OBJECT_MODE`。要确定大小，可以使用以下命令：

```shell
$ python -c "import sys; print(sys.maxsize <= 2**32)"
True
```

当答案为 `True` （如上）时，Python 将作为 32 位可执行文件构建。

处理 32 位 Python 可执行文件时，请按以下步骤操作：

```shell
unset OBJECT_MODE
pyinstaller <your arguments>
```

处理 64 位 Python 可执行文件时，请按以下步骤操作：

```shell
export OBJECT_MODE=64
pyinstaller <your arguments>
```
