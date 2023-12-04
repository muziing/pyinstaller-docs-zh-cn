# 运行时信息

> 翻译自 [PyInstaller 文档 v6.2.0 - Run-time Information](https://pyinstaller.org/en/v6.2.0/runtime-information.html)

你的应用程序在捆绑包中运行时，行为应当和从源码运行时完全一样。不过，你可能希望在运行时了解应用程序是在正在源码中运行还是已被捆绑（"frozen"）。你可以使用下面的代码来检查 "在捆绑包里吗？"：

```python
import sys
if getattr(sys, 'frozen', False) and hasattr(sys, '_MEIPASS'):
    print('running in a PyInstaller bundle')
else:
    print('running in a normal Python process')
```

当捆绑应用程序启动时，引导加载程序会设置 `sys.frozen` 属性，并在 `sys._MEIPASS` 中存储捆绑文件夹的绝对路径。对于单文件夹捆绑，这是指捆绑中 `_internal` 文件夹的路径；对于单文件捆绑，则是引导加载程序创建的临时文件夹的路径（参阅[*单文件程序如何工作*](./operating-mode.md#单文件程序如何工作)）。

应用程序运行时，可能需要访问以下位置的数据文件：

- 与之捆绑的文件（参阅[*添加数据文件*](./spec-files.md#添加数据文件)）。
- 用户添加的与捆绑包放在一起的文件，例如同一文件夹中的文件。
- 用户当前工作目录中的文件。

程序可以访问数个变量来实现这些用途。

## 使用 `__file__`

当你的程序未被捆绑时，Python 变量 `__file__` 指向它所包含于的模块的当前路径。从捆绑脚本导入模块时，PyInstaller 引导加载程序会将模块的 `__file__` 属性设置为相对于捆绑文件夹的正确路径。

例如，如果你从一个捆绑脚本中导入 `mypackage.mymodule`，那么该模块的 `__file__` 属性将会是 `sys._MEIPASS + 'mypackage/mymodule.pyc'`。因此，如果你在 `mypackage/file.dat` 处有一个数据文件，而又将它添加到了 `mypackage/file.dat` 处的捆绑包中，那么通过下面的代码就可以获取其路径（在捆绑和非捆绑的情况下均可）：

```python
from os import path
path_to_dat = path.abspath(path.join(path.dirname(__file__), 'file.dat'))
```

在主脚本（`__main__` 模块）中，`__file__` 变量包含脚本文件的路径。在 Python 3.8 及更早版本中，这个路径是绝对路径或相对路径（取决于脚本是如何传递给 `python` 解释器的）；而在 Python 3.9 及更高版本中，它总是绝对路径。在捆绑脚本中，PyInstaller 引导加载程序总是将 `__main__` 模块中的 `__file__` 变量设置为捆绑目录中的绝对路径，就像字节编译的入口点脚本存在于那里一样。

例如，如果你的入口点脚本名为 `program.py`，那么捆绑脚本中的 `__file__` 属性将指向 `sys._MEIPASS + 'program.py'`。因此，可以直接使用 `sys._MEIPASS` 或通过主脚本内 `__file__` 的父路径来定位相对于主脚本的数据文件。

下面的示例将获取文件 `other-file.dat` 的路径，如果没有捆绑，则该文件位于主脚本旁边；如果捆绑来，则位于捆绑文件夹内：

```python
from os import path
bundle_dir = path.abspath(path.dirname(__file__))
path_to_dat = path.join(bundle_dir, 'other-file.dat')
```

Or, if you'd rather use [pathlib](https://docs.python.org/zh-cn/3/library/pathlib.html):下面的示例将获取文件 `other-file.dat` 的路径，如果没有捆绑，该文件位于主脚本旁边；如果捆绑了，则位于捆绑文件夹内：

```python
from pathlib import Path
path_to_dat = Path(__file__).resolve().with_name("other-file.dat")
```

*在版本 4.3 中更改：* 以前，入口点脚本（`__main__` 模块）的 `__file__` 属性只被设置为其基类名，而不是它在捆绑目录中的完整（绝对或相对）路径。因此，PyInstaller 文档曾建议使用 `sys._MEIPASS` 作为相对于捆绑入口点脚本的资源定位方式。现在，`__file__` 总是设置为绝对路径，并且是定位此类资源的首选方式。

### 将数据文件放在捆绑包内的预期位置

要将数据文件放在代码期望的位置（即，相对于主脚本或捆绑目录），可以使用 `--add-data="source:dest" <--add-data>` 命令行开关的 **dest** 参数。假设你通常在名为 `my_script.py` 的文件中使用如下代码来定位同一文件夹中的文件 `file.dat`：

```python
from os import path
path_to_dat = path.abspath(path.join(path.dirname(__file__), 'file.dat'))
```

或 [pathlib](https://docs.python.org/zh-cn/3/library/pathlib.html) 等效版本：

```python
from pathlib import Path
path_to_dat = Path(__file__).resolve().with_name("file.dat")
```

如果 `my_script.py` **不是**某个包的一部分（不在一个包含 `__init__.py` 的文件夹中），那么其 `__file__` 将是 `[app root]/my_script.pyc`。也就是说，如果使用如下命令，把 `file.dat` 放在软件包的根目录中：

```shell
PyInstaller --add-data="/path/to/file.dat:."
```

则无需更改 `my_script.py`，即可在运行时正确找到它。

> Note
>
> Windows 用户需要把上一行命令中的 `:` 替换为 `;`。

如果是在一个包或库（例如 `my_library.data`）中检查 `__file__`，那么 `__file__` 将是 `[app root]/my_library/data.pyc`，`--add-data` 应镜像该文件：

```shell
PyInstaller --add-data="/path/to/my_library/file.dat:./my_library"
```

不过，在这种情况下，改用 [spec 文件](./spec-files.md#使用-spec-文件) 并使用 `PyInstaller.utils.hooks.collect_data_files` 辅助函数要容易得多：

```python
from PyInstaller.utils.hooks import collect_data_files

a = Analysis(...,
             datas=collect_data_files("my_library"),
             ...)
```

## 使用 `sys.executable` 与 `sys.argv[0]`

运行普通 Python 脚本时，`sys.executable` 是执行程序，即 Python 解释器的路径。在被冻结的应用程序中，`sys.executable` 也是执行程序的路径，但此时的执行程序并非 Python；而是单文件应用程序中的引导加载程序或单文件夹应用程序中的可执行文件。这样就有了一种可靠的方法来获取用户启动冻结可执行文件的实际位置的方法。

`sys.argv[0]` 的值是用户命令中使用的名称或相对路径。它可能是相对路径，也可能是绝对路径，这取决于平台和应用程序的启动方式。

如果用户通过符号链接启动应用程序，`sys.argv[0]` 将使用符号名称，而 `sys.executable` 则是可执行文件的实际路径。有时，同一个应用程序会以不同的名称链接，并根据启动时使用的名称而又不同的行为。为处理这种情况，可以测试 `os.path.basename(sys.argv[0])`。

另一方面，有时会要求用户将可执行文件存储在与其操作的文件相同的文件夹中，例如音乐播放器应与其播放的音频文件存储在相同的文件夹中。在这种情况下，可以使用 `os.path.dirname(sys.executable)`。

下面的小程序探讨了其中的一些可能性。将其保存为 `directories.py`。将其作为 Python 脚本执行，然后捆绑成单文件夹应用程序。然后将其捆绑成单文件应用程序并直接启动或通过符号链接启动：

```python
#!/usr/bin/env python3
import sys, os
frozen = 'not'
if getattr(sys, 'frozen', False):
    # 当前正在捆绑包中运行
    frozen = 'ever so'
    bundle_dir = sys._MEIPASS
else:
    # 当前正在普通 Python 环境中运行
    bundle_dir = os.path.dirname(os.path.abspath(__file__))
print('we are',frozen,'frozen')
print('bundle dir is', bundle_dir)
print('sys.argv[0] is', sys.argv[0])
print('sys.executable is', sys.executable)
print('os.getcwd is', os.getcwd())
```

## 对 LD_LIBRARY_PATH / LIBPATH 的考量

在 GNU/Linux 和 \*BSD 上的环境变量 *LD_LIBRARY_PATH*，或 AIX 上的环境变量 *LIBPATH*，是库的搜索路径，用于发现库。

如果它存在，PyInstaller 会将其原始值保存到 *\*\_ORIG*，然后修改搜索路径，以便捆绑代码首先找到捆绑库。

但如果你的代码执行的是一个系统程序，你通常不希望这个系统程序加载你的捆绑库（可能与你的系统程序不兼容）——它应该像通常那样从系统位置加载正确的库。

因此，在使用系统程序创建子进程之前，需要先恢复原来的路径。

```python
env = dict(os.environ)  # 创建环境的副本
lp_key = 'LD_LIBRARY_PATH'  # 适用于 GNU/Linux 或 *BSD 平台
lp_orig = env.get(lp_key + '_ORIG')
if lp_orig is not None:
    env[lp_key] = lp_orig  # 恢复未经修改的原始值
else:
    # 未设置 LD_LIBRARY_PATH 时会出现这种情况。
    # 在万不得已的情况下，删除 env var：
    env.pop(lp_key, None)
p = Popen(system_cmd, ..., env=env)  # 创建进程
```
