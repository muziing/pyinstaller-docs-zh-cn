# 系统要求

> 翻译自 [PyInstaller 文档 v6.2.0 - Requirements](https://pyinstaller.org/en/v6.2.0/requirements.html)

## Windows

PyInstaller 可在 Windows 8 及更新版本中运行。它可以创建图形窗口应用程序（不需要命令窗口的应用程序）。

## macOS

PyInstaller 可在 macOS 10.15 (Catalina) 或更新版本上运行。它可以构建图形窗口应用程序（不需要使用终端窗口的应用程序）。PyInstaller 构建的应用程序与运行它的 macOS 版本及后续版本兼容。它可以在任一架构的 macOS 机器上构建 `x86_64`、`arm64` 或混合 *universal2* 二进制文件。详情参阅 [macOS multi-arch support](https://pyinstaller.org/en/v6.2.0/feature-notes.html#macos-multi-arch-support)。

## GNU/Linux

PyInstaller 需要 `ldd` 终端应用程序来发现每个程序或共享库所需的共享库。它通常存在于发行包 `glibc` 或 `libc-bin` 中。

它还需要 `objdump` 终端应用程序从对象文件中提取信息，以及 `objcopy` 终端应用程序向 bootloader 添加数据。这些通常可以在发行包 `binutils` 中找到。

## AIX、Solaris、FreeBSD 与 OpenBSD

有用户报告称在这些平台上成功运行了 PyInstaller，但官方没有在这些平台上进行测试。需要可以使用 `ldd` 和 `objdump` 命令。

每个捆绑应用程序都包含一个 bootloader 的副本，这是一个用于设置应用程序并启动它的程序（参阅 [The Bootstrap Process in Detail](https://pyinstaller.org/en/v6.2.0/advanced-topics.html#the-bootstrap-process-in-detail)）。

使用 [pip](http://www.pip-installer.org/) 安装 PyInstaller 时，安装程序会尝试为该平台构建 bootloader。如果成功，安装将继续，PyInstaller 就可以使用了。

如果 pip 安装失败或没有使用 pip 安装，则必须手动编译 bootloader。具体过程参阅 [Building the Bootloader](https://pyinstaller.org/en/v6.2.0/bootloader-building.html#building-the-bootloader)。
