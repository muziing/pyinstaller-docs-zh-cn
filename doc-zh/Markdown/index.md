# PyInstaller 指南（简体中文翻译版）

## 翻译信息

- 翻译自： [PyInstaller 文档 v6.2.0 - PyInstaller Manual](https://pyinstaller.org/en/v6.2.0/index.html)
- 译者：[muzing](https://github.com/muziing) [\<muzi2001@foxmail.com\>](mailto:muzi2001@foxmail.com)
- 文档源码仓库：<https://github.com/muziing/pyinstaller-docs-zh-cn>

## PyInstaller 简介

| 键   | 值                                                           |
| ---- | ------------------------------------------------------------ |
| 版本 | PyInstaller 6.2.0                                            |
| 主页 | <https://pyinstaller.org/>                                   |
| 联系 | [pyinstaller@googlegroups.com](mailto:pyinstaller@googlegroups.com) |
| 作者 | David Cortesi, based on structure by Giovanni Bajo & William Caban, based on Gordon McMillan’s manual |
| 版权 | 本文档已置于公共域。                                         |

PyInstaller 将 Python 应用程序及其所有依赖项捆绑到单个软件包中。用户无需安装 Python 解释器或任何模块，即可运行打包后的应用程序。PyInstaller 支持 Python 3.8 及其更新版本，并能正确捆绑 numpy、matplotlib、PyQt、wxPython 等许多主流 Python 包。

PyInstaller 已经过 Windows、MacOS X 和 Linux 测试。不过，它并不是一个交叉编译器；要制作 Windows 应用程序就需要在 Windows 上运行 PyInstaller，要制作 Linux 应用程序就需要在 Linux 上运行它，依此类推。PyInstaller 已经成功地在 AIX、Solaris、FreeBSD 和 OpenBSD 上使用，但针对这些平台的测试并不是我们持续集成测试的一部分，开发团队也不能保证（这些平台的所有代码都来自外部贡献）PyInstaller 将能够在这些平台上运行，或得到持续支持。

## 快速入门

确保你的系统符合[要求](./requirements.md#系统要求)，然后从 PyPI 安装 PyInstaller：

```shell
pip install -U pyinstaller
```

打开命令提示符/shell 窗口，导航到你的 .py 文件所在目录，然后用如下命令创建应用程序：

```shell
pyinstaller your_program.py
```

现在应该可以在 dist 文件夹中找到你的捆绑应用程序了。

## 目录

1. [系统要求](requirements.md)
2. [许可证](license.md)
3. [贡献指南](contributing.md)
4. [如何安装 PyInstaller](installation.md)
5. [PyInstaller 的功能和原理](operating-mode.md)
6. [使用 PyInstaller](usage.md)
7. [运行时信息](runtime-information.md)
8. [使用 Spec 文件](spec-files.md)
9. [特定功能注意事项](feature-notes.md)
10. [当发生错误时](when-things-go-wrong.md)
11. [进阶话题](advanced-topics.md)
12. [理解 PyInstaller 钩子](hooks.md)
13. [钩子配置选项](hooks-config.md)
14. [构建 Bootloader](bootloader-building.md)
15. [pyi-makespec 命令](pyi-makespec.md)
