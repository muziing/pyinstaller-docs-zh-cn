# PyInstaller 文档 - 简体中文翻译

> [PyInstaller 文档](https://pyinstaller.org) 非官方简体中文翻译，基于 v6.2.0。
>
> Simplified Chinese Translation of [PyInstaller Documentation](https://pyinstaller.org) (Unofficial).

## 目录

1. [系统要求](./doc-zh/Markdown/requirements.md)
2. [许可证](./doc-zh/Markdown/license.md)
3. [贡献指南](./doc-zh/Markdown/contributing.md)
4. [如何安装 PyInstaller](./doc-zh/Markdown/installation.md)
5. [PyInstaller 的功能和原理](./doc-zh/Markdown/operating-mode.md)
6. [使用 PyInstaller](./doc-zh/Markdown/usage.md)
7. [运行时信息](./doc-zh/Markdown/runtime-information.md)
8. [使用 Spec 文件](./doc-zh/Markdown/spec-files.md)
9. 翻译工作仍在进行中，敬请期待……

## 翻译声明

- 本套译文除明确标注外，均由 [muzing](https://github.com/muziing) 翻译，转载时请包含指向[此仓库](https://github.com/muziing/pyinstaller-docs-zh-cn)的链接与译者姓名。
- 本翻译仅供参考，由于双语间固有表达方式差异、专业术语习惯性翻译差异以及译者水平有限，不保证内容绝对正确。
- 原作者：David Cortesi, based on structure by Giovanni Bajo & William Caban, based on Gordon McMillan’s manual
- 版权信息：This document has been placed in the public domain.

## 译注

目前中文社区习惯性将 PyInstaller「整合代码、解释器、依赖项等构建可执行文件」这一过程翻译为「打包」。但其实英文官方文档中称之为捆绑（bundle）或冻结（freeze）。至于构建产物，英文文档中将其称为捆绑包或冻结应用程序。而「包」（package），在 Python 中更多时候是专指[具有 __path__ 属性的 Python 模块](https://docs.python.org/zh-cn/3/glossary.html#term-package)。综上所述，在本套 PyInstaller 文档译文中，暂统一译做「捆绑」与「捆绑包」。
