# 当发生错误时

> 翻译自 [PyInstaller 文档 v6.3.0 - When Things Go Wrong](https://pyinstaller.org/en/v6.3.0/when-things-go-wrong.html)

前面几篇中的信息涵盖了 PyInstaller 的大部分常规用法。然而，Python 和第三方库的变化是无穷无尽和不可预测的。当你尝试捆绑应用程序时，可能会出现 PyInstaller 本身或捆绑应用程序以 Python 回溯终止的情况。那么，在寻求技术帮助之前，请依次考虑以下操作。

## 针对具体问题的配方和示例

PyInstaller [FAQ](https://github.com/pyinstaller/pyinstaller/wiki/FAQ) 页面有一些常见问题的解决方法。我们的 [PyInstaller Recipes](https://github.com/pyinstaller/pyinstaller/wiki/Recipes) 页面提供了一些高级用法和常见问题的代码示例。其中包括：

- 比上述方法更复杂的数据文件收集方法（[*向软件包添加文件*](./spec-files.html#向软件包添加文件)）。
- 捆绑一个典型的 Django 应用程序。
- 使用运行时钩子设置 PyQt5 API 级别。
- Windows 下多进程限制的变通方法。

等。其中许多配方都是由用户贡献的。请随时贡献更多！

## 找出问题所在

### 构建时信息

当 `Analysis` 步骤运行时，它会产生错误和警告信息。如果 `--log-level` 选项允许，这些信息会显示在命令行之后。此外，Analysis 还会在 `work-path=` 目录中名为 `build/name/warn-name.txt` 的警告文件中放置信息。

当 Analysis 检测到导入时，如果找不到模块名称，就会生成一条信息。当一个类或函数声明在一个包（一个 `__init__.py` 模块）中，而导入指定了 `package.name` 时，也会产生一条信息。在这种情况下，Analysis 无法判断 name 是子模块还是包。

"module not found" 信息不被归为错误，因为通常有很多条这样的信息。例如，许多标准模块为不同的平台选择性地导入模块，而这些被导入的模块可能存在也可能不存在。

所有 "module not found" 信息都会被写入 `build/name/warn-name.txt` 文件。由于数量众多，不会显示在标准输出中。检查警告文件；通常会有几十个模块未找到，但它们的缺失不会产生任何影响。

当你运行捆绑应用程序，但以 ImportError 终止时，就该检查警告文件了。然后参阅下面的 [*帮助 PyInstaller 查找模块*](#帮助-pyinstaller-查找模块) 继续解决。

### 构建时依赖图

每次运行时，PyInstaller 都会在构建文件夹中写入一个关于依赖关系的交叉引用文件：在 `work-path=` 目录中的 `build/name/xref-name.html` 是一个 HTML 文件，它列出了导入图的全部内容，显示了哪些模块被哪些模块导入。你可以在任意网页浏览器中打开它。找到一个模块名称，然后不断点击 "imported by" 链接，直到找到导致该模块被包含在内的顶级导入。

如果在 `pyinstaller` 命令中指定 `--log-level=DEBUG`，PyInstaller 会额外生成一个 [GraphViz](https://graphviz.org/) 输入文件，表示依赖关系图。该文件是 `work-path=` 目录中的 `build/name/graph-name.dot`。你可以使用任何 [GraphViz](https://graphviz.org/) 命令（如 `dot`）处理该文件，来生成导入依赖关系的图形显示。

这些文件非常大，因为即使是最简单的 "hello world" Python 程序最终也会包含大量的标准模块。因此图形文件在此发行版中不是很有用。

### 构建时 Python 错误

PyInstaller 有时会以引发一个 Python 异常而终止。大多数情况下，异常消息中的原因都很明确，例如 "Your system is not supported"（你的系统不支持）或 "Pyinstaller requires at least Python 3.8"（PyInstaller 要求至少 Python 3.8）。其他情况则清楚地表明存在一个应该报告的 bug。

然而，其中一个错误可能令人费解：`IOError("Python library not found!")`。PyInstaller 需要捆绑 Python libaray，也是 Python 解释器的主要部分，以动态加载库的形式链接。该文件的名称和位置因使用的平台而异。有些 Python 安装默认不包含动态 Python library（可能存在静态链接的，但无法使用）。你可能需要安装某种开发包。或者，library 可能存在，但不在 PyInstaller 搜索的文件夹中。

在不同的操作系统中，PyInstaller 查找 Python library 的位置不同，但在大多数系统中都会检查 `/lib` 和 `/usr/lib`。如果无法将 Python library 放在此处，请尝试在 GNU/Linux 的环境变量 `LD_LIBRARY_PATH` 或 macOS 的环境变量 `DYLD_LIBRARY_PATH` 中设置正确的路径。

### 获取调试信息

`--debug=all` 选项（及其[子选项](./usage.md#如何生成)）提供了大量诊断信息。在开发复杂软件包、应用程序似乎无法启动，或只是想了解运行时如何工作时，这些信息可能非常有用。

通常情况下，调试进度信息会发送到标准输出。如果在捆绑 Windows 应用程序时使用了 `--windowed` 选项，则会将调试进度信息发送到任何附加的调试器。如果你不使用（或者没有）调试器，可以使用免费的（:beer:）[DebugView](https://docs.microsoft.com/en-us/sysinternals/downloads/debugview) 工具来显示这些信息。它必须在运行捆绑应用程序之前启动。

对于 `--windowed` macOS 应用程序，则不会显示。

建议在生产版本中不使用 `--debug` 捆绑。调试信息需要系统调用，会影响性能。

### 获取 Python 的详尽导入信息

你可以使用 `--debug=imports` 选项（参见上文[*获取调试信息*](#获取调试信息)），这会将 `-v`（verbose imports）标志传递给内嵌的 Python 解释器。这可能非常有用。即使对于那些表面看起来可以正常运行的应用程序，它也能提供一些信息，以确保应用程序从捆绑中获取所有导入，而不是泄漏到本地安装的 Python 中。

使用 `--windowed` 选项时，Python 详细信息和警告信息总是转到标准输出，并且不可见。切记不要在生产版本中使用该选项。

### 找出 GUI 应用程序无法启动的原因

如果使用 `--windowed` 选项，捆绑应用程序可能无法启动，并显示类似 `Failed to execute script my_gui` 的错误信息。在这种情况下，你需要获得更多的详细输出，以找出问题所在。

- 对于 macOS，可以通过命令行运行应用程序，即，在*终端（Terminal）* 中运行 `./dist/my_gui` 而不是点击 `my_gui.app`。
- 对于 Windows 系统，需要以不使用 `--windowed` 选项方式重新捆绑应用程序。然后可以通过命令行运行生成的可执行文件，即 `my_gui.exe`。
- 对于 Unix 和 GNU/Linux 没有 `--windowed` 选项。总之，如果 GUI 应用程序启动失败，可以通过命令行运行应用程序，即 `./dist/my_gui`。

这样就能找出妨碍应用程序初始化的相关错误，然后就可以继续执行其他调试步骤了。

### Operation not permitted 错误

如果使用了 `--onefile`，但程序运行失败，出现类似这样的错误：

```shell
./hello: error while loading shared libraries: libz.so.1:
failed to map segment from shared object: Operation not permitted
```

造成这种情况的原因可能是 /tmp 目录权限错误（例如，文件系统在挂载时带有 `noexec` 标志）。

解决这个问题的一种简单方法是在环境变量 TMPDIR 中设置一个路径，指向一个无 `noexec` 标志的文件系统挂载中的目录，例如：

```shell
export TMPDIR=/var/tmp/
```

## 帮助 PyInstaller 查找模块

### 扩展路径

如果 Analysis 发现需要某个模块被需要，但却找不到，通常是因为脚本在操作 `sys.path`。在这种情况下，最简单的方法是使用 `--paths` 选项列出所有脚本可能搜索导入的其他地方：

```shell
pyi-makespec --paths=/path/to/thisdir \
                --paths=/path/to/otherdir myscript.py
```

这些路径将在 spec 文件的 `pathex` 参数中注明。

在分析过程中，它们将被添加到当前的 `sys.path` 中。

### 列出隐式导入

如果 Analysis 认为它已经找到了所有导入，但应用程序却因导入错误而失败，那么问题就出在隐式导入上；也就是是说，一个对分析阶段不可见的导入。

当代码使用 `__import__`、`importlib.import_module`、`exec` 或 `eval` 时，可能会出现隐式导入。当一个扩展模块使用 Python/C API 进行导入时，也会出现隐式导入。发生这种情况时，Analysis 什么也检测不到。运行时不会有任何警告，只有一个 ImportError。

要找到这些隐式导入，使用 `--debug=imports` 标志（见上文[*获取 Python 的详尽导入信息*](#获取-python-的详尽导入信息)）构建应用程序并运行它。

一旦知道需要使用哪些模块，就可以使用 `--hidden-import` 命令选项、编辑 spec 文件或使用钩子文件（见 [Understanding PyInstaller Hooks](https://pyinstaller.org/en/v6.3.0/hooks.html#understanding-pyinstaller-hooks)），将需要的模块添加到捆绑中。

### 扩展一个包的 `__path__`

Python 允许脚本通过 `__path__` 机制扩展用于导入的搜索路径。通常，一个导入的模块的 `__path__` 只有一个条目，即找到 `__init__.py` 的目录。但是 `__init__.py` 可以自由地扩展它的 `__path__` 来包含其他目录。例如，`win32com.shell.shell` 模块实际上解析为 `win32com/win32comext/shell/shell.pyd`。这是因为 `win32com/__init__.py` 将 `../win32comext` 添加到了它的 `__path__`。

由于被导入模块的 `__init__.py` 在分析过程中并未实际执行，因此 PyInstaller 无法看到它对 `__path__` 所做的更改。我们使用与解决隐式导入相同的钩子机制来解决这个问题，并附加了一些额外的逻辑，参阅 [Understanding PyInstaller Hooks](https://pyinstaller.org/en/v6.3.0/hooks.html#understanding-pyinstaller-hooks)。

注意，以这种钩子方式操作 `__path__` 只对 Analysis 有影响。在运行时，所有的导入都会在捆绑中被拦截和满足。`win32com.shell` 的解析方式与 `win32com.anythingelse` 相同，而 `win32com.__path__` 对 `../win32comext` 一无所知。

有时，这还不够。

### 改变运行时行为

运行时钩子可以处理更多奇怪的情况。这些小脚本会在主脚本运行前对环境进行操作，从而为脚本提供额外的顶层代码。

有两种方法提供运行时钩子。可以使用选项 `--runtime-hook`=*path-to-script* 来指定。

第二种则是，部分运行时钩子已提供。在分析结束时，Analysis 段落生成的模块列表中的名称，会在 PyInstaller 安装文件夹中的 `loader/rthooks.dat` 中进行查找。这个文本文件是一个 Python 字典，键是模块名称，值是钩子脚本路径名列表。如果匹配，那些钩子脚本就会包含在捆绑的应用程序中，并在主脚本启动前被调用。

使用 `--runtime-hook` 选项指定的钩子将按照指定的顺序执行，并在任何已安装的运行时钩子之前执行。如果指定 `--runtime-hook=file1.py --runtime-hook=file2.py`，那么运行时的执行顺序将是：

1. `file1.py` 的代码。
2. `file2.py` 的代码。
3. 在 `rthooks/rthooks.dat` 中找到的为所含模块指定的钩子。
4. 你的主脚本。

以这种方式调用的钩子，虽然需要谨慎处理其导入内容，但几乎可以做任何事情。编写运行时钩子的另一个用途是覆盖某些模块的函数或变量。Django 运行时钩子就是一个很好的例子（参见 PyInstaller 文件夹中的 `loader/rthooks/pyi_rth_django.py`）。Django 动态导入一些模块并寻找一些 `.py` 文件。但是在单文件捆绑包中没有 `.py` 文件。我们需要覆盖 `django.core.management.find_commands` 函数，使其只返回一个值列表。运行时钩子的做法如下：

```python
import django.core.management
def _find_commands(_):
    return """cleanup shell runfcgi runserver""".split()
django.core.management.find_commands = _find_commands
```

## 获取最新版本

如果你有理由认为发现了 PyInstaller 中的错误，可以尝试下载最新的开发版本。该版本可能有 [PyPI](https://pypi.python.org/pypi/PyInstaller/) 尚未提供的修复或功能。可以从 [PyInstaller Downloads](https://github.com/pyinstaller/pyinstaller/releases) 页面下载最新的稳定版和最新的开发版。

也可以使用 [pip](http://www.pip-installer.org/) 直接安装最新版本的 PyInstaller：

```shell
pip install https://github.com/pyinstaller/pyinstaller/archive/develop.zip
```

## 向他人寻求帮助

如果上述建议都无济于事，请通过 [*PyInstaller 邮件列表*](https://groups.google.com/forum/#!forum/pyinstaller) 寻求帮助。

然后，如果你认为有可能在 PyInstaller 中发现了错误，请参阅 [How to Report Bugs](https://github.com/pyinstaller/pyinstaller/wiki/How-to-Report-Bugs) 页面。
