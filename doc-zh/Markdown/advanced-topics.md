# 进阶话题

> 翻译自 [PyInstaller 文档 v6.3.0 - Advanced Topics](https://pyinstaller.org/en/v6.3.0/advanced-topics.html)

下面的讨论涉及 PyInstaller 内部方法的细节。在正常使用中，你应该不需要这种程度的细节。但如果你想研究 PyInstaller 代码，并可能按[*贡献指南*](./contributing.md)对其做出贡献，那会很有帮助。

## Bootstrap 流程详解

在捆绑脚本开始执行之前，必须进行许多步骤。这些步骤的摘要已在概述（[*单文件夹程序如何工作*](./operating-mode.md#单文件夹程序如何工作) 和[*单文件程序如何工作*](./operating-mode.md#单文件程序如何工作)）中给出。以下是更多细节，以帮助你了解 bootloader 做的事情以及如何找出问题。

### Bootloader

Bootloader 为运行 Python 代码做好一切准备。它启动 setup，然后在另一个进程中返回自身。这种使用两个进程的方法有很大的灵活性，除 Windows 下的单文件夹模式外，所有捆绑程序都使用这种方式。因此，如果你在系统任务管理器中看到捆绑应用程序有两个进程，不必感到惊讶。

执行 bootloader 时会发生什么：

1. 第一个进程：bootloader 启动。

    1. 如果使用了单文件模式，将捆绑文件提取到 `temppath/_MEIxxxxxx`。
    2. 修改各种环境变量：
         - GNU/Linux：如果已设置，则将 LD_LIBRARY_PATH 的原始值保存到 LD_LIBRARY_PATH_ORIG中，将我们的路径预置到 LD_LIBRARY_PATH。
         - AIX：相同做法，只是使用 LIBPATH 和 LIBPATH_ORIG。
         - macOS：取消设置 DYLD_LIBRARY_PATH。
    3. 设置为处理两个进程的信号。
    4. 运行子进程。
    5. 等待子进程结束。
    6. 如果是单文件模式，删除 `temppath/_MEIxxxxxx`。

2. 第二个进程：bootloader 本身作为子进程启动。

    1. 在 Windows 设置[激活上下文](https://learn.microsoft.com/zh-cn/windows/win32/sbscs/activation-contexts)。
    2. 加载 Python 动态链接库。动态库的名称嵌入在可执行文件中。
    3. 初始化 Python 解释器：设置 sys.path、sys.prefix 和 sys.executable。
    4. 运行 Python 代码。

运行 Python 代码需要几个步骤：

1. 运行 Python 初始化代码，为运行用户的主脚本做好一切准备。初始化代码只能使用 Python 内置模块，因为一般的导入机制还不可用。它设置了 Python 导入机制，使其只能从嵌入可执行文件的归档文件中加载模块。它还为 `sys` 内置模块添加了属性 `frozen` 和 `_MEIPASS`。
2. 执行任何运行时钩子：首先是用户指定的钩子，然后是标准钩子。
3. 安装 Python "egg" 文件。当模块是一个 zip 文件（.egg）的一部分时，它已被捆绑到 `./eggs` 目录中。安装意味着将 .egg 文件名追加到 `sys.path` 中。Python 会自动检测 `sys.path` 中的项目是 zip 文件还是目录。
4. 运行主脚本。

### 捆绑应用程序中的 Python 导入

PyInstaller 将编译后的 Python 代码（`.pyc` 文件）嵌入到可执行文件中。PyInstaller 将其代码注入正常的 Python 导入机制。Python 允许这种做法，[PEP 302](https://www.python.org/dev/peps/pep-0302) "New Import Hooks" 中描述了其支持。

PyInstaller 实现了 PEP 302 规范，用于导入内置模块，导入 "冻结" 模块（与应用程序捆绑的已编译 Python 代码）和 C 扩展。可以在 `./PyInstaller/loader/pyi_mod03_importers.py` 中阅读代码。

在运行时，PyInstaller PEP 302 钩子会被追加到变量 `sys.meta_path` 中。在尝试导入模块时，解释器会首先尝试 `sys.meta_path` 中的 PEP 302 钩子，然后再搜索 `sys.path`。因此，Python 解释器会从捆绑的可执行文件中嵌入的归档文件中加载导入的 Python 模块。

下面是捆绑应用程序中导入语句的解析顺序：

1. 是内置模块吗？在变量 `sys.builtin_module_names` 中存有一份内置模块列表。
2. 它是嵌入在可执行文件中的模块吗？那就从嵌入的归档中加载。
3. 是 C 扩展吗？应用程序将尝试查找名为 `package.subpackage.module.pyd` 或 `package.subpackage.module.so` 的文件。
4. 接下来检查 `sys.path` 中的路径。其中可能有带有 Python 模块或 `.egg` 文件名的附加位置。
5. 如果模块没有找到，则引发 `ImportError`。

### 闪屏启动

> Note
>
> 该功能与 macOS 不兼容。在当前的设计中，闪屏在一个二级线程中运行，而 macOS 上的 Tcl/Tk（或者说底层 GUI 工具包）不允许这样做。

如果应用程序捆绑了闪屏，那么 bootloader 的启动程序和线程模型就会稍微复杂一些。下面描述了捆绑闪屏时的操作顺序：

1. Bootloader  会检查它是否作为最外层应用程序运行（而不是 bootloader 启动的子进程）。

2. 如果捆绑了闪屏资源，则尝试提取（单文件模式）。提取路径位于 `temppath/_MEIxxxxxx/__splashx` 内。如果在单文件夹模式下，应用程序会认为资源是相对于可执行文件的。

3. 将 tcl 和 tk 共享库加载到 bootloader 中。

    - Windows: `tcl86t.dll`/`tk86t.dll`
    - Linux: `libtcl.so`/`libtk.so`

4. 通过修改/替换以下函数，为 [Tcl/Tk](http://www.tcl.tk/) 解释器准备一个最小环境：

    1. `::tclInit`：该命令用于查找 tcl 的标准库。我们替换该命令是为了强制 tcl 只加载/执行捆绑的模块。
    2. `::tcl_findLibrary`：Tk 使用该函数获取其所有组件的源。覆盖函数会设置所需的环境变量，并评估所请求的文件。
    3. `::exit`：对该函数进行了修改，以确保正确退出闪屏线程。
    4. `::source`：该命令执行一个传入的文件的内容。由于我们在最小环境中运行，因此我们会模拟执行未捆绑的文件，并执行捆绑的文件。

5. 启动 tcl 解释器，并执行由 PyInstaller 的构建目标 `Splash` 在构建时生成的闪屏脚本。该脚本会创建环境变量 `_PYIBoot_SPLASH`，Python 解释器也可以使用该变量。它还会初始化一个 tcp 服务端 socket，以接收来自 Python 的命令。

> Note
>
> tcl 解释器在单独的线程中启动。只有在 tcl 解释器执行完闪屏脚本后，负责提取/启动 Python 解释器的 bootloader 线程才会恢复。

## `pyi_splash` 模块（详细）

此模块连接 bootloader，向闪屏发送信息。

它将作为 bootloader 所提供函数（比如显示文本或关闭）的 RPC 接口。这使得用户的 Python 程序和与 bootloader 的通信如何实现无关，因为提供了一致的 API。

要连接 bootloader，它需要连接到一个本地的 tcp 服务端 socket，其端口通过环境变量 `_PYIBoot_SPLASH` 传递。Bootloader 通过 Python 模块 `_socket` 连接到该 socket。虽然这个 socket 是双向的，但模块被配置为只发送数据。由于请求环境变量所需的 os 模块在启动时不可用，所以模块在初始化前不会建立连接。

该模块不支持在显示闪屏时重新加载，即无法 reload（比如通过 `importlib.reload`），因为当模块实例的连接丢失时，闪屏会自动关闭。

### 函数

> Note
>
> 注意，如果 `_PYIBoot_SPLASH` 环境变量不存在，或者在连接过程中发生错误，模块将**不会**引发一个错误，而只是不进行初始化（即，`pyi_splash.is_alive` 将返回 `False`）。在向闪屏发送命令之前，应检查模块是否已正确初始化，否则将引发一个 `RuntimeError`。

[`is_alive()`](https://pyinstaller.org/en/v6.3.0/advanced-topics.html#pyi_splash.is_alive)  
&ensp;&ensp;&ensp;&ensp;表示该模块是否可用。  
&ensp;&ensp;&ensp;&ensp;如果模块未初始化或被关闭闪屏禁用，则返回 `False`。否则，模块应该可以使用。

[`update_text(msg)`](https://pyinstaller.org/en/v6.3.0/advanced-topics.html#pyi_splash.update_text)  
&ensp;&ensp;&ensp;&ensp;更新闪屏窗口上的文本。  
&ensp;&ensp;&ensp;&ensp;**参数：** **msg**(*str*) - 要显示的文本  
&ensp;&ensp;&ensp;&ensp;**引发：** [ConnectionError](https://docs.python.org/zh-cn/3/library/exceptions.html#ConnectionError) - 如果操作系统无法向 socket 写入；[RuntimeError](https://docs.python.org/zh-cn/3/library/exceptions.html#RuntimeError) - 如果模块未初始化；

[`close()`](https://pyinstaller.org/en/v6.3.0/advanced-topics.html#pyi_splash.close)  
&ensp;&ensp;&ensp;&ensp;关闭与 ipc tcp 服务 socket 的连接  
&ensp;&ensp;&ensp;&ensp;这将关闭闪屏并使该模块无法使用。调用此函数后，将无法再次打开与闪屏的连接，此模块的所有功能也将无法使用

## 目录（TOC）和 Tree 类

PyInstaller 以所谓的目录（Table of Contents，TOC）列表格式管理要收集的文件列表。这些列表包含三元素元组，封装了文件的目标名称、完整源路径及其类型信息。

作为管理 TOC 列表组件的一部分，PyInstaller 提供了一个 `Tree` 类，作为从给定目录的内容构建 TOC 列表的便捷方法。该工具类可以在 [.spec 文件](./spec-files.md#使用-spec-文件) 中使用，也可以在自定义钩子中使用。

### 目录（TOC）列表

`Analysis` 对象会生成多个 TOC 列表，提供待收集文件的相关信息。文件根据其类型或功能被归类到不同的列表中，例如：

- `Analysis.scripts`：程序脚本
- `Analysis.pure`：纯 Python 模块
- `Analysis.binaries`：二进制扩展模块和共享库
- `Analysis.datas`：数据文件

生成的 TOC 列表会传递给 [spec 文件](./spec-files.md#使用-spec-文件)中的各种构建目标，例如 `PYZ`、`EXE` 和 `COLLECT`。

每个 TOC 列表都包含三元素元组：

```python
(dest_name, src_name , typecode)
```

其中，`dest_name` 是目标文件名（即冻结应用程序中的文件名，因此必须始终是相对名称），`src_name` 是源文件名（收集文件的路径），`typecode` 是表示文件（或条目）类型的字符串。

在内部，PyInstaller 使用了许多 *typecode* 值，但在一般情况下，你只需要知道这些：

| **typecode** | **描述**                        | **dest_name**                      | **src_name**                 |
| ------------ | ------------------------------- | ---------------------------------- | ---------------------------- |
| 'DATA'       | 任意（数据）文件。              | 冻结应用程序中的名称。             | 文件在构建系统中的完整路径。 |
| 'BINARY'     | 共享库。                        | 冻结应用程序中的名称。             | 文件在构建系统中的完整路径。 |
| 'EXTENSION'  | Python 二进制扩展。             | 冻结应用程序中的名称。             | 文件在构建系统中的完整路径。 |
| 'OPTION'     | PyInstaller/Python 运行时选项。 | 选项名称（和选项值，用空格分隔）。 | 忽略。                       |

目标名称与冻结应用程序中的最终名称相对应，相对于应用程序的顶层目录。它可能包含路径元素，例如 `extras/mydata.txt`。

`BINARY` 和 `EXTENSION` 类型的实例被假定为代表包含可加载可执行代码（如动态链接库）的文件。通常，`EXTENSION` 用于表示 Python 扩展模块，例如由 [Cython](http://www.cython.org/) 编译的模块。这两种文件类型的处理方式是相同的；PyInstaller 会扫描它们以查找额外的链接时依赖关系，并收集发现的任何依赖关系。在某些操作系统上，二进制文件和扩展会经过额外的处理（例如在 macOS 上对链接时依赖的路径重写和代码签名）。

在将 `Analysis` 生成的 TOC 列表传递给构建目标之前，可以在 [spec 文件](./spec-files.md#使用-spec-文件)中对其进行修改，以加入更多条目（不过最好是通过 *binaries* 或 *Analysis* 的 *datas* 参数传递额外的文件）或删除不需要的条目。

*在版本 5.11 中修改：* PyInstaller 5.11 之前的版本中，TOC 列表实际上是 `TOC` 类的实例，它在内部执行隐式条目去重复；也就是说，尝试插入具有已存在目标名称的条目不会导致列表发生任何变化。然而，由于 TOC 类定义松散、语义冲突等缺点，`TOC` 类已被弃用。TOC 列表现在是普通列表的实例，PyInstaller 会执行显式列表规范化（条目去重复）。显式规范化在 `Analysis` 实例化结束时执行，此时列表存储在类的属性中（如 `Analysis.datas` 和 `Analysis.binaries`）。同样，一旦构建目标（`EXE`、`PYZ`、`PKG`、`COLLECT`、`BUNDLE`）将输入的 TOC 列表合并为最终列表，也会执行显式列表规范化。

### Tree 类

`Tree` 类提供了一种创建 TOC 列表（描述给定目录的内容）的便捷方法：

> Tree(*root*, prefix=*run-time-folder*, excludes=*string_list*, typecode=*code* | 'DATA' )

- *root* 参数是表示目录路径的字符串。可以是绝对路径，也可以是相对于 spec 文件目录的路径。
- 可选的 *prefix* 参数是应用程序目录中一个子目录的名称，文件将被收集到该子目录中。如果未指定或设置为 `None`，文件将被收集到顶层应用程序目录中。
- 可选的 *excludes* 参数是一个由一个或多个字符串组成的列表，这些字符串与应从树中排除的 *root* 文件相匹配。列表中的项目可以是：
  - 一个名称，使具有该名称的文件或文件夹被排除
  - 一个全局匹配模式（比如 `*.ext`），使与之匹配的文件被排除
- 可选的 *typecode* 参数指定分配给 TOC 列表中所有条目的 TOC typecode 字符串。默认值为 `DATA`，适合大多数情况。

例如：

```python
extras_toc = Tree('../src/extras', prefix='extras', excludes=['tmp', '*.pyc'])
```

这将创建一个 TOC 列表 `extras_toc`，其中包含相对路径 `../src/extras` 中所有文件的条目，但省略了那些以 `tmp` 为文件名（或位于名为 `tmp` 的目录中）或以 `.pyc` 为扩展名的文件。TOC 中的每个元组都有：

- 一个形式为 `extras/{filename}` 的 *dest_name*。
- 一个 *src_name*，对应于该文件在 `../src/extras` 文件夹中的完整绝对路径（相对于 spec 文件的位置）。
- 一个值为 `DATA` （默认值）的 *typecode*。

下面是一个创建 TOC 的示例，列出了一些二进制模块：

```python
cython_mods = Tree('..src/cy_mods', excludes=['*.pyx', '*.py', '*.pyc'], typecode='EXTENSION')
```

这将创建一个 TOC 列表，其中包含 `cy_mods` 目录中每个文件条目，但不包括扩展名为 `.pyx`、`.py` 或 `.pyc` 的文件（基本上是只收集 Cython 创建的 `.pyd` 或 `.so` 模块）。该 TOC 中的每个元组都有：

- 一个与文件名相对应的 *dest_name*（所有文件都收集在顶层应用程序目录中）。
- 一个与在 `../src/cy_mods` 相对于 spec 文件的该文件的完整绝对路径相对应的 *src_name*。
- 一个值为 `EXTENSION` 的 *typecode*（也可使用 `BINARY`）。

## 检视归档

归档文件是一种包含其他文件的文件，例如 `.tar` 文件、`.jar` 文件或 `.zip` 文件。PyInstaller 中使用了两种归档文件，一种是 ZlibArchive，它允许高效地存储 Python 模块，并通过一些钩子直接导入。另一种是 CArchive，类似于 `.zip` 文件，是一种打包和压缩任意数据块的通用方式。CArchive 之所以叫 CArchive，是因为它既可以在 C 语言中操作，也可以在 Python 中操作。它们都派生自一个共同的基类，因此创建新类型的归档文件非常容易。

### ZlibArchive

ZlibArchive 包含压缩的 `.pyc` 或 `.pyo` 文件。Spec 文件中的 `PYZ` 类调用会创建一个 ZlibArchive。

ZlibArchive 中的目录是一个 Python 字典，它将一个键——即 `import` 语句中给出的成员名称与 ZlibArchive 中的查找位置和长度相关联。ZlibArchive 的所有部分都以 [marshalled](https://docs.python.org/zh-cn/3/library/marshal.html) 格式存储，因此与平台无关。

运行时使用 ZlibArchive 导入捆绑的 Python 模块。即使使用最大压缩等级，导入速度也比普通导入快。不需要搜索 `sys.path`，而是在字典中查找。没有目录操作，也不需要打开文件（文件已经打开了）。只需查找、读取和解压缩。

Python 错误跟踪将指向创建归档条目的源文件（`.pyc` 编译、捕获并保存在归档时的 `__file__` 属性）。这不会向你的用户提供任何有用的信息，但如果他们向你反馈 Python 错误跟踪，你可以理解它。

![Structure of the ZlibArchive](https://pyinstaller.org/en/v6.3.0/_images/ZlibArchive.png)

### CArchive

CArchive 可用包含任何类型的文件。它非常像 `.zip` 文件。用 Python 创建它们很容易，用 C 代码解压也很容易。CArchive 可以附加到另一个文件，如 ELF 和 COFF 可执行文件。为了实现这一点，归档的目录被放在文件的末尾，后面只有一个 cookie，它告诉我们目录从何处开始，归档本身从何处开始。

一个 CArchive 可以嵌入另一个 CArchive 中。内部归档可在原地打开和使用，无需提取。

每个目录条目都有可变长度。条目的第一个字段给出条目的长度。最后一个字段是相应打包文件的名称。文件名以 null 结束。每个成员的压缩都是可选的。

每个成员还有一个类型代码。自解压可执行文件会使用这些类型代码。如果将 `CArchive` 用作 `.zip` 文件，则无需担心代码问题。

ELF 可执行文件格式（Windows、GNU/Linux 和其他一些操作系统）允许在可执行文件的末尾连接任意数据，而不会影响其功能。因此，CArchive 的目录位于归档的末尾。可执行文件可以二进制文件的形式打开自己，找到末尾并“打开”CArchive。

![Structure of the CArchive](https://pyinstaller.org/en/v6.3.0/_images/CArchive.png)

![Structure of the Self Extracting Executable](https://pyinstaller.org/en/v6.3.0/_images/SE_exe.png)

### 使用 pyi-archive_viewer

使用 `pyi-archive_viewer` 命令可以检查任何类型的归档文件：

> `pyi-archive_viewer` *archivefile*

使用该命令，你可以检查用 PyInstaller 构建的任何归档文件（`PYZ` 或 `PKG`），或任何可执行文件（`.exe` 文件或 ELF 或 COFF 二进制文件）的内容。可以使用这些命令浏览归档：

`O name`  
打开嵌入的归档 *name*（如果省略会有提示）。例如，在查找单文件可执行文件时，可以打开其中的 `PYZ-00.pyz` 归档。

`U`  
向上一级（返回查看包含归档的页面）。

`X name`  
提取 *name*（如果省略将提示）。提示输出文件名。如果没有给定，则将成员提取到 stdout。

`Q`  
退出。

`pyi-archive_viewer` 命令有如下选项：

`-h, --help`  
显示帮助。

`-l, --log`  
快速记录日志。

`-b, --brief`  
打印可在 Python 中评估的文件内容列表。

`-r, --recursive`  
与 -l 或 -b 一起使用时，应用递归行为。

## 检视可执行文件

你可以使用 `pyi-bindepend` 来检视任何可执行文件：

> `pyi-bindepend` *executable_or_dynamic_library*

`pyi-bindepend` 命令会分析你指定的可执行文件或 DLL，并将其所有二进制依赖关系写入 stdout。这样可以轻松找出哪些 DLL 被可执行文件或另一个 DLL 所需要。

PyInstaller 在分析过程中使用 `pyi-bindepend` 来跟踪二进制扩展的依赖链。

## 创建可重复的构建

在特定情况下，当使用完全相同的依赖项构建同一个应用程序两次时，两个捆绑包应该完全逐位相同，这非常重要。

通常情况并非如此。Python 使用随机散列来制作字典和其他散列类型，这会影响编译的字节码以及 PyInstaller 内部的数据结构。因此，即使应用程序捆绑包的所有组件都相同，两个应用程序的执行方式也相同，两次编译也可能不会产生逐位完全相同的结果。

你可以在运行 PyInstaller 之前，将环境变量 `PYTHONHASHSEED` 设置为一个已知的整数值，以确保编译时产生相同的位。这将强制 Python 使用相同的随机散列序列，直到 `PYTHONHASHSEED` 被取消设置或设置为 `'random'`。例如，在如下脚本中执行 PyInstaller（适用于 GNU/Linux 和 macOS）：

```shell
# 将种子设置为已知的可重复整数值
PYTHONHASHSEED=1
export PYTHONHASHSEED
# 以 myscript 创建单文件构建
pyinstaller myscript.spec
# 校验
cksum dist/myscript/myscript | awk '{print $1}' > dist/myscript/checksum.txt
# 让 Python 再度变幻莫测
unset PYTHONHASHSEED
```

*在版本 4.8 中修改：* 在组装过程中，生成的 Windows 可执行文件 PE 头中的构建时间戳会设置为当前时间。可以通过 `SOURCE_DATE_EPOCH` 环境变量指定自定义时间戳值，以实现可重现的编译。
