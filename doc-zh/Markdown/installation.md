# 如何安装 PyInstaller

> 翻译自 [PyInstaller 文档 v6.2.0 - installation](https://pyinstaller.org/en/v6.2.0/installation.html)

PyInstaller 可作为普通 Python 软件包获取。已发布版本的源代码压缩包可从 [PyPi](https://pypi.python.org/pypi/PyInstaller/) 获取，但使用 [pip](http://www.pip-installer.org/) 安装最新版本更为方便：

```shell
pip install pyinstaller
```

要将已有的 PyInstaller 安装升级到最新版本，使用：

```shell
pip install --upgrade pyinstaller
```

要安装当前的开发版本，使用：

```shell
pip install https://github.com/pyinstaller/pyinstaller/tarball/develop
```

要直接使用 pip 内置的 git 签出支持进行安装，使用：

```shell
pip install git+https://github.com/pyinstaller/pyinstaller
```

或安装特定分支（比如，`develop`）：

```shell
pip install git+https://github.com/pyinstaller/pyinstaller@develop
```

## 从源码压缩包中安装

PyInstaller 已发布版本的源代码压缩包可在 [PyPI](https://pypi.python.org/pypi/PyInstaller/) 和 [PyInstaller Downloads](https://github.com/pyinstaller/pyinstaller/releases) 页面获取。

> Note
>
> 尽管源代码压缩包提供了 `setup.py` 脚本，但通过 `python setup.py install` 进行安装已被弃用，不应再使用。取而代之的是在解压后的源代码目录下运行 `pip install .`，如下所述。

安装步骤如下：

1. 解压源码压缩包。
2. 进入解压后的源码目录。
3. 在解压后的源代码目录运行 `pip install .`。如果要安装到系统级的 Python 中，则需要管理员权限。

同样的步骤也适用于从手动 git 签出安装：

```shell
git clone https://github.com/pyinstaller/pyinstaller
cd pyinstaller
pip install .
```

如果你打算对源代码进行修改，并且希望修改立刻生效而无需每次都重新安装软件包，则可以在可编辑模式下安装：

```shell
pip install -e .
```

对于除 Windows、GNU/Linux 和 macOS 之外的平台，必须首先为该平台构建引导加载程序（bootloader）：参阅 [Building the Bootloader](https://pyinstaller.org/en/v6.2.0/bootloader-building.html#building-the-bootloader)。在构建引导加载程序之后，使用 `pip install .` 命令来完成安装。

## 验证安装

在所有平台上，`pyinstaller` 命令现在都应存在于可执行路径中。要验证这一点，输入命令：

```shell
pyinstaller --version
```

结果应类似于发布版本的 `6.n`，以及开发分支的 `6.n.dev0-xxxxxx`。

如果找不到命令，请确保执行路径（环境变量）包含正确的目录：

- Windows： `C:\PythonXY\Scripts` 其中 *XY* 表示 Python 的主要和次要版本号，例如 `C:\Python38\Scripts` 对应 Python 3.8
- GNU/Linux： `/usr/bin/`
- macOS （使用苹果提供的默认 Python）： `/usr/bin`
- macOS （使用通过 homebrew 安装的 Python）： `/usr/local/bin`
- macOS （使用通过 macports 安装的 Python）： `/opt/local/bin`

在 Windows 系统中显示当前 PATH 的命令是 `echo %path%`，而在其他系统中则是 `echo $PATH`。

> Note
>
> 如果由于脚本目录不在 `PATH` 中而无法使用 `pyinstaller` 命令，可以通过运行 `python -m PyInstaller`（注意模块名区分大小写）来调用 `PyInstaller` 模块。当你在多个 Python 环境中安装了 PyInstaller，而不能确定将从哪个安装中运行 `pyinstaller` 命令时，这种调用方式也很有用。

## 已安装的命令

安装完成后，这些命令就会进入可执行路径：

- `pyinstaller` 是构建捆绑应用程序的主要命令。参阅 [Using PyInstaller](https://pyinstaller.org/en/v6.2.0/usage.html#using-pyinstaller)。
- `pyi-makespec` 用于创建 spec 文件。参阅 [使用 Spec 文件](./spec-files.md)。
- `pyi-archive_viewer` 用于检查捆绑应用程序。参阅 [Inspecting Archives](https://pyinstaller.org/en/v6.2.0/advanced-topics.html#inspecting-archives)。
- `pyi-bindepend` 用于显示可执行文件的依赖关系。参阅 [Inspecting Executables](https://pyinstaller.org/en/v6.2.0/advanced-topics.html#inspecting-executables)。
- `pyi-grab_version` 用于从 Windows 可执行文件中提取版本资源。参阅 [Capturing Windows Version Data](https://pyinstaller.org/en/v6.2.0/usage.html#capturing-windows-version-data)。
- `pyi-set_version` 可用于将先前提取的版本资源应用于现有的 Windows 可执行文件。
