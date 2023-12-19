# PyInstaller 的功能和原理

> 翻译自 [PyInstaller 文档 v6.3.0 - What PyInstaller Does and How It Does It](https://pyinstaller.org/en/v6.3.0/operating-mode.html)

本节介绍 PyInstaller 的基本思想。这些思想适用于所有平台。选项和特殊情况将在[*使用PyInstaller*](./usage.md#使用-pyinstaller) 中介绍。

PyInstaller 读取你编写的 Python 脚本。它分析你的代码，找到脚本执行所需的其他模块和库。然后搜集所有这些文件的副本——包括活动的 Python 解释器！——并把它们与你的脚本放在一个文件夹中，或可选地放在一个可执行文件中。

对于绝大多数程序，只需一条简短的命令即可完成：

```shell
pyinstaller myscript.py
```

或者增加一些选项，例如把窗口化应用程序作为单文件的可执行文件：

```shell
pyinstaller --onefile --windowed myscript.py
```

你可以将捆绑包以文件夹或文件形式分发给其他人，他们就可以执行你的程序。对你的用户来说，应用程序是独立的，他们不需要安装任何特定版本的 Python 或其他模块。实际上，他们根本不需要安装 Python。

> Note
>
> PyInstaller 的输出是针对当前操作系统和当前 Python 版本的。这意味着，要为
>
> - 不同的操作系统
> - 不同的 Python 版本
> - 32位或64位的操作系统
>
> 准备发行版，需要在该操作系统的该版本 Python 下运行 PyInstaller。执行 PyInstaller 的 Python 解释器是捆绑包的一部分，它与操作系统和字的大小（word size）有关。

## 分析：查找项目所需的文件

你的脚本还需要哪些模块和库才能运行？（这些有时被称为 "依赖项"）。

PyInstaller 会查找脚本中的所有 `import` 语句来查明。它找到已导入的模块，并在其中查找 `import` 语句，如此递归，直到获得脚本可能使用的模块的完整列表。

PyInstaller 能够理解 Python 软件包常用的 "egg" 发布格式。如果你的脚本从 "egg" 中导入了一个模块，PyInstaller 就会将该模块及其依赖项添加到需求文件集中。

PyInstaller 还了解许多主流 Python 包，包括 GUI 包 [Qt](https://www.qt.io/)（通过 [PyQt](http://www.riverbankcomputing.co.uk/software/pyqt/intro) 或 [PySide](https://doc.qt.io/qtforpython-6/) 导入）、[WxPython](http://www.wxpython.org/)、[TkInter](http://wiki.python.org/moin/TkInter)、[matplotlib](https://matplotlib.org)，以及其他主要软件包。如需完整列表，参阅 [Supported Packages](https://github.com/pyinstaller/pyinstaller/wiki/Supported-Packages)。

有些 Python 脚本导入模块的方式是 PyInstaller 无法检测到的：例如，使用带有变量数据的 `__import__` 函数、使用 `importlib.import_module`，或在运行时操作 `sys.path` 值。如果你的脚本需要 PyInstaller 不知道的文件，那么你必须提供额外帮助：

- 你可以在 `pyinstaller` 命令行中添加额外文件。
- 你可以在命令行中提供额外导入路径。
- 你可以编辑 `myscript.spec` 文件，PyInstaller 会在第一次为脚本运行时写入该文件。在 spec 文件中，你可以告知 PyInstaller 你的脚本所特有的代码模块。
- 你可以编写 "hook" 钩子文件，告知 PyInstaller 隐式导入信息。如果你为一个软件包创建了一个 "hook" 钩子，而其他用户可能也会使用这个软件包，那么可以将你的钩子文件贡献给 PyInstaller。

如果你的程序依赖于对某些数据文件的访问，你可以要求 PyInstaller 将它们也包含在捆绑包中。可以通过修改 spec 文件来实现这一点，这是[*使用 Spec 文件*](./spec-files.md#使用-spec-文件)中涉及的一个高级主题。

为了在运行时找到包含的文件，无论是否从捆绑包中运行，程序都需要能够在运行时了解其路径。这将在[*运行时信息*](./runtime-information.md#运行时信息) 中介绍。

PyInstaller *不会*包含当前操作系统的所有安装中都应该存在的库。例如在 GNU/Linux 中，它不会捆绑 `/lib` 或 `/usr/lib` 中的任何文件，因为它认为这些文件在每个系统中都能找到。

## 捆绑至单个文件夹

对 `myscript.py` 使用 PyInstaller 时，默认结果是一个名为 `myscript` 的文件夹。该文件夹包含脚本的所有依赖项，以及一个同样名为 `myscript` 的可执行文件（Windows 中为 `myscript.exe`）。

你可以把文件夹压缩成 `myscript.zip` 并发送给用户。用户只需解压缩即可安装程序，打开文件夹，启动其中的 `myscript` 可执行文件，即可运行应用程序。

使用单文件夹模式时，很容易调试构建应用程序时出现的问题。你可以看到 PyInstaller 究竟收集了哪些文件到文件夹中。

单文件夹捆绑的另一个优点是，当你修改你的代码时，只要*导入的依赖集完全相同*，就可以只发送更新后的 `myscript` 可执行文件。这通常比整个文件夹要小得多。（如果你修改了脚本，而且导入了更多或不同的依赖项，或依赖项版本升级了，你就必须重新发布整个捆绑包。）

## 单文件夹程序如何工作

捆绑程序总是在 PyInstaller bootloader（引导加载程序）中开始执行。这是文件夹中 `myscript` 可执行文件的核心。

PyInstaller bootloader 是当前活动平台（Windows、GNU/Linux、macOS 等）下的二进制可执行程序。当用户启动你的程序时，其实就是在运行 bootloader。引导程序会创建一个临时 Python 环境，该 Python 解释器能在 `myscript` 文件夹中找到所有导入的模块和库。

Bootloader 会启动一个 Python 解释器的拷贝来执行你的脚本。只要包含了所有必须的支持文件，一切都会正常进行。

（这只是一个概述。更多详情，参阅 [The Bootstrap Process in Detail](https://pyinstaller.org/en/v6.3.0/advanced-topics.html#the-bootstrap-process-in-detail)。）

## 捆绑至单个文件

PyInstaller 能够把你的脚本及其依赖项都捆绑至一个名为 `myscript`（Windows 下为 `myscript.exe`）的可执行文件中。

这样做的好处是，用户可以得到简单易理解的东西——只需启动一个可执行文件。缺点是所有相关文件（比如 README）都必须分别发布。此外，单个可执行文件的启动速度也比单文件夹捆绑稍慢一些。

在尝试捆绑至单个文件之前，确保你的应用程序在捆绑至单个文件夹时工作正常。在单文件夹模式下，诊断问题要*容易得多*。

## 单文件程序如何工作

Bootloader 也是单文件捆绑包的核心。启动时，它会在该操作系统的临时文件夹位置创建一个名为 `_MEIxxxxxx` 的临时文件夹。其中 *xxxxxx* 是一串随机数。

单可执行文件中包含了脚本使用的所有 Python 模块的嵌入式归档，以及所有非 Python 支持文件（比如 `.so` 文件）的压缩副本。Bootloader 会解压缩支持文件并将其副本写入临时文件夹。这个过程需要一点时间，这也正是单文件应用程序比单文件夹应用程序启动速度稍慢的原因。

> Note
>
> PyInstaller 目前不会保留文件属性。参阅 [#3926](https://github.com/pyinstaller/pyinstaller/issues/3926)。

创建临时文件夹后，bootloader 将在临时文件夹中继续执行与单文件夹捆绑完全相同的程序。当捆绑代码运行结束时，bootloader 会删除临时文件夹。

（在 GNU/Linux 和相关系统中，挂载 `/tmp` 目录时可以选择 "no-execution" 选项。该选项与 PyInstaller 的单文件捆绑不兼容，因为后者需要在 `/tmp` 中执行代码。如果你了解目标环境，或可以使用 `--runtime-tmpdir` 作为一种变通办法。）

由于程序会创建一个具有唯一名称的临时文件夹，因此你可以运行多个程序副本而不必担心相互干扰。不过，运行多个副本会消耗大量磁盘空间，因为没有任何东西是共享的。

如果程序崩溃或被杀死（在 Unix 上 kill -9、在 Windows 上被任务管理器杀死、在 macOS 上 "强制退出"），`_MEIxxxxxx` 文件夹不会被删除。因此，如果你的应用程序经常崩溃，你的用户就会因为多个 `_MEIxxxxxx` 临时文件夹而耗费磁盘空间。

使用 `--runtime-tmpdir` 命令行选项可以控制 `_MEIxxxxxx` 文件夹的位置。指定的路径将存储在可执行文件中，bootloader 将在指定位置创建 `_MEIxxxxxx` 文件夹。详情请参阅[*定义提取位置*](./usage.md#定义提取位置)。

> Note
>
> *不要*给 Windows 上的单文件可执行文件赋予管理员权限（"以管理员身份运行此程序"）。恶意攻击者可能在 bootloader 准备临时文件夹中的一个共享库时破坏它。虽然这种可能性不大，但也并非绝无可能。一般来说，在分发特权程序时，应通过文件权限机制来防止共享库或可执行文件被篡改。否则，拥有这些文件写入权限的未提权进程可能会通过修改这些文件来提升自身权限。
>
> Note
>
> 使用 `os.setuid()` 的应用程序可能会遇到权限错误。调用 `setuid` 后，运行捆绑应用程序的临时文件夹可能会无法读取。如果你的脚本需要调用 `setuid`，最好使用单文件夹模式，以便更好地控制其文件的权限。

## 使用控制台窗口

默认情况下，bootloader 会创建一个命令行控制台（GNU/Linux 和 macOS 中的终端窗口，Windows 中的命令窗口）。它将此窗口提供给 Python 解释器用于标准输入和输出。你的脚本中使用的 `print()` 和 `input()` 都会指向这里。Python 的错误信息和默认日志输出也会显示在控制台窗口中。

Windows 和 macOS 下的一个选项是告知 PyInstaller 不提供控制台窗口。bootloader 启动 Python 时不会为标准输出或输入提供目标。当你的脚本有图形界面供用户输入，并能正确报告自己的诊断结果时，可以使用此选项。

如 Python 教程附录中的[*可执行的Python脚本*](https://docs.python.org/zh-cn/3/tutorial/appendix.html#executable-python-scripts)所述，对于 Windows 操作系统，扩展名为 *.pyw* 的文件会隐藏通常出现的控制台窗口。类似地，在 PyInstaller 中使用 `myscript.pyw` 脚本时，也不会出现控制台窗口。

## 隐藏源代码

捆绑应用程序不包含任何源代码。不过，PyInstaller 捆绑了编译后的 Python 脚本（`.pyc` 文件）。这些文件原则上可以反编译，显示代码的逻辑。

如果想更彻底地隐藏源代码，一个可行的办法是用 [Cython](http://www.cython.org/) 编译某些模块。使用 Cython，可以将 Python 模块转换为 C 语言，然后将 C 语言编译为机器语言。PyInstaller 可以跟踪引用 Cython C 对象模块的导入语句，并捆绑它们。
