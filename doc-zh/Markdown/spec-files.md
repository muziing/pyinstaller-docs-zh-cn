# 使用 Spec 文件

> 翻译自 [PyInstaller 文档 v6.3.0 - Using Spec Files](https://pyinstaller.org/en/v6.3.0/spec-files.html)

当运行

```shell
pyinstaller options myscript.py
```

时，PyInstaller 要做的第一件事就是创建一个 spec 文件（specification 的缩写，规范文件）。该文件存储在 `--specpath` 目录（默认为当前目录）中。

Spec 文件告知 PyInstaller 如何处理你的脚本。它编码了脚本名称和大部分传递给 `pyinstaller` 命令的选项。Spec 文件实际上是可执行的 Python 代码，PyInstaller 通过执行其内容来构建应用程序。

对于 PyInstaller 的大部分使用情况，并不需要检查或修改规格文件。通常只需要将所有需要的信息（比如隐式导入）作为选项传递给 `pyinstaller` 命令，然后让它运行即可。

有四种情况需要修改 spec 文件：

- 想要将数据文件与应用程序捆绑时。
- 想要包含运行时库（`.dll` 或 `.so` 文件）时。
- 想要在可执行文件中添加 Python 运行时选项时。
- 想要创建一个多程序捆绑包，其中包含合并的通用模块时。

这些用法将在下文的专题中介绍。

可以通过如下命令创建 spec 文件：

```shell
pyi-makespec options name.py [other scripts ...]
```

其中，*options* 与上文所述的 `pyinstaller` 命令中的 options 选项相同。此命令会创建 *`name.spec`* 文件，但不会继续构建可执行文件。

创建 spec 文件并对其进行必要修改后，将 spec 文件传给 `pyinstaller` 命令即可构建应用程序：

```shell
pyinstaller options name.spec
```

创建 spec 文件时，大多数命令选项都会被编入其中。通过 spec 文件进行构建时，这些选项将无法更改。如果在命令行中再次添加这些选项，则会被忽略，并由 spec 文件中的选项替代。

通过 spec 文件进行构建时，只有以下命令行选项有效：

- `--upx-dir`
- `--distpath`
- `--workpath`
- `--noconfirm`
- `--clean`
- `--log-level`

## Spec 文件选项

在 PyInstaller 创建新的 spec 文件或打开给定的 spec 文件后，`pyinstaller` 命令把 spec 文件作为代码执行。捆绑应用程序就是通过执行 spec 文件创建的。下面是一个简短的示例，用于最小单文件夹应用程序的 spec 文件：

```python
a = Analysis(['minimal.py'],
         pathex=['/Developer/PItests/minimal'],
         binaries=None,
         datas=None,
         hiddenimports=[],
         hookspath=None,
         runtime_hooks=None,
         excludes=None)
pyz = PYZ(a.pure)
exe = EXE(pyz,... )
coll = COLLECT(...)
```

Spec 文件中的语句会创建四个类的实例：`Analysis`、`PYZ`、`EXE` 和 `COLLECT`。

- 一个类  `Analysis` 的新实例将脚本名称列表作为输入。它会分析所有导入和依赖关系。生成的对象（赋值给 `a`）在类成员中包含依赖列表：
  - `scripts`：命令行中指定的 Python 脚本；
  - `pure`：脚本所需的纯 Python 模块；
  - `pathex`：用于搜索导入的路径列表（比如使用 `PYTHONPATH`），包括由 `--paths` 选项给出的路径；
  - `binaries`：脚本所需的非 Python 模块，包括由 `--add-binary` 选项给出的名称；
  - `datas`：应用程序中包含的非二进制文件，包括由 `--add-data` 选项给出的文件名；
- 类 `PYZ` 的实例是一个 `.pyz` 归档文件（在[*检视归档*](./advanced-topics.md#检视归档)中描述），它包含 `a.pure` 中的所有 Python 模块。
- 根据分析的脚本和 `PYZ` 归档创建一个 `EXE` 实例。该对象创建可执行文件。
- 一个 `COLLECT` 的实例，从其他所有部分创建输出文件夹。

在单文件模式下，无需调用 `COLLECT`，而 `EXE` 实例会接收所有脚本、模块和二进制文件。

你可以修改 spec 文件，向 `Analysis` 和 `EXE` 中传递附加值。

## 向软件包添加文件

要将文件添加到捆绑软件包，你需要创建一个描述文件的列表，并将其提供给 `Analysis` 调用。捆绑到单个文件夹时（参阅[*捆绑至单个文件夹*](./operating-mode.md#捆绑至单个文件夹)），添加的数据文件将被复制到包含可执行文件的文件夹中。捆绑到单个可执行文件时（参阅[*捆绑至单个文件*](./operating-mode.md#捆绑至单个文件)），添加文件的副本会被压缩到可执行文件中，并在执行前解压到 `_MEIxxxxxx` 临时文件夹。这意味着，使用单文件模式的可执行程序运行结束后，所有对添加文件做的修改都会丢失。

无论哪种模式，要在运行时查找数据文件，参阅[*运行时信息*](./runtime-information.md#运行时信息)。

### 添加数据文件

你可以使用 `--add-data` 命令选项，或将数据文件作为列表添加到 spec 文件中，从而将它们添加到捆绑包中。

使用 spec 文件时，提供一个文件路径列表作为 `Analysis` 的 `datas=` 参数值。数据文件列表是一个元组列表，每个元组由两个字符串类型的值组成：

- 首个字符串指定当前系统中的一个或多个文件。
- 第二个指定文件在运行时位于的文件夹名称。

例如，要在单文件夹应用程序的顶层添加一个 README 文件，可以对 spec 文件作如下修改：

```python
a = Analysis(...
         datas=[ ('src/README.txt', '.') ],
         ...
         )
```

与之对应的命令行：

```shell
pyinstaller --add-data "src/README.txt:." myscript.py
```

这里把只有一个条目的列表作为 `datas=` 的参数。它是一个元组，其中第一个字符串表示现有文件为 `src/README.txt`。该文件将被查找（相对路径从 spec 文件所在位置开始计算）并复制到捆绑应用程序的顶层。

字符串中可以使用 `/` 或 `\` 作为路径分隔符。可以使用通配符 `*` 指定输入文件。例如，包含某个文件夹中所有的 `.mp3` 文件：

```python
a = Analysis(...
         datas= [ ('/mygame/sfx/*.mp3', 'sfx' ) ],
         ...
         )
```

文件夹 `/mygame/sfx` 中的所有 `.mp3` 文件将被复制到捆绑程序中名为 `sfx` 的文件夹中。

如果在单独的语句中创建待添加文件列表，spec 文件的可读性会更好一些：

```python
added_files = [
         ( 'src/README.txt', '.' ),
         ( '/mygame/sfx/*.mp3', 'sfx' )
         ]
a = Analysis(...
         datas = added_files,
         ...
         )
```

也可以包含文件夹中的全部内容：

```python
added_files = [
         ( 'src/README.txt', '.' ),
         ( '/mygame/data', 'data' ),
         ( '/mygame/sfx/*.mp3', 'sfx' )
         ]
```

文件夹 `/mygame/data` 将以名称 `name` 复制到捆绑包中。

### 使用模块中的数据文件

如果想要添加的数据文件包含在一个 Python 模块中，可以使用 `pkgutil.get_data()` 来获取它们。

例如，假设应用程序的组成部分之一是一个名为 `helpmod` 的模块。在脚本及其 spec 文件所在的文件夹中，有这样的文件树：

```shell
$ tree helpmod
helpmod
├── __init__.py
├── helpmod.py
└── help_data.txt
```

由于脚本中包含了 `import helpmod` 语句，PyInstaller 将在捆绑的应用程序中创建该文件夹。但是它只会包含 `.py` 文件。数据文件 `help_data.txt` 不会被自动包含。要使它也被包含，需要在 spec 文件中添加一个 `datas` 元组：

```python
a = Analysis(...
         datas= [('helpmod/help_data.txt', 'helpmod' )],
         ...
         )
```

脚本执行时，可以按上一节所述，通过其基文件夹路径找到 `help_data.txt`。不过，该数据文件是模块的一部分，因此也可以使用标准库中的函数 [`pkgutil.get_data()`](https://docs.python.org/zh-cn/3/library/pkgutil.html#pkgutil.get_data) 来获取其内容：

```python
import pkgutil
help_bin = pkgutil.get_data('helpmod', 'help_data.txt')
```

这将以二进制串的形式返回 `help_data.txt` 文件的内容。如果原本是字符，则需要解码：

```python
help_utf = help_bin.decode('UTF-8', 'ignore')
```

### 添加二进制文件

> Note
>
> 二进制文件指的是 DLL、动态链接库、共享对象文件等，PyInstaller 将搜索这些文件以进一步查找二进制依赖。像图像和 PDF 这样的文件应该放到 `datas` 中。

为实现把二进制文件添加到捆绑包中，可以使用 `--add-binary` 命令选项，或将其作为列表添加至 spec 文件。在 spec 文件中，构建一个描述所需文件的元组列表，然后把这个元组列表赋值给 Analysis 的 `binaries=` 参数。

添加二进制文件的方法与添加数据文件类似。每个元组应包含两个值：

- 第一个字符串指定当前系统中的一个或多个文件。
- 第二个指定文件在运行时位于的文件夹名称。

通常情况下，PyInstaller 通过分析导入的模块来理解 `.so` 和 `.dll` 库的情况。但有时候，某个模块是否被导入并不明确，这种情况下可以使用 `--hidden-import` 命令选项。但即便如此，还可能无法找到所有的依赖项。

假设有一个用 C 语言编写、并使用了 Python C-API 的模块 `special_ops.so`。程序中导入了 `special_ops`，PyInstaller 找到并包含了 `special_ops.so`。但也许 `special_ops.so` 链接到了 `libiodbc.2.dylib`，PyInstaller 找不到这个依赖关系。可以这样把它添加到捆绑中：

```python
a = Analysis(...
         binaries=[ ( '/usr/lib/libiodbc.2.dylib', '.' ) ],
         ...
```

或者通过命令行：

```shell
pyinstaller --add-binary "/usr/lib/libiodbc.2.dylib:." myscript.py
```

如果希望将 `libiodbc.2.dylib` 存储在捆绑包内的某个特定文件夹（比如 `vendor`）中，则可以使用元组的第二个元素来指定：

```python
a = Analysis(...
         binaries=[ ( '/usr/lib/libiodbc.2.dylib', 'vendor' ) ],
         ...
)
```

与数据文件一样，如果要添加多个二进制文件，为提高可读性，可以在单独的语句中创建列表，并将列表名称传递。

### 添加文件的高级方法

PyInstaller 支持一种更高级（也更复杂）的将文件添加到捆绑包的方法，可能对特殊情况有用。参阅[*目录（TOC）和 Tree 类*](./advanced-topics.md#目录（TOC）和-tree-类)。

## 指定 Python 解释器选项

PyInstaller 打包后的应用程序在独立的嵌入式 Python 解释器中运行程序代码。因此，**向 Python 解释器传递选项的典型方法并不适用**，包括：

- [环境变量](https://docs.python.org/zh-cn/3/using/cmdline.html#environment-variables)（例如 *`PYTHONUTF8`* 与 *`PYTHONHASHSEED`*） - 因为打包的应用程序应与目标系统中可能存在的 Python 环境隔离开来。
- [命令行参数](https://docs.python.org/zh-cn/3/using/cmdline.html#miscellaneous-options)（例如 *`-v`* 与 *`-O`*） - 因为命令行参数是为应用程序保留的。

替代性的，PyInstaller 提供了一个选项，通过它自己的 `OPTIONS` 机制，为应用程序的 Python 解释器指定永久的运行时选项。要传递运行时选项，需要创建一个三元素元组列表：*('选项字符串', None, 'OPTION')*，并将其作为附加参数传递给 EXE，置于关键字参数之前。选项元组的第一个元素是选项字符串（有效选项见下文），第二个元素总是 *None*，第三个总是 *'OPTION'*。

一个 spec 文件示例，指定了两个运行时选项：

```python
options = [
    ('v', None, 'OPTION'),
    ('W ignore', None, 'OPTION'),
]

a = Analysis(
    ...
)
...
exe = EXE(
    pyz,
    a.scripts,
    options,  # <-- 这里把选项列表传递给了 EXE
    exclude_binaries=...
    ...
)
```

该机制支持以下选项：

- `'v'` 或 `'verbose'`：增加 `sys.flags.verbose` 的值，这会导致每个模块初始化时都向 stdout 中写入信息。该选项等同于 Python 的 `-v` 命令行选项。当通过 PyInstaller 自带的 `--debug imports` 选项启用 [verbose 导入](./when-things-go-wrong.md#获取-python-的详尽导入信息)时，它将自动启用。
- `'u'` 或 `'unbuffered'`：启用无缓冲的 stdout 与 stderr。相当于 Python 的 `-u` 命令行选项。
- `'O'` 或 `'optimize'`：增加 `sys.flags.optimize` 的值，相当于 Python 的 `-O` 命令行选项。
- `'W <arg>'`：传递 [Python 的 W 选项](https://docs.python.org/zh-cn/3/using/cmdline.html#cmdoption-W)，控制警告信息。
- `'X <arg>'`：传递 [Python 的 X 选项](https://docs.python.org/zh-cn/3/using/cmdline.html#cmdoption-X)。其中用于控制 UFT-8 模式和开发者模式的 `utf8` 和 `dev` X 选项将由 PyInstaller 的 bootloader 明确解析，并在解释器预初始化时使用；其余的 X 选项只是传递给解释器配置。
- `'hash_seed=<value>'`：一个用于将打包应用程序中的 Python 哈希种子设置为固定值的选项。相当于 `PYTHONHASHSEED` 环境变量。在撰写本文时，该选项还没有作为 X 选项存在，因此作为自定义选项实现。

进一步举例说明语法：

```python
options = [
    # 警告控制
    ('W ignore', None, 'OPTION'),  # 禁用所有警告
    ('W ignore::DeprecationWarning', None, 'OPTION'),  # 禁用已弃用的警告

    # UTF-8 模式；除非明确启用/禁用，否则会根据本地语言 locale 自动启用
    ('X utf8', None, 'OPTION'),  # 强制启用 UTF-8 模式 on
    ('X utf8=1', None, 'OPTION'),  # 强制启用 UTF-8 模式 on
    ('X utf8=0', None, 'OPTION'),  # 强制禁用 UTF-8 模式

    # 开发者模式；默认禁用
    ('X dev', None, 'OPTION'),  # 启用开发者模式
    ('X dev=1', None, 'OPTION'),  # 启用开发者模式

    # 哈希种子
    ('hash_seed=0', None, 'OPTION'),  # 禁用哈希随机化；sys.flags.hash_randomization=0
    ('hash_seed=123', None, 'OPTION'),  # 具有固定种子的哈希随机化
]
```

## 用于 macOS 捆绑的 spec 文件选项

当构建一个窗口化的 macOS 应用程序（即，在 macOS 下运行，指定了 `--windowed` 选项）时，spec 文件包含一个额外的语句来创建 macOS 应用程序捆绑包或应用程序文件夹：

```python
app = BUNDLE(exe,
         name='myscript.app',
         icon=None,
         bundle_identifier=None)
```

`BUNDLE` 的 `icon=` 参数将使用由 `--icon` 选项指定的图标文件路径。`bundle_identifier` 将包含通过 `--osx-bundle-identifier` 选项指定的值。

`Info.plist` 文件是 macOS 捆绑应用程序包的重要组成部分。（关于 `Info.plist` 内容的讨论，参阅  [Apple bundle overview](https://developer.apple.com/library/mac/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html)。）

PyInstaller 会创建一个最小 `Indo.plist`。`version` 选项可用于设置使用 CFBundleShortVersionString Core Foundation Key 的应用程序的版本。

通过向 BUNDLE 调用传递一个 `info_plist=` 参数，可以添加或覆盖 plist 中的条目。其参数应为一个 Python 字典，由要包含在 `Info.plist` 文件中的键值组成。PyInstaller 使用 Python 标准库模块 [plistlib](https://docs.python.org/zh-cn/3/library/plistlib.html) 来从 info_plist 字典创建 `Info.plist`。plistlib 可以处理嵌套的 Python 对象（转换为嵌套的 XML），并将 Python 数据类型转换为适当的 `Info.plist` XML 类型。下面是一个例子：

```python
app = BUNDLE(exe,
         name='myscript.app',
         icon=None,
         bundle_identifier=None,
         version='0.0.1',
         info_plist={
            'NSPrincipalClass': 'NSApplication',
            'NSAppleScriptEnabled': False,
            'CFBundleDocumentTypes': [
                {
                    'CFBundleTypeName': 'My File Format',
                    'CFBundleTypeIconFile': 'MyFileIcon.icns',
                    'LSItemContentTypes': ['com.example.myformat'],
                    'LSHandlerRank': 'Owner'
                    }
                ]
            },
         )
```

在上面的例子中，键值对  `'NSPrincipalClass': 'NSApplication'` 是允许 macOS 使用视网膜分辨率渲染应用程序所必须的。键 `'NSAppleScriptEnabled'` 被赋值为 Python 布尔类型 `False`，将以 `<false/>` 的形式正确输出到 `Info.plist` 中。最后，键 `CFBundleDocumentTypes` 告知 macOS 应用程序支持哪些文件类型（参阅 [Apple document types](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/20001431-101685)）。

## POSIX 专有选项

默认情况下，所有用到的系统库都会被捆绑。要在捆绑时排除所有或大部分非 Python 共享系统库，可以在 Analysis 类中添加对函数 `exclude_system_libraries` 的调用。对 POSIX 及其相关操作系统，系统库是指在 `/lib*` 或 `usr/lib*` 下的文件。该函数接受一个可选参数——例外文件通配符列表，用于在捆绑时不排除与这些通配符匹配的库文件。例如，要排除几乎所有非 Python 系统库，只保留 "libexpat"  以及名称中包含 "krb" 的系统库，这样做：

```python
a = Analysis(...)

a.exclude_system_libraries(list_of_exceptions=['libexpat*', '*krb*'])
```

## `Splash` 目标

For a splash screen to be displayed by the bootloader, the `Splash` target must be called at build time. This class can be added when the spec file is created with the command-line option `--splash IMAGE_FILE <--splash>`. By default, the option to display the optional text is disabled (`text_pos=None`). For more information about the splash screen, see [闪屏（*实验性功能*）](./usage.md#闪屏实验性功能) section. The `Splash` Target looks like this:

```python
a = Analysis(...)

splash = Splash('image.png',
                binaries=a.binaries,
                datas=a.datas,
                text_pos=(10, 50),
                text_size=12,
                text_color='black')
```

Splash bundles the required resources for the splash screen into a file,
which will be included in the CArchive.

A `Splash` has two outputs, one is itself and one is stored in `splash.binaries`. Both need to be passed on to other build targets in order to enable the splash screen. To use the splash screen in a **onefile** application, please follow this example:

```python
a = Analysis(...)

splash = Splash(...)

# onefile
exe = EXE(pyz,
          a.scripts,
          splash,                   # <-- both, splash target
          splash.binaries,          # <-- and splash binaries
          ...)
```

In order to use the splash screen in a **onedir** application, only a small change needs to be made. The `splash.binaries` attribute has to be moved into the `COLLECT` target, since the splash binaries do not need to be included into the executable:

```python
a = Analysis(...)

splash = Splash(...)

# onedir
exe = EXE(pyz,
          splash,                   # <-- splash target
          a.scripts,
          ...)
coll = COLLECT(exe,
               splash.binaries,     # <-- splash binaries
               ...)
```

On Windows/macOS images with per-pixel transparency are supported. This allows non-rectangular splash screen images. On Windows the transparent borders of the image are hard-cuted, meaning that fading transparent values are not supported. There is no common implementation for non-rectangular windows on Linux, so images with per-pixel transparency is not supported.

The splash target can be configured in various ways. The constructor of the `Splash` target is as follows:

<https://pyinstaller.org/en/v6.3.0/spec-files.html#PyInstaller.building.splash.Splash.__init__>

## 多软件包捆绑

有些产品由多个不同的应用程序组成，每个应用程序都可能依赖于一套共同的第三方库，或以其他方式共享代码。在打包此类产品时，如果孤立地处理每个应用程序，把它和所有依赖捆绑在一起，那就太可惜了，因为这意味着要存储重复的代码和库的副本。

可以使用多软件包功能捆绑一组可执行程序，使它们共享单一的库副本。在单文件或单文件夹应用程序中都可以使用这一功能。

### 单文件夹应用程序中的多软件包捆绑

要合并多个单文件夹应用程序，使用[共享 COLLECT 语句](https://www.zacoding.com/en/post/pyinstaller-create-multiple-executables/)。这将把所有单文件夹应用程序的外部资源收集到同一个目录中。

### 单文件应用程序中的多软件包捆绑

每个依赖项（比如一个 DLL 文件）只打包一次，放在其中一个应用程序中。程序集中其他依赖该 DLL 的应用程序都有一个 "外部引用"，告诉它们从包含该 DLL 的应用程序的可执行文件中提取该依赖项。

这样可以节省磁盘空间，因为每个依赖项只存储一次。不过，在应用程序启动时，跟踪外部引用需要额外的时间。除了那个应用程序外，其他所有应用程序的启动时间都会稍慢一些。

二进制文件之间的外部引用包含了输出目录的硬编码路径，不能重新排列。安装应用程序时，必须把所有相关应用程序放在同一目录下。

要创建这样一组应用程序，必须编写一个自定义的 spec 文件，其中包含对 `MERGE` 函数的调用。该函数接收要分析的脚本列表，查找它们的共同依赖关系，并修改分析结果以尽可能减少存储开销。

参数列表中分析对象的顺序很重要。MERGE 函数会把每个依赖项打包到从左到右第一个需要该依赖项的脚本中。列表中排在后面的脚本如果需要相同的文件，就会在外部引用中加上排在列表前面的脚本。你可以对脚本进行排序，将最常用的脚本放在列表的最前面。

适用于多软件包捆绑的 spec 文件包含对 MERGE 函数的一次调用：

```python
MERGE(*args)
```

MERGE 在 Analysis 分析阶段之后、`EXE` 之前使用。其参数为任意长度的元组列表，每个元组包含三个元素：

- 第一个元素是一个 Analysis 对象，即 Analysis 类的一个实例，应用于其中一个应用程序。
- 第二个元素是要分析的应用程序脚本名称（不含 `.py` 扩展名）。
- 第三个元素是可执行文件的名称（通常与脚本相同）。

MERGE 会检查分析对象以了解每个脚本的依赖关系。它会修改这些对象，避免库和模块的重复。最终，生成的软件包将相互连接。

### 示例 MERGE spec 文件

为多软件包捆绑构建一个 spec 文件的方法之一是，首先为软件包中的每个应用程序构建 spec 文件。假设你有一个由三个应用程序组成的产品，它们被（缺乏新意地）命名为 `foo`、`bar` 和 `zap`：

```shell
pyi-makespec 适当的选项... foo.py
```

```shell
pyi-makespec 适当的选项... bar.py
```

```shell
pyi-makespec 适当的选项... zap.py
```

检查警告信息，并逐一测试每个应用程序，处理所有隐式导入和其他问题。当三个应用程序都能正常运行时，将 `foo.spec`、`bar.spec` 和 `zap.spec` 三个文件中的语句合并如下：

首先复制每个 Analysis 语句并进行修改，给每个 Analysis 对象一个唯一的名称：

```python
foo_a = Analysis(['foo.py'],
        pathex=['/the/path/to/foo'],
        hiddenimports=[],
        hookspath=None)

bar_a = Analysis(['bar.py'], etc., etc...)

zap_a = Analysis(['zap.py'], etc., etc...)
```

然后调用 MEGRE 方法处理这三个 Analysis 对象：

```python
MERGE((foo_a, 'foo', 'foo'), (bar_a, 'bar', 'bar'), (zap_a, 'zap', 'zap'))
```

Analysis 对象 `foo_a`、`bar_a` 和 `zap_a` 会被修改，使得后两个对象引用第一个对象中的共同依赖。

随后，可以从原始的三个 spec 文件中复制 `PYZ`、`EXE` 和 `COLLECT` 语句，把原始 spec 文件中使用 `a.` 的地方替换为 Analysis 对象的唯一名称。修改 EXE 语句，除了在原始 EXE 语句中传递的所有其他参数外，还要传递 `Analysis.dependencies` 参数。例如：

```python
foo_pyz = PYZ(foo_a.pure)
foo_exe = EXE(foo_pyz, foo_a.dependencies, foo_a.scripts, ... etc.)

bar_pyz = PYZ(bar_a.pure)
bar_exe = EXE(bar_pyz, bar_a.dependencies, bar_a.scripts, ... etc.)
```

将合并后的 spec 文件保存为 `foobarzap.spec`，然后构建它：

```shell
pyinstaller foobarzap.spec
```

输出至 `dist` 文件夹的将是全部三个应用程序，但应用程序 `dist/bar` 和 `dist/zap` 将引用 `dist/foo` 中的内容以共享依赖关系。

记住，spec 文件是可执行的 Python。在创建 Analysis 对象和执行 `PYZ`、`EXE`、`COLLECT` 语句时，可以使用所有的 Python 工具（`for`、`with` 以及标准库 `sys` 和 `io` 的成员）。你可能还需要了解[*目录（TOC）和 Tree 类*](./advanced-topics.md#目录（TOC）和-tree-类)。

## Spec 文件中可用的全局变量

在 spec 文件执行时，它可以访问一组有限的全局名称。这些名称包含 PyInstaller 定义的类：`Analysis`、`BUNDLE`、`COLLECT`、`EXE`、`MERGE`、`PYZ`、`TOC`、`Tree` 和 `Splash`，这些已在前面的章节中有过讨论。

其他包含构建环境信息的全局变量：

`DISTPATH`
    指向 `dist` 文件夹（即，构建捆绑生成的应用程序存储位置）的相对路径。默认路径为当前目录的相对路径。如果使用了 `--distpath` 选项，则 `DISTPATH` 包含该值。

`HOMEPATH`
    PyInstaller 发行版的绝对路径，通常位于当前 Python 的 site-packages 文件夹中。

`SPEC`
    为 `pyinstaller` 命令提供的完整的 spec 文件参数，例如 `myscript.spec` 或 `source/myscript.spec`。

`SPECPATH`
    `SPEC` 值的路径前缀，由 `os.path.split()` 返回。

`specnm`
    Spec 文件的名称，例如 `myscript`。

`workpath`
    `build` 目录的路径。默认为相对当前路径。如果使用了 `workpath=` 选项，则 `workpath` 包含该值。

`WARNFILE`
    构建目录中警告文件的完整路径，例如 `build/warn-myscript.txt`。

## 在 spec 文件中添加参数

有时，可能希望通过同一个 spec 文件实现不同的构建模式（比如 *debug* 调试模式和 *production* 发布模式）。PyInstaller 不会解析出现在 `--` 分隔符后面的任何传递给 `pyinstaller` 的命令行参数，而会把这些参数转发到 spec 文件中。在 spec 文件中你可以实现自己的参数解析，并相应地处理选项。

例如，对于下面的 spec 文件，如果通过 `pyinstaller example.spec -- --debug` 调用，将创建一个启用类控制台的单目录应用程序；如果只使用 `pyinstaller example.spec`，则将一个单文件无控制台应用程序。如果你使用基于 `argparse` 的解析器，而不是使用 `sys.argv` 自行创建解析器，那么 `pyinstaller example.spec --help` 将显示你的 spec 选项。

```python
# example.spec

import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--debug", action="store_true")
options = parser.parse_args()

a = Analysis(
    ['example.py'],
)
pyz = PYZ(a.pure)

if options.debug:
    exe = EXE(
        pyz,
        a.scripts,
        exclude_binaries=True,
        name='example',
    )
    coll = COLLECT(
        exe,
        a.binaries,
        a.datas,
        name='example_debug',
    )
else:
    exe = EXE(
        pyz,
        a.scripts,
        a.binaries,
        a.datas,
        name='example',
        console=False,
    )
```
