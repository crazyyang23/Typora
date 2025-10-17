# echo

输出指定的字符串或者变量

## 补充说明

**echo命令** 用于在shell中打印shell变量的值，或者直接输出指定的字符串。linux的echo命令，在shell编程中极为常用, 在终端下打印变量value的时候也是常常用到的，因此有必要了解下echo的用法echo命令的功能是在显示器上显示一段文字，一般起到一个提示的作用。

### 语法

```shell
echo(选项)(参数)
```

### 选项

```shell
-e：启用转义字符。
-E: 不启用转义字符（默认）
-n: 结尾不换行
```

使用`-e`选项时，若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出：

- `\a` 发出警告声；
- `\b` 删除前一个字符；
- `\c` 不产生进一步输出 (\c 后面的字符不会输出)；
- `\f` 换行但光标仍旧停留在原来的位置；
- `\n` 换行且光标移至行首；
- `\r` 光标移至行首，但不换行；
- `\t` 插入tab；
- `\v` 与\f相同；
- `\\` 插入\字符；
- `\nnn` 插入 `nnn`（八进制）所代表的ASCII字符；

### 参数

变量：指定要打印的变量。

### 实例

```shell
/bin/echo Hello, world!
```

在上面的命令中，两个词（Hello 和 world！）作为单独的参数传递给 echo，并且 echo 按顺序打印它们，用空格分隔

下一个命令产生相同的输出：

```shell
/bin/echo 'Hello, World!'
```

但是，与第一个示例不同，上述命令提供了单引号字符串 'Hello, world!' 作为一个单一的一个参数。

单引号将可靠地保护它免受 shell 解释，将特殊字符和转义序列逐字传递给 echo。

例如，在 `bash shell` 中，变量名前面有一个美元符号 ($)。 在下一个命令中，引号内的变量名按字面意思处理； 在引号之外，它被转换为它的值。

```shell
/bin/echo 'The value of $PATH is' $PATH
# The value of $PATH is
# /home/hope/bin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

用echo命令打印带有色彩的文字：

**文字色：**

```shell
echo -e "\e[1;31mThis is red text\e[0m"
This is red text
```

- `\e[1;31m` 将颜色设置为红色
- `\e[0m` 将颜色重新置回

颜色码：重置=0，黑色=30，红色=31，绿色=32，黄色=33，蓝色=34，洋红=35，青色=36，白色=37

```shell
echo -e "\x1b[30;1m 0 黑色 \x1b[0m"\
"\x1b[31;1m 1 红色 \x1b[0m"\
"\x1b[32;1m 2 绿色 \x1b[0m"\
"\x1b[33;1m 3 黄色 \x1b[0m"\
"\x1b[34;1m 4 蓝色 \x1b[0m"\
"\x1b[35;1m 5 洋红 \x1b[0m"\
"\x1b[36;1m 6 青色 \x1b[0m"\
"\x1b[37;1m 7 白色 \x1b[0m"
```

**背景色** ：

```shell
echo -e "\e[1;42mGreed Background\e[0m"
Greed Background
```

颜色码：重置=0，黑色=40，红色=41，绿色=42，黄色=43，蓝色=44，洋红=45，青色=46，白色=47

**文字闪动：**

```shell
echo -e "\033[37;31;5mMySQL Server Stop...\033[39;49;0m"
```

红色数字处还有其他数字参数：0 关闭所有属性、1 设置高亮度（加粗）、4 下划线、5 闪烁、7 反显、8 消隐

**输出内容结尾不添加换行符**

```shell
echo -n 'hello'
```

# find

在指定目录下查找文件

## 解释

从每个指定的起始点 (目录) 开始，搜索以该点为根的目录树，并按照运算符优先级规则**从左至右**评估给定的表达式，直到结果确定，此时`find`会继续处理下一个文件名。

## 补充说明

本文列出的选项指的是**表达式列表中的选项**。这些选项控制了`find`的行为，需在**最后一个路径名之后**立即指定。

五个真实选项: `-H、-L、-P、-D 和 -O`。如果出现，**必须位于第一个路径名之前**。关于这部分内容本文不做描述，具体内容可参考[man7.org中的find](https://man7.org/linux/man-pages/man1/find.1.html#top_of_page)

如果使用该命令时，不设置任何参数，则`find`命令将在当前目录下查找子目录与文件，并且将查找到的子目录和文件全部进行显示。等效于以下命令:

```shell
find . -print
```

## 语法

```shell
find [-H] [-L] [-P] [-D debugopts] [-Olevel] [起始点...] [表达式]
```

忽略真实选项后 (更为常见):

```shell
find [起始点...] [表达式]
```

## 表达式分类

起始点（列表）之后的部分是表达式。这是一种**查询规范**，描述了我们如何匹配文件（返回**真**或者**假**）以及对匹配到的文件进行何种操作。表达式由一系列元素组成：

- 测试（Tests）：测试返回一个真或假值，通常基于我们正在考虑的文件的某个属性。例如，`-empty`测试仅在当前文件为空时为真。
- 操作（Actions）：操作具有副作用（例如在标准输出上打印内容），并返回真或假，通常基于它们是否成功。例如，`-print`操作会在标准输出上打印当前文件的名称。
- 全局（Global）：全局选项影响命令行中任何部分指定的测试和操作的执行。全局选项始终返回真值。例如，`-depth`选项使find以深度优先的顺序遍历文件系统。
- 位置（Positional）：位置选项仅影响其后的测试或操作。位置选项始终返回真值。例如，`-regextype`选项是位置选项，用于指定命令行中后续正则表达式所使用的正则表达式方言。
- 操作符（Operators）：运算符将表达式中的其他项连接起来。例如，它们包括`-o`（表示逻辑或）和`-a`（表示逻辑与）。如果缺少运算符，则默认使用`-a`。

## 表达式选项

### 测试选项

```shell
-amin<分钟>：查找在指定时间曾被存取过的文件或目录，单位以分钟计算；
-anewer<参考文件或目录>：查找其存取时间较指定文件或目录的存取时间更接近现在的文件或目录；
-atime<24小时数>：查找在指定时间曾被存取过的文件或目录，单位以24小时计算；
-cmin<分钟>：查找在指定时间之时被更改过的文件或目录；
-cnewer<参考文件或目录>查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录；
-ctime<24小时数>：查找在指定时间之时被更改的文件或目录，单位以24小时计算；
-empty：寻找文件大小为0 Byte的文件，或目录下没有任何子目录或文件的空目录；
-executable 匹配当前用户可执行的文件和可搜索的目录。
-false：将find指令的回传值皆设为False；
-fstype<文件系统类型>：只寻找该文件系统类型下的文件或目录；
-gid<群组识别码>：查找符合指定之群组识别码的文件或目录；
-group<群组名称>：查找符合指定之群组名称的文件或目录；
-ilname<范本样式>：此参数的效果和指定“-lname”参数类似，但忽略字符大小写的差别；
-iname<范本样式>：此参数的效果和指定“-name”参数类似，但忽略字符大小写的差别；
-inum<inode编号>：查找符合指定的inode编号的文件或目录；
-ipath<范本样式>：此参数的效果和指定“-path”参数类似，但忽略字符大小写的差别；
-iregex<范本样式>：此参数的效果和指定“-regexe”参数类似，但忽略字符大小写的差别；
-iwholename 模式参见`-ipath`。此选项的可移植性较`-ipath`差。
-links<连接数目>：查找符合指定的硬连接数目的文件或目录；
-lname<范本样式>：指定字符串作为寻找符号连接的范本样式；
-mmin<分钟>：查找在指定时间曾被更改过的文件或目录，单位以分钟计算；
-mtime<24小时数>：查找在指定时间曾被更改过的文件或目录，单位以24小时计算；
-name<范本样式>：指定字符串作为寻找文件或目录的范本样式；
-newer<参考文件或目录>：查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录；
-newerXY<引用>：如果正在考虑的文件的时间戳 X 比文件引用的时间戳 Y 更新则成功。
-nogroup：找出不属于本地主机群组识别码的文件或目录；
-nouser：找出不属于本地主机用户识别码的文件或目录；
-path<范本样式>：指定字符串作为寻找目录的范本样式；
-perm<权限数值>：查找符合指定的权限数值的文件或目录；
-readable：匹配当前用户可读的文件
-regex<范本样式>：指定字符串作为寻找文件或目录的范本样式；
-samefile 名称 文件与名称指向相同的 inode。
-size<文件大小>：查找符合指定的文件大小的文件；
-type<文件类型>：只寻找符合指定的文件类型的文件；
-uid<用户识别码>：查找符合指定的用户识别码的文件或目录；
-used<日数>：查找文件或目录被更改之后在指定时间曾被存取过的文件或目录，单位以日计算；
-user<拥有者名称>：查找符和指定的拥有者名称的文件或目录；
-writable：匹配当前用户可写入的文件。
-xtype<文件类型>：此参数的效果和指定“-type”参数类似，差别在于它针对符号连接检查。
-context<表达式>：仅限 SELinux。文件的安全上下文与全局模式匹配
```

### 操作选项

#### -delete 删除文件或目录。

> ⚠️警告：find 命令会将命令行作为表达式进行解析，因此将`-delete`放在首位会将指定的起始点下的**所有内容删除**。且`-delete`操作无法删除一个目录，除非它是空的。

##### *无参数*

##### 描述

如果删除成功则返回真。若删除失败，将显示错误消息，并且 find 最终退出时的状态码将为非零。

##### 相关选项

- **-depth**：在命令行中使用`-delete`操作会自动启用`-depth`选项。为了避免意外情况，通常最好在早期的**Tests选项**中**明确使用**`-depth`选项。
- **-prune**：由于`-depth`会使`-prune`失效，因此`-delete`操作无法与`-prune`有效结合使用。通常，用户可能希望在实际删除操作前，先用带有`-print`的查找命令行进行测试，以确保在添加`-delete`进行实际删除时不会出现意外结果。
- **-ignore_readdir_race**：`-delete`与此选项一起使用时，find 会忽略自父目录读取以来文件已消失的情况下`-delete`操作的错误：它不会输出错误诊断，不会将退出代码更改为非零，并且`-delete`操作的返回代码将为真。

#### -exec 执行命令

> ⚠️警告：使用`-exec`操作存在不可避免的安全问题，应改用`-execdir`选项。

##### 参数

```
command ;` 或 `command {} +
```

##### 描述

如果返回状态为 0，则结果为真。**注意**：find 命令会将**所有后续参数**视为`command`的参数，直到遇到包含`;`的参数为止。字符串`{}`会在`command`的参数中所有出现的位置被替换为当前正在处理的文件名，而不仅仅是在它单独出现的参数中，这与某些版本的 find 不同。这两种结构可能需要使用反斜杠`\`或引号来转义，以防止被 shell 扩展。指定的命令会为每个匹配的文件运行一次。命令在起始目录中执行。

#### -execdir 在包含匹配文件的子目录中执行命令

##### 参数

```
command ;` | `command {} +
```

##### 描述

类似于`-exec`，但指定的`command`会在包含匹配文件的**子目录中运行**，而非find的起始点目录。与`-exec`一样，如果从shell调用find，`{}`应加引号。这是一种更安全的调用`command`方式，因为它避免了在解析匹配文件路径时出现的竞争条件。与`-exec`操作类似，`+`形式的`-execdir`会构建一个命令行来处理多个匹配文件，但任何给定的`command`调用只会列出存在于同一子目录中的文件。如果使用此选项，必须确保 PATH 环境变量未引用`.`，否则攻击者可以通过在您将运行`-execdir`的目录中留下一个适当命名的文件来运行任何命令。同样，PATH 中的条目**不应为空**或**非绝对目录名**。如果使用`+`形式的任何调用以非零值作为退出状态返回，则 find 也会返回非零退出状态。如果 find 遇到错误，有时会导致立即退出，**因此某些待处理的command可能根本不会运行**。 操作结果取决于使用的是`+`还是`;`变体。`-execdir command {} + `总是返回真，而 `-execdir command {} ;`仅在命令返回 0 时返回真。

#### -fls 创建文件并将结果写入文件

##### 参数

```
file
```

##### 描述

此选项始终返回真。`-fls`类似于`-ls`和`-fprint`，但`-fls`会将结果写入文件中。无论谓词是否匹配，输出文件始终会被创建。有关文件名中特殊字符处理的信息，请参阅“特殊文件名处理”部分。

#### -fprint 将完整文件名打印到指定文件中

##### 参数

```
file
```

##### 描述

此选项始终返回真。若运行 find 时`file`不存在，则创建该`file`；若`file`已存在，则截断其内容。文件名`/dev/stdout`和`/dev/stderr`有特殊处理，分别指向标准输出和标准错误输出。即使谓词从未匹配，输出文件也会始终创建。

#### -fprint0

##### 参数

```
file
```

##### 描述

此选项始终返回真。类似于`-print0`，但将输出写入文件；类似于`-fprint`。即使谓词从未匹配，输出文件也始终会被创建。

#### -fprintf

##### 参数

```
file
```

##### 描述

此选项始终返回真。类似于`-printf`，但将输出写入文件；类似于`-fprint`，即使谓词从未匹配，输出文件也会始终创建。

#### -ls 列出当前文件并输出到标准输出

##### *无参数*

##### 描述

此选项始终返回真。以`ls -dils`格式列出当前文件并输出到标准输出。块计数为 1 KB 块，除非设置了环境变量 POSIXLY_CORRECT，此时使用 512 字节块。

#### -ok 执行命令前询问用户

##### 参数

```
command ;
```

##### 描述

类似于`-exec`，但首先会询问用户。如果用户同意，则运行该命令；否则仅返回 false。若运行该命令，其标准输入将被重定向至`/dev/null`。对提示的响应会与一对正则表达式进行匹配，以确定其为肯定或否定回答。若设置POSIXLY_CORRECT 环境变量，则该正则表达式从系统获取；否则，从 find 的消息翻译中获取。如果系统没有合适的定义，将使用 find 自身的定义。无论哪种情况，正则表达式本身的解释都会受到环境变量 LC_CTYPE（字符类）和 LC_COLLATE（字符范围和等价类）的影响。

##### 相关选项

- **-files0-from**：不能与`-ok`同时指定。

#### -okdir

##### 参数

```
command ;
```

##### 描述

类似于`-execdir`，但在执行前会以与`-ok`相同的方式询问用户。如果用户不同意，则直接返回 false。如果命令被执行，其标准输入将从`/dev/null`重定向。

##### 相关选项

- **-files0-from**：不能与`-okdir`同时指定。

#### -print 打印完整文件名，后跟一个换行符

##### *无参数*

##### 描述

此选项始终返回真。如果你将 find 的输出通过管道传输到另一个程序，并且你正在搜索的文件可能包含换行符，那么应该考虑使用`-print0`而不是`-print`。

#### -print0 打印完整文件名，后跟一个空字符

##### *无参数*

##### 描述

此选项始终返回真。包含换行符或其他类型空白字符的文件名能被正确解析，以便处理 find 输出的程序能正确理解。此选项对应于`xargs`的`-0`选项。

#### -printf 打印格式

##### 参数

```
format
```

可用的转义字符和指令包括：

- \a 警报。
- \b 退格键。
- \c 立即停止打印并清空输出。
- \f 换页。
- \n 换行。
- \r 回车符。
- \t 水平制表符。
- \v 垂直制表符。
- \0 空字符。
- \\ 一个字面的反斜杠`\`。
- \NNN 字符，其 ASCII 码为 NNN（八进制）。
- A 一个反斜杠字符`\`后跟任何其他字符，都会被视为普通字符，因此它们都会被打印出来。
- %% 一个字面的百分号。
- %a 文件的最后访问时间，格式为 C 语言 ctime(3)函数返回的样式。 .....更多内容待补充

##### 描述

*暂无*

#### -prune 如果文件是目录，则不进入该目录

##### *无参数*

##### 描述

此选项始终返回真。

##### 相关选项

- **-depth**：如果指定了`-depth`，那么`-prune`将无效。
- **-delete**：因为`-delete`隐含了`-depth`，所以不能有效地同时使用两者。

#### -quit 立即退出

##### *无参数*

##### 描述

如果没有发生错误，则返回值为零。这与`-prune `不同，因为`-prune`仅适用于被修剪目录的内容，而`-quit`则使 find 立即停止。不会有任何子进程继续运行。在程序退出之前，任何通过`-exec ... +`或`-execdir ... +`构建的命令行都会被调用。执行`-quit`后，命令行中指定的文件将不再被处理。例如，`find /tmp/foo /tmp/bar -print -quit`将仅打印 `/tmp/foo`。`-quit`的一个常见用途是在找到所需内容后停止搜索文件系统。

### 全局选项

始终返回真值。全局选项对命令行中较早出现的测试也会生效。为避免混淆，全局选项应在命令行上列出**起始点之后、第一个测试选项、位置选项或操作选项之前指定**。若在其他位置指定全局选项，find 会发出警告消息，说明这可能引起混淆。

> 全局选项出现在起始点列表之后，因此与例如`-L` 这样的选项不属于同一类别。

#### -d `-depth`的同义词

##### *无参数*

##### 描述

仅用于与 FreeBSD、NetBSD、MacOS X 和 OpenBSD 兼容。

#### -depth 遍历级别

##### 参数

```
levels
```

##### 描述

在处理目录本身之前，先处理目录中的内容。`-delete`操作也隐含了`-depth`。

#### -files0-from 从文件中读取起始点，而非通过命令行获取。

##### 参数

```
file
```

##### 描述

使用此选项可以安全地给 find 命令传递任意数量的起始点。使用此选项和在命令行中传递起始点**是互斥的**，因此不允许同时进行。文件参数是强制性的。文件中的起始点必须用 ASCII NUL 字符分隔。两个连续的 NUL 字符，即带有零长度文件名的起始点是不允许的，这将导致错误诊断，并随后产生非零退出码。

与标准调用不同，在标准调用中，如果没有传递路径参数，find 会默认将当前目录作为起始点。起始点的处理方式与其他情况相同，例如，find 命令会递归进入子目录，除非另有阻止。若要仅处理起始点，可以额外传递`-maxdepth 0`参数。

**其他说明**：如果一个文件在输入文件中被列出多次，则其是否会被多次访问未作规定。如果在查找操作期间文件被修改，结果同样未作规定。最后，find 退出时（无论是通过`-quit`还是其他方式），命名文件中的查找位置也未作规定。此处**未作规定**意味着它**可能有效也可能无效**，**或者不做任何特定的事情**，并且该行为可能因平台或 findutils 版本而异。

> 💡可以使用`-files0-from`**从标准输入流中读取起始点列表**，例如从管道中读取。在这种情况下，不允许使用`-ok`和`-okdir`操作，因为它们会干扰从标准输入读取以获取用户确认。

> ⚠️警告：如果给定文件为空，find 不会处理任何起始点，因此在解析完程序参数后会立即退出。

#### -help 和 --help 打印 find 命令行用法的摘要并退出。

##### *无参数*

##### 描述

*无描述*

#### -ignore_readdir_race

##### *无参数*

##### 描述

通常情况下，当 find 无法对文件进行状态检查（stat）时，会发出错误消息。如果您**启用此选项**，并且在 find 从目录读取文件名，到尝试进行状态检查**之间的时间内文件被删除**，则不会发出任何错误消息。这也适用于命令行中指定的文件或目录。此选项在命令行读取时生效，这意味着您不能在文件系统的某部分启用此选项，而在另一部分禁用它（如果需要这样做，您需要发出两个 find 命令，一个启用选项，一个不启用）。此外，使用`-ignore_readdir_race`选项时，如果在读取父目录后文件已消失，find 命令将忽略`-delete`操作的错误：它不会输出错误诊断信息，并且`-delete`操作的返回码将为真。

#### -maxdepth 最大遍历级别

##### 参数

```
levels
```

##### 描述

最多向下遍历 levels 级（一个非负整数）目录层级。使用`-maxdepth 0`表示**仅对起始点本身**应用测试和操作。

#### -mindepth 最小遍历级别

##### 参数

```
levels
```

##### 描述

在小于指定级别（非负整数）的层级上不执行任何测试或操作。使用`-mindepth 1`表示处理**除起始点外的所有文件**。

#### -mount 不在其他文件系统中下降目录

##### *无参数*

##### 描述

这是`-xdev`的替代名称，用于与其他一些版本的 find 兼容。

#### -noignore_readdir_race

##### *无参数*

##### 描述

关闭了`-ignore_readdir_race`的效果。

#### -noleaf 不进行优化。

##### *无参数*

##### 描述

不通过假设目录包含比其硬链接数少 2 个子目录来进行优化。在搜索不遵循 Unix 目录链接惯例的文件系统时，需要此选项，例如 CD-ROM、MS-DOS 文件系统或 AFS 卷挂载点。在正常的 Unix 文件系统上，每个目录至少有 2 个硬链接：其名称及其`.`条目。此外，其子目录（如果有）各自有一个指向该目录的`..`条目。当 find 检查一个目录时，在它已经统计了比目录链接数少 2 个子目录之后，它知道该目录中的其余条目是非目录（目录树中的“叶”文件）。如果只需要检查文件的名称，则无需对其进行状态检查；这可以显著提高搜索速度。

#### -version 和 --version 打印 find 的版本号并退出。

##### *无参数*

##### 描述

*无描述*

#### -xdev 不进入其他文件系统的目录。

##### *无参数*

##### 描述

*无描述*

### 位置选项

始终返回真值。它们仅影响命令行中后续的测试。

#### -daystart 从今天开始

> 用于 `-amin`、`-atime`、`-cmin`、`-ctime`、`-mmin` 和 `-mtime`

##### *无参数*

##### 描述

从今天开始而非从 24 小时前开始。此选项仅影响命令行中后续出现的测试。

#### ~~-follow~~ 解引用符号链接。

##### *无参数*

##### 描述

**已弃用，请改用`-L`选项**。隐含`-noleaf`。`-follow`选项仅影响命令行中出现在其后的那些测试。除非已指定`-H`或`-L`选项，否则`-follow`选项的位置会改变`-newer`谓词的行为；作为`-newer`参数列出的任何文件，如果它们是符号链接，则会被解引用。同样的情况适用于`-newerXY`、`-anewer`和`-cnewer`。类似地，`-type `谓词将始终匹配符号链接所指向的文件类型，而非链接本身。使用`-follow`会导致 `-lname`和`-ilname`谓词始终返回 false。

#### -regextype 更改正则表达式语法

##### 参数

```
type
```

##### 描述

更改`-regex`和`-iregex`测试在命令行后续部分所理解的正则表达式语法。要查看已知的正则表达式类型，请使用`-regextype help`。Texinfo 文档解释了各种正则表达式类型的含义及其差异。如果您不使用此选项，find 的行为如同已指定正则表达式类型为`emacs`。

#### -warn 和 -nowarn 开启或关闭警告消息。

##### *无参数*

##### 描述

这些警告仅适用于命令行使用，不适用于 find 在搜索目录时可能遇到的情况。默认行为是：如果标准输入是`tty`，则对应`-warn`；否则对应`-nowarn`。如果产生与命令行使用相关的警告消息，find 的退出状态不受影响。如果设置了 POSIXLY_CORRECT 环境变量，并且也使用了`-warn`，则未指定哪些（如果有）警告会被激活。

### 运算符选项

运算符按优先级递减顺序列出：

- `(expr)` 强制优先级。由于括号对 shell 有特殊含义，通常需要对它们进行引用。许多示例为此使用了反斜杠：`\(...\)` 而非 `(...)`。
- `! expr` 若表达式为假则结果为真（取反）。此字符通常也需要防止被 shell 解释。

> 💡提示：当`-a`隐式指定（例如两个测试之间没有显式运算符）或显式指定时，其优先级高于`-o`。例如，`find . -name foo -o -name bar -print`永远不会打印`foo`。

#### -not

##### 参数

```
expr
```

##### 描述

等同于`! expr`，但不符合 POSIX 标准。

#### -a

##### 参数

```
expr1` -a `expr2
```

##### 描述

两个连续的表达式被视为隐含地用`-a`连接；如果`expr1`为假，则不评估`expr2`。等同于`expr1 expr2`。

#### -and

##### 参数

```
expr1` -and `expr2
```

##### 描述

与`-a`相同。但不符合 POSIX 标准。

#### -o

##### 参数

```
expr1` -o `expr2
```

##### 描述

`expr1`和`expr2`始终都会被评估。`expr1`的值会被丢弃；列表的值即为`expr2`的值。逗号运算符（`,`）在搜索多种不同类型的事物时非常有用，但只会遍历文件系统层次结构一次。`-fprintf`动作可用于将各种匹配项列出到多个不同的输出文件中。若`expr1`为真，则不评估`expr2`。

#### -or

##### 参数

```
expr1` -or `expr2
```

##### 描述

与`-o`相同。但不符合 POSIX 标准。

## 例子

当前目录搜索所有文件，且文件内容包含 “140.206.111.111”

```shell
find . -type f -name "*" | xargs grep "140.206.111.111"
```

#### 根据文件或者正则表达式进行匹配

列出当前目录及子目录下所有文件和文件夹

```shell
find .
```

在`/home`目录下查找以.txt结尾的文件名

```shell
find /home -name "*.txt"
```

同上，但忽略大小写

```shell
find /home -iname "*.txt"
```

当前目录及子目录下查找所有以.txt和.pdf结尾的文件

```shell
find . \( -name "*.txt" -o -name "*.pdf" \)

或

find . -name "*.txt" -o -name "*.pdf"
```

匹配文件路径或者文件

```shell
find /usr/ -path "*local*"
```

基于正则表达式匹配文件路径

```shell
find . -regex ".*\(\.txt\|\.pdf\)$"
```

同上，但忽略大小写

```shell
find . -iregex ".*\(\.txt\|\.pdf\)$"
```

#### 否定参数

找出/home下不是以.txt结尾的文件

```shell
find /home ! -name "*.txt"
```

#### 根据文件类型进行搜索

```shell
find . -type 类型参数
```

类型参数列表：

- **f** 普通文件
- **l** 符号连接
- **d** 目录
- **c** 字符设备
- **b** 块设备
- **s** 套接字
- **p** Fifo

#### 基于目录深度搜索

向下最大深度限制为3

```shell
find . -maxdepth 3 -type f
```

搜索出深度距离当前目录至少2个子目录的所有文件

```shell
find . -mindepth 2 -type f
```

#### 根据文件时间戳进行搜索

```shell
find . -type f 时间戳
```

UNIX/Linux文件系统每个文件都有三种时间戳：

- **访问时间** （-atime/天，-amin/分钟）：用户最近一次访问时间。
- **修改时间** （-mtime/天，-mmin/分钟）：文件最后一次修改时间。
- **变化时间** （-ctime/天，-cmin/分钟）：文件数据元（例如权限等）最后一次修改时间。

搜索最近七天内被访问过的所有文件

```shell
find . -type f -atime -7
```

搜索恰好在七天前被访问过的所有文件

```shell
find . -type f -atime 7
```

搜索超过七天内被访问过的所有文件

```shell
find . -type f -atime +7
```

搜索访问时间超过10分钟的所有文件

```shell
find . -type f -amin +10
```

找出比file.log修改时间更长的所有文件

```shell
find . -type f -newer file.log
```

#### 根据文件大小进行匹配

```shell
find . -type f -size 文件大小单元
```

文件大小单元：

- **b** —— 块（512字节）
- **c** —— 字节
- **w** —— 字（2字节）
- **k** —— 千字节
- **M** —— 兆字节
- **G** —— 吉字节

搜索大于10KB的文件

```shell
find . -type f -size +10k
```

搜索小于10KB的文件

```shell
find . -type f -size -10k
```

搜索等于10KB的文件

```shell
find . -type f -size 10k
```

#### 删除匹配文件

删除当前目录下所有.txt文件

```shell
find . -type f -name "*.txt" -delete
```

#### 根据文件权限/所有权进行匹配

当前目录下搜索出权限为777的文件

```shell
find . -type f -perm 777
```

找出当前目录下权限不是644的php文件

```shell
find . -type f -name "*.php" ! -perm 644
```

找出当前目录用户tom拥有的所有文件

```shell
find . -type f -user tom
```

找出当前目录用户组sunk拥有的所有文件

```shell
find . -type f -group sunk
```

#### 借助`-exec`选项与其他命令结合使用

找出当前目录下所有root的文件，并把所有权更改为用户tom

```shell
find .-type f -user root -exec chown tom {} \;
```

上例中， **{}** 用于与 **-exec** 选项结合使用来匹配所有文件，然后会被替换为相应的文件名。

找出自己家目录下所有的.txt文件并删除

```shell
find $HOME/. -name "*.txt" -ok rm {} \;
```

上例中， **-ok** 和 **-exec** 行为一样，不过它会给出提示，是否执行相应的操作。

查找当前目录下所有.txt文件并把他们拼接起来写入到all.txt文件中

```shell
find . -type f -name "*.txt" -exec cat {} \;> /all.txt
```

将30天前的.log文件移动到old目录中

```shell
find . -type f -mtime +30 -name "*.log" -exec cp {} old \;
```

找出当前目录下所有.txt文件并以“File:文件名”的形式打印出来

```shell
find . -type f -name "*.txt" -exec printf "File: %s\n" {} \;
```

因为单行命令中-exec参数中无法使用多个命令，以下方法可以实现在-exec之后接受多条命令

```shell
-exec ./text.sh {} \;
```

#### 搜索但跳过指定的目录

查找当前目录或者子目录下所有.txt文件，但是跳过子目录sk

```shell
find . -path "./sk" -prune -o -name "*.txt" -print
```

> ⚠️ ./sk 不能写成 ./sk/ ，否则没有作用。

忽略两个目录

```shell
find . \( -path ./sk -o  -path ./st \) -prune -o -name "*.txt" -print
```

> ⚠️ 如果写相对路径必须加上`./`

#### find其他技巧收集

要列出所有长度为零的文件

```shell
find . -empty
```

#### 其它实例

```shell
find ~ -name '*jpg' # 主目录中找到所有的 jpg 文件。 -name 参数允许你将结果限制为与给定模式匹配的文件。
find ~ -iname '*jpg' # -iname 就像 -name，但是不区分大小写
find ~ ( -iname 'jpeg' -o -iname 'jpg' ) # 一些图片可能是 .jpeg 扩展名。幸运的是，我们可以将模式用“或”（表示为 -o）来组合。
find ~ \( -iname '*jpeg' -o -iname '*jpg' \) -type f # 如果你有一些以 jpg 结尾的目录呢？ （为什么你要命名一个 bucketofjpg 而不是 pictures 的目录就超出了本文的范围。）我们使用 -type 参数修改我们的命令来查找文件。
find ~ \( -iname '*jpeg' -o -iname '*jpg' \) -type d # 也许你想找到那些命名奇怪的目录，以便稍后重命名它们
```

最近拍了很多照片，所以让我们把它缩小到上周更改的文件

```shell
find ~ \( -iname '*jpeg' -o -iname '*jpg' \) -type f -mtime -7
```

你可以根据文件状态更改时间 （ctime）、修改时间 （mtime） 或访问时间 （atime） 来执行时间过滤。 这些是在几天内，所以如果你想要更细粒度的控制，你可以表示为在几分钟内（分别是 cmin、mmin 和 amin）。 除非你确切地知道你想要的时间，否则你可能会在 + （大于）或 - （小于）的后面加上数字。

但也许你不关心你的照片。也许你的磁盘空间不够用，所以你想在 log 目录下找到所有巨大的（让我们定义为“大于 1GB”）文件：

```shell
find /var/log -size +1G
```

或者，也许你想在 /data 中找到 bcotton 拥有的所有文件：

```shell
find /data -owner bcotton
```

你还可以根据权限查找文件。也许你想在你的主目录中找到对所有人可读的文件，以确保你不会过度分享。

```shell
find ~ -perm -o=r
```

删除 mac 下自动生成的文件

```shell
find ./ -name '__MACOSX' -depth -exec rm -rf {} \;
```

统计代码行数

```shell
find . -name "*.java"|xargs cat|grep -v ^$|wc -l # 代码行数统计, 排除空行
```

# docker

容器化技术，可以将应用程序及其依赖项打包到一个可移植的容器中，使其可以在不同的环境中运行

## 补充说明

Docker 容器可以快速部署、可移植、可扩展，并且可以在不同的平台上运行。Docker 可以帮助开发人员和运维人员更轻松地构建、发布和管理应用程序。

## 安装

在 Linux 中输入以下命令安装 Docker。

```bash
# CentOS    参考：https://blog.csdn.net/zhaoyuanh/article/details/126610347
#如果系统里有旧版本docker的话需要先行删除：
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

#设置仓库：
yum install -y yum-utils

#添加Docker仓库：
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

#安装Docker引擎(默认最新):
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

#启动docker:
sudo systemctl start docker
# Docker官方提供的快速安装脚本 https://github.com/docker/docker-install 
# 不建议在生产环境中使用
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run

# 使用systemctl设置开机启动
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

## 语法

```shell
docker create [options] IMAGE
```

## 选项参数

```shell
attach	将本地标准输入、输出和错误流附加到正在运行的容器
build	从 Dockerfile 构建镜像
commit	从容器的更改创建新镜像
cp	在容器和本地文件系统之间复制文件/文件夹
create	创建一个新容器
diff	检查容器文件系统上文件或目录的更改
events	从服务器获取实时事件
exec	在正在运行的容器中运行命令
export	将容器的文件系统导出为 tar 存档
history	显示镜像的历史
images	列出镜像
import	从 tarball 导入内容以创建文件系统映像
info	显示系统范围的信息
inspect	返回有关 Docker 对象的低级信息
kill	杀死一个或多个正在运行的容器
load	从 tar 存档或 STDIN 加载镜像
login	登录到 Docker 注册表
logout	从 Docker 注册表中注销
logs	获取容器的日志
pause	暂停一个或多个容器内的所有进程
port	列出容器的端口映射或特定映射
ps	列出容器
pull	从注册表中提取镜像或存储库
push	将镜像或存储库推送到注册表
rename	重命名容器
restart	重启一个或多个容器
rm	移除一个或多个容器
rmi	移除一张或多张镜像
run	在新容器中运行命令
save	将一个或多个镜像保存到 tar 存档（默认流式传输到 STDOUT）
search	在 Docker Hub 中搜索镜像
start	启动一个或多个停止的容器
stats	显示容器资源使用统计的实时流
stop	停止一个或多个正在运行的容器
tag	创建一个引用 SOURCE_IMAGE 的标记 TARGET_IMAGE
top	显示容器的运行进程
unpause	取消暂停一个或多个容器中的所有进程
update	更新一个或多个容器的配置
version	显示 Docker 版本信息
wait	阻塞直到一个或多个容器停止，然后打印它们的退出代码

<环境参数>
    --add-host list            # 添加自定义主机到 IP 映射 (host:ip)
-a, --attach list              # 连接到 STDIN、STDOUT 或 STDERR
    --blkio-weight uint16      # 块 IO（相对权重），介于 10 和 1000 之间，或 0 禁用（默认 0）
    --blkio-weight-device list # 块 IO 权重（相对设备权重）（默认 []）
    --cap-add list             # 添加 Linux 功能
    --cap-drop list            # 放弃 Linux 功能
    --cgroup-parent string     # 容器的可选父 cgroup
    --cgroupns string   # 要使用的 Cgroup 命名空间（主机|私有）
                        #  'host':    在 Docker 主机的 cgroup 命名空间中运行容器
                        #  'private': 在自己的私有 cgroup 命名空间中运行容器
                        #  '':        使用由守护进程上的 
                        #        default-cgroupns-mode 选项配置的 cgroup 命名空间（默认）
    --cidfile string           # 将容器 ID 写入文件
    --cpu-period int           # 限制 CPU CFS（完全公平调度器）周期
    --cpu-quota int            # 限制 CPU CFS（完全公平调度器）配额
    --cpu-rt-period int        # 以微秒为单位限制 CPU 实时周期
    --cpu-rt-runtime int       # 以微秒为单位限制 CPU 实时运行时间
-c, --cpu-shares int           # CPU 份额（相对权重）
    --cpus decimal             # CPU 数量
    --cpuset-cpus string       # 允许执行的 CPU (0-3, 0,1)
    --cpuset-mems string       # 允许执行的 MEM (0-3, 0,1)
    --device list              # 将主机设备添加到容器
    --device-cgroup-rule list  # 将规则添加到 cgroup 允许的设备列表
    --device-read-bps list     # 限制设备的读取速率（每秒字节数）（默认 []）
    --device-read-iops list    # 限制设备的读取速率（每秒 IO）（默认 []）
    --device-write-bps list    # 限制设备的写入速率（每秒字节数）（默认 []）
    --device-write-iops list   # 限制设备的写入速率（每秒 IO）（默认 []）
    --disable-content-trust    # 跳过镜像验证（默认为 true）
    --dns list                 # 设置自定义 DNS 服务器
    --dns-option list          # 设置 DNS 选项
    --dns-search list          # 设置自定义 DNS 搜索域
    --domainname string        # 容器 NIS 域名
    --entrypoint string        # 覆盖镜像的默认入口点
-e, --env list                 # 设置环境变量
    --env-file list            # 读入环境变量文件
    --expose list              # 公开一个端口或一系列端口
    --gpus gpu-request         # 要添加到容器中的 GPU 设备（“全部”以传递所有 GPU）
    --group-add list           # 添加其他组以加入
    --health-cmd string        # 运行以检查运行状况的命令
    --health-interval duration # 运行检查之间的时间 (ms|s|m|h) (默认 0s)
    --health-retries int           # 需要报告不健康的连续失败
    --health-start-period duration # 开始健康重试倒计时之前容器初始化的开始时间（ms|s|m|h）（默认 0s）
    --health-timeout duration      # 允许运行一项检查的最长时间 (ms|s|m|h) (默认 0s)
    --help                     # 打印使用
-h, --hostname string          # 容器主机名
    --init                     # 在容器内运行一个 init 来转发信号并收获进程
-i, --interactive              # 即使没有连接，也保持 STDIN 打开
    --ip string                # IPv4 地址（例如 172.30.100.104）
    --ip6 string               # IPv6 地址（例如，2001:db8::33）
    --ipc string               # 要使用的 IPC 模式
    --isolation string         # 容器隔离技术
    --kernel-memory bytes      # 内核内存限制
-l, --label list               # 在容器上设置元数据
    --label-file list          # 读入以行分隔的标签文件
    --link list                # 添加到另一个容器的链接
    --link-local-ip list       # 容器 IPv4/IPv6 链路本地地址
    --log-driver string        # 容器的日志记录驱动程序
    --log-opt list             # 日志驱动程序选项
    --mac-address string       # 容器 MAC 地址（例如 92:d0:c6:0a:29:33）
-m, --memory bytes             # 内存限制
    --memory-reservation bytes # 内存软限制
    --memory-swap bytes        # 交换限制等于内存加上交换：'-1' 启用无限交换
    --memory-swappiness int    # 调整容器内存交换（0 到 100）（默认 -1）
    --mount mount              # 将文件系统挂载附加到容器
    --name string              # 为容器分配名称
    --network network          # 将容器连接到网络
    --network-alias list       # 为容器添加网络范围的别名
    --no-healthcheck           # 禁用任何容器指定的 HEALTHCHECK
    --oom-kill-disable         # 禁用 OOM 杀手
    --oom-score-adj int        # 调整主机的 OOM 首选项（-1000 到 1000）
    --pid string               # 要使用的 PID 命名空间
    --pids-limit int           # 调整容器 pids 限制（设置 -1 表示无限制）
    --platform string          # 如果服务器支持多平台，则设置平台
    --privileged               # 授予此容器扩展权限
-p, --publish list             # 将容器的端口发布到主机
-P, --publish-all              # 将所有暴露的端口发布到随机端口
    --pull string              # 创建前拉取镜像("always"|"missing"|"never")(默认"missing")
    --read-only                # 将容器的根文件系统挂载为只读
    --restart string           # 容器退出时应用的重启策略（默认“否”）
    --rm                       # 容器退出时自动移除
    --runtime string           # 用于此容器的运行时
    --security-opt list        # 安全选项
    --shm-size bytes           # /dev/shm 的大小
    --stop-signal string       # 停止容器的信号（默认“SIGTERM”）
    --stop-timeout int         # 停止容器的超时（以秒为单位）
    --storage-opt list         # 容器的存储驱动程序选项
    --sysctl map               # Sysctl 选项（默认 map[]）
    --tmpfs list               # 挂载 tmpfs 目录
-t, --tty                      # 分配一个伪 TTY
    --ulimit ulimit            # ulimit 选项（默认 []）
-u, --user string              # 用户名或 UID（格式：<name|uid>[:<group|gid>]）
    --userns string            # 要使用的用户命名空间
    --uts string               # 要使用的 UTS 命名空间
-v, --volume list              # 绑定挂载卷
    --volume-driver string     # 容器的可选卷驱动程序
    --volumes-from list        # 从指定容器挂载卷
-w, --workdir string           # 容器内的工作目录
```

## 实例

介绍几个常用场景：Docker Hub镜像市场相关，镜像仓库命令。

1、下载docker hub镜像市场中的镜像。

```bash
docker pull user/image
```

2、在 docker hub 中搜索镜像。

```bash
# 注意需要下载镜像才能使用
docker search search_word
```

3、向 docker hub 进行身份验证。

```bash
docker login
```

4、将镜像上传到 docker hub。

```bash
docker push user/image
```

## docker network

## 语法

```
docker network [COMMAND]
```

## COMMAND

### docker network connect

将容器连接到网络。您可以按名称或ID连接容器。连接后，容器可以与同一网络中的其他容器通信。

```shell
docker network connect [OPTIONS] NETWORK CONTAINER
```

#### 选项参数

```shell
--alias	为容器添加网络范围的别名
--driver-opt	网络的驱动程序选项
--ip	IPv4地址（例如172.30.100.104）
--ip6	IPv6地址（例如2001：db8 :: 33）
--link	将链接添加到另一个容器(建议不用,后期应该会删除的)
--link-local-ip	为容器添加本地链接地址
```

### docker network disconnect

断开容器与网络的连接

```shell
docker network disconnect [OPTIONS] NETWORK CONTAINER
```

#### 选项参数

```shell
-f,--force	强制容器断开网络连接
```

### docker network create

创建一个新的网络

```shell
docker network create [OPTIONS] NETWORK
```

#### 选项参数

```shell
--attachable		API 1.25+启用手动容器附件
--aux-address		网络驱动程序使用的辅助IPv4或IPv6地址
--config-from		API 1.30+从中复制配置的网络
--config-only		API 1.30+创建仅配置网络
-d,--driver	bridge	驱动程序来管理网络
--gateway		主子网的IPv4或IPv6网关
--ingress		API 1.29+创建群集路由网状网络
--internal		限制外部访问网络
--ip-range		从子范围分配容器ip
--ipam-driver		IP地址管理驱动程序
--ipam-opt		设置IPAM驱动程序特定选项
--ipv6		启用IPv6网络
--label		在网络上设置元数据
-o,--opt		设置驱动程序特定选项
--scope		API 1.30+控制网络范围
--subnet		代表网段的CIDR格式的子网
```

### docker network inspect

返回有关一个或多个网络的信息。默认情况下，此命令将所有结果呈现在JSON对象中。

```shell
docker network inspect [OPTIONS] NETWORK [NETWORK...]
```

#### 选项参数

```shell
-f,--format	使用给定的Go模板格式化输出
-v,--verbose	详细输出以进行诊断
```

### docker network ls

列出引擎daemon知道的所有网络。这包括跨群集中多个主机的网络

```shell
docker network ls [OPTIONS]
```

#### 选项参数

```shell
-f,--filter	提供过滤器值（例如"driver = bridge"）
--format	使用Go模板的精美印刷网络
--no-trunc	不要截断输出
-q,--quiet	仅显示网络ID
```

### docker network prune

删除所有未使用的网络。未使用的网络是未被任何正在使用的容器引用的网络()。

```shell
docker network prune [OPTIONS]
```

#### 选项参数

```shell
--filter	提供过滤器值（例如'until ='）
-f,--force	不提示确认
```

### docker network rm

按名称或标识符删除一个或多个网络。要删除网络，必须首先断开连接到它的所有容器。

```shell
docker network rm NETWORKID [NETWORKID...]
```

# lscpu

显示有关CPU架构的信息

## 补充说明

**lscpu命令** 是显示有关CPU架构的信息。

### 语法

```shell
lscpu [选项]
```

### 选项

```shell
 -a, --all               # 打印在线和离线CPU（默认为-e）
 -b, --online            # 仅打印在线CPU（-p的默认值）
 -c, --offline           # 打印离线CPU
 -e, --extended[=<list>] # 打印出一个扩展的可读格式
 -p, --parse[=<list>]    # 打印出可解析的格式
 -s, --sysroot <dir>     # 将指定的目录用作系统根目录
 -x, --hex               # 打印十六进制掩码，而不是CPU列表

 -h, --help     # 显示此帮助并退出
 -V, --version  # 输出版本信息并退出
```

### 参数

```shell
可用列：
           CPU  逻辑CPU编号
          CORE  逻辑核心号码
        SOCKET  逻辑套接字号
          NODE  逻辑NUMA节点号
          BOOK  逻辑书号
         CACHE  显示了如何在CPU之间共享高速缓存
  POLARIZATION  虚拟硬件上的CPU调度模式
       ADDRESS  CPU的物理地址
    CONFIGURED  显示管理程序是否分配了CPU
        ONLINE  显示Linux是否正在使用CPU
```

### 例子

```shell
[root@localhost ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 30
Model name:            Intel(R) Xeon(R) CPU           X3430  @ 2.40GHz
Stepping:              5
CPU MHz:               2394.055
BogoMIPS:              4788.11
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              8192K
NUMA node0 CPU(s):     0-3
# 查看cpu编号对应的核心号码，区分是大核还是小核。
[root@localhost ~]# lscpu -e
CPU NODE SOCKET CORE L1d:L1i:L2:L3 ONLINE MAXMHZ    MINMHZ
0   0    0      0    0:0:0:0       是     3600.0000 800.0000
1   0    0      1    1:1:1:0       是     3600.0000 800.0000
2   0    0      2    2:2:2:0       是     3600.0000 800.0000
3   0    0      3    3:3:3:0       是     3600.0000 800.0000
4   0    0      0    0:0:0:0       是     3600.0000 800.0000
5   0    0      1    1:1:1:0       是     3600.0000 800.0000
6   0    0      2    2:2:2:0       是     3600.0000 800.0000
7   0    0      3    3:3:3:0       是     3600.0000 800.0000
```

# journalctl

检索 systemd 日志，只要使用 systemd 的 Linux 发行版（如 Fedora、Ubuntu Modern、Debian、SUSE、Arch），几乎都会配备 journalctl。

### 语法

```shell
journalctl [OPTIONS...] [MATCHES...]
```

### 选项

```shell
Flags:
 --system               # 显示系统日志
 --user                 # 显示当前用户的用户日志
-M --machine=CONTAINER  # 在本地容器上操作
-S --since=DATE         # 显示不早于指定日期的条目
-U --until=DATE         # 显示不晚于指定日期的条目
-c --cursor=CURSOR      # 显示从指定光标开始的条目
  --after-cursor=CURSOR # 在指定光标后显示条目
  --show-cursor         # 在所有条目之后打印光标
-b --boot[=ID]          # 显示当前启动或指定启动
  --list-boots          # 显示有关已记录引导的简洁信息
-k --dmesg              # 显示当前启动的内核消息日志
-u --unit=UNIT          # 显示指定单元的日志
-t --identifier=STRING  # 显示具有指定系统日志标识符的条目
-p --priority=RANGE     # 显示具有指定优先级的条目
-e --pager-end          # 在pager中立即跳转到末尾
-f --follow             # 关注期刊
-n --lines[=INTEGER]    # 要显示的日志条目数
  --no-tail             # 显示所有行，即使在跟随模式下
-r --reverse            # 首先显示最新的条目
-o --output=STRING      # 更改日志输出模式 (short, short-iso,
                                   short-precise, short-monotonic, verbose,
                                   export, json, json-pretty, json-sse, cat)
--utc                   # 以协调世界时 (UTC) 表示的时间
-x --catalog            # 在可用的情况下添加消息说明
   --no-full            # Ellipsize 字段
-a --all                # 显示所有字段，包括长的和不可打印的
-q --quiet              # 不显示特权警告
   --no-pager           # 不要将输出通过管道传输到寻呼机
-m --merge              # 显示所有可用期刊的条目
-D --directory=PATH     # 显示目录中的日志文件
   --file=PATH          # 显示日志文件
   --root=ROOT          # 对根目录下的目录文件进行操作
   --interval=TIME      # 更改 FSS 密封键的时间间隔
   --verify-key=KEY     # 指定FSS验证密钥
   --force              # 使用 --setup-keys 覆盖 FSS 密钥对 

Commands:
-h --help              # 显示此帮助文本
   --version           # 显示包版本
-F --field=FIELD       # 列出指定字段的所有值
   --new-id128         # 生成新的 128 位 ID
   --disk-usage        # 显示所有日志文件的总磁盘使用情况
   --vacuum-size=BYTES # 将磁盘使用量减少到指定大小以下
   --vacuum-time=TIME  # 删除早于指定日期的日志文件
   --flush             # 将所有日志数据从 /run 刷新到 /var
   --header            # 显示期刊头信息
   --list-catalog      # 显示目录中的所有消息 ID
   --dump-catalog      # 在消息目录中显示条目
   --update-catalog    # 更新消息目录数据库
   --setup-keys        # 生成新的 FSS 密钥对
   --verify            # 验证日志文件的一致性
```

### 实例

**过滤输出**

`journalctl` 可以根据特定字段过滤输出。如果过滤的字段比较多，需要较长时间才能显示出来。

示例：

显示本次启动后的所有日志：

```shell
journalctl -b
```

不过，一般大家更关心的不是本次启动后的日志，而是上次启动时的（例如，刚刚系统崩溃了）。可以使用 -b 参数：

- `journalctl -b -0` 显示本次启动的信息
- `journalctl -b -1` 显示上次启动的信息
- `journalctl -b -2` 显示上上次启动的信息 `journalctl -b -2`

只显示错误、冲突和重要告警信息

```shell
journalctl -p err..alert
```

也可以使用数字， `journalctl -p 3..1`。如果使用单个 number/keyword，则 `journalctl -p 3` - 还包括所有更高的优先级。

显示从某个日期 ( 或时间 ) 开始的消息:

```shell
journalctl --since="2012-10-30 18:17:16"
```

显示从某个时间 ( 例如 20分钟前 ) 的消息:

```shell
journalctl --since "20 min ago"
```

显示最新信息

```shell
journalctl -f
```

显示特定程序的所有消息:

```shell
journalctl /usr/lib/systemd/systemd
```

显示特定进程的所有消息:

```shell
journalctl _PID=1
```

显示指定单元的所有消息：

```shell
journalctl -u man-db.service
```

显示内核环缓存消息r:

```shell
journalctl -k
```

**手动清理日志**

`/var/log/journal` 存放着日志, `rm` 应该能工作. 或者使用 `journalctl`,

例如:

清理日志使总大小小于 100M:

```shell
journalctl --vacuum-size=100M
```

清理最早两周前的日志.

```shell
journalctl --vacuum-time=2weeks
```

# rm

用于删除给定的文件和目录

## 补充说明

**rm** **命令** 可以删除一个目录中的一个或多个文件或目录，也可以将某个目录及其下属的所有文件及其子目录均删除掉。对于链接文件，只是删除整个链接文件，而原有文件保持不变。

注意：使用rm命令要格外小心。因为一旦删除了一个文件，就无法再恢复它。所以，在删除文件之前，最好再看一下文件的内容，确定是否真要删除。rm命令可以用-i选项，这个选项在使用文件扩展名字符删除多个文件时特别有用。使用这个选项，系统会要求你逐一确定是否要删除。这时，必须输入y并按Enter键，才能删除文件。如果仅按Enter键或其他字符，文件不会被删除。

### 语法

```shell
rm (选项)(参数)
```

### 选项

```shell
-d：直接把欲删除的目录的硬连接数据删除成0，删除该目录；
-f：强制删除文件或目录；
-i：删除已有文件或目录之前先询问用户；
-r或-R：递归处理，将指定目录下的所有文件与子目录一并处理；
--preserve-root：不对根目录进行递归操作；
-v：显示指令的详细执行过程。
```

### 参数

文件：指定被删除的文件列表，如果参数中含有目录，则必须加上`-r`或者`-R`选项。

### 实例

交互式删除当前目录下的文件test和example

```shell
rm -i test example
Remove test ?n（不删除文件test)
Remove example ?y（删除文件example)
```

删除当前目录下除隐含文件外的所有文件和子目录

```shell
# rm -r *
```

应注意，这样做是非常危险的!

**删除当前目录下的 package-lock.json 文件**

```shell
find .  -name "package-lock.json" -exec rm -rf {} \;
```

**查找 \*.html 结尾的文件并删除**

```shell
find ./docs -name "*.html" -exec rm -rf {} \;
```

**删除当前项目下 \*.html 结尾的文件**

```shell
rm -rf *.html
```

**删除当前目录下的 node_modules 目录**

```shell
find . -name 'node_modules' -type d -prune -exec rm -rf '{}' +
```

**删除文件**

```shell
# rm 文件1 文件2 ...
rm testfile.txt
```

**删除目录**

> rm -r [目录名称] -r 表示递归地删除目录下的所有文件和目录。 -f 表示强制删除

```shell
rm -rf testdir
rm -r testdir
```

**删除操作前有确认提示**

> rm -i [文件/目录]

```shell
rm -r -i testdir
```

**批量删除 `icons` 文件夹中的子文件夹中的 data 文件夹**

```shell
rm -rf icons/**/data
```

**rm 忽略不存在的文件或目录**

> -f 选项（LCTT 译注：即 “force”）让此次操作强制执行，忽略错误提示

```shell
rm -f [文件...]
```

**仅在某些场景下确认删除**

> 选项 -I，可保证在删除超过 3 个文件时或递归删除时（LCTT 译注： 如删除目录）仅提示一次确认。

```shell
rm -I file1 file2 file3
```

**删除根目录**

> 当然，删除根目录（/）是 Linux 用户最不想要的操作，这也就是为什么默认 rm 命令不支持在根目录上执行递归删除操作。 然而，如果你非得完成这个操作，你需要使用 --no-preserve-root 选项。当提供此选项，rm 就不会特殊处理根目录（/）了。

```shell
不给示例了，操作系统都被你删除了，你太坏了😆
```

**rm 显示当前删除操作的详情**

```shell
rm -v [文件/目录]
```

# grep

强大的文本搜索工具

## 补充说明

**grep** （global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。用于过滤/搜索的特定字符。可使用正则表达式能配合多种命令使用，使用上十分灵活。

### 选项

```shell
-a --text  # 不要忽略二进制数据。
-A <显示行数>   --after-context=<显示行数>   # 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
-b --byte-offset                           # 在显示符合范本样式的那一行之外，并显示该行之前的内容。
-B<显示行数>   --before-context=<显示行数>   # 除了显示符合样式的那一行之外，并显示该行之前的内容。
-c --count    # 计算符合范本样式的列数。
-C<显示行数> --context=<显示行数>或-<显示行数> # 除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
-d<进行动作> --directories=<动作>  # 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作。
-e<范本样式> --regexp=<范本样式>   # 指定字符串作为查找文件内容的范本样式。
-E --extended-regexp             # 将范本样式为延伸的普通表示法来使用，意味着使用能使用扩展正则表达式。
-f<范本文件> --file=<规则文件>     # 指定范本文件，其内容有一个或多个范本样式，让grep查找符合范本条件的文件内容，格式为每一列的范本样式。
-F --fixed-regexp   # 将范本样式视为固定字符串的列表。
-G --basic-regexp   # 将范本样式视为普通的表示法来使用。
-h --no-filename    # 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。
-H --with-filename  # 在显示符合范本样式的那一列之前，标示该列的文件名称。
-i --ignore-case    # 忽略字符大小写的差别。
-l --file-with-matches   # 列出文件内容符合指定的范本样式的文件名称。
-L --files-without-match # 列出文件内容不符合指定的范本样式的文件名称。
-n --line-number         # 在显示符合范本样式的那一列之前，标示出该列的编号。
-P --perl-regexp         # PATTERN 是一个 Perl 正则表达式
-q --quiet或--silent     # 不显示任何信息。
-R/-r  --recursive       # 此参数的效果和指定“-d recurse”参数相同。
-s --no-messages  # 不显示错误信息。
-v --revert-match # 反转查找。
-V --version      # 显示版本信息。   
-w --word-regexp  # 只显示全字符合的列。
-x --line-regexp  # 只显示全列符合的列。
-y # 此参数效果跟“-i”相同。
-o # 只输出文件中匹配到的部分。
-m <num> --max-count=<num> # 找到num行结果后停止查找，用来限制匹配行数
```

### 规则表达式

```shell
^    # 锚定行的开始 如：'^grep'匹配所有以grep开头的行。    
$    # 锚定行的结束 如：'grep$' 匹配所有以grep结尾的行。
.    # 匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。    
*    # 匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。    
.*   # 一起用代表任意字符。   
[]   # 匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。    
[^]  # 匹配一个不在指定范围内的字符，如：'[^A-Z]rep' 匹配不包含 A-Z 中的字母开头，紧跟 rep 的行
\(..\)  # 标记匹配字符，如'\(love\)'，love被标记为1。    
\<      # 锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。    
\>      # 锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。    
x\{m\}  # 重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。    
x\{m,\}   # 重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。    
x\{m,n\}  # 重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。   
\w    # 匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。   
\W    # \w的反置形式，匹配一个或多个非单词字符，如点号句号等。   
\b    # 单词锁定符，如: '\bgrep\b'只匹配grep。  
```

## grep命令常见用法

在文件中搜索一个单词，命令会返回一个包含 **“match_pattern”** 的文本行：

```shell
grep match_pattern file_name
grep "match_pattern" file_name
```

在多个文件中查找：

```shell
grep "match_pattern" file_1 file_2 file_3 ...
```

输出除之外的所有行 **-v** 选项：

```shell
grep -v "match_pattern" file_name
```

标记匹配颜色 **--color=auto** 选项：

```shell
grep "match_pattern" file_name --color=auto
```

使用正则表达式 **-E** 选项：

```shell
grep -E "[1-9]+"
# 或
egrep "[1-9]+"
```

使用正则表达式 **-P** 选项：

```shell
grep -P "(\d{3}\-){2}\d{4}" file_name
```

只输出文件中匹配到的部分 **-o** 选项：

```shell
echo this is a test line. | grep -o -E "[a-z]+\."
line.

echo this is a test line. | egrep -o "[a-z]+\."
line.
```

统计文件或者文本中包含匹配字符串的行数 **-c** 选项：

```shell
grep -c "text" file_name
```

搜索命令行历史记录中 输入过 `git` 命令的记录：

```shell
history | grep git
```

输出包含匹配字符串的行数 **-n** 选项：

```shell
grep "text" -n file_name
# 或
cat file_name | grep "text" -n

#多个文件
grep "text" -n file_1 file_2
```

打印样式匹配所位于的字符或字节偏移：

```shell
echo gun is not unix | grep -b -o "not"
7:not
#一行中字符串的字符偏移是从该行的第一个字符开始计算，起始值为0。选项  **-b -o**  一般总是配合使用。
```

搜索多个文件并查找匹配文本在哪些文件中：

```shell
grep -l "text" file1 file2 file3...
```

### grep递归搜索文件

在多级目录中对文本进行递归搜索：

```shell
grep "text" . -r -n
# .表示当前目录。
```

忽略匹配样式中的字符大小写：

```shell
echo "hello world" | grep -i "HELLO"
# hello
```

选项 **-e** 制动多个匹配样式：

```shell
echo this is a text line | grep -e "is" -e "line" -o
is
is
line

#也可以使用 **-f** 选项来匹配多个样式，在样式文件中逐行写出需要匹配的字符。
cat patfile
aaa
bbb

echo aaa bbb ccc ddd eee | grep -f patfile -o
```

在grep搜索结果中包括或者排除指定文件：

```shell
# 只在目录中所有的.php和.html文件中递归搜索字符"main()"
grep "main()" . -r --include *.{php,html}

# 在搜索结果中排除所有README文件
grep "main()" . -r --exclude "README"

# 在搜索结果中排除filelist文件列表里的文件
grep "main()" . -r --exclude-from filelist
```

使用0值字节后缀的grep与xargs：

```shell
# 测试文件：
echo "aaa" > file1
echo "bbb" > file2
echo "aaa" > file3

grep "aaa" file* -lZ | xargs -0 rm

# 执行后会删除file1和file3，grep输出用-Z选项来指定以0值字节作为终结符文件名（\0），xargs -0 读取输入并用0值字节终结符分隔文件名，然后删除匹配文件，-Z通常和-l结合使用。
```

grep静默输出：

```shell
grep -q "test" filename
# 不会输出任何信息，如果命令运行成功返回0，失败则返回非0值。一般用于条件测试。
```

打印出匹配文本之前或者之后的行：

```shellegrep
# 显示匹配某个结果之后的3行，使用 -A 选项：
seq 10 | grep "5" -A 3
5
6
7
8

# 显示匹配某个结果之前的3行，使用 -B 选项：
seq 10 | grep "5" -B 3
2
3
4
5

# 显示匹配某个结果的前三行和后三行，使用 -C 选项：
seq 10 | grep "5" -C 3
2
3
4
5
6
7
8

# 如果匹配结果有多个，会用“--”作为各匹配结果之间的分隔符：
echo -e "a\nb\nc\na\nb\nc" | grep a -A 1
a
b
--
a
b
```

# egrep

在文件内查找指定的字符串

## 补充说明

**egrep命令** 用于在文件内查找指定的字符串。egrep执行效果与`grep -E`相似，使用的语法及参数可参照grep指令，与grep的不同点在于解读字符串的方法。egrep是用extended regular expression语法来解读的，而grep则用basic regular expression 语法解读，extended regular expression比basic regular expression的表达更规范。

### 语法

```shell
egrep(选项)(查找模式)(文件名1，文件名2，……)
```

### 实例

显示文件中符合条件的字符。例如，查找当前目录下所有文件中包含字符串"Linux"的文件，可以使用如下命令：

```shell
egrep Linux *
```

结果如下所示：

```shell
# 以下五行为 testfile 中包含Linux字符的行
testfile:hello Linux!
testfile:Linux is a free Unix-type operating system.
testfile:This is a Linux testfile!
testfile:Linux
testfile:Linux

# 以下两行为testfile1中含Linux字符的行
testfile1:helLinux!
testfile1:This a Linux testfile!

# 以下两行为 testfile_2 中包含Linux字符的行
testfile_2:Linux is a free unix-type opterating system
testfile_2:Linux test
```

过滤注释行和空白行

```shell
egrep -v '^\s*(#|$)' filename
```

# cat

连接多个文件并打印到标准输出。

## 概要

```shell
cat [OPTION]... [FILE]...
```

## 主要用途

- 显示文件内容，如果没有文件或文件为`-`则读取标准输入。
- 将多个文件的内容进行连接并打印到标准输出。
- 显示文件内容中的不可见字符（控制字符、换行符、制表符等）。

## 参数

FILE（可选）：要处理的文件，可以为一或多个。

## 选项

```shell
长选项与短选项等价

-A, --show-all           等价于"-vET"组合选项。
-b, --number-nonblank    只对非空行编号，从1开始编号，覆盖"-n"选项。
-e                       等价于"-vE"组合选项。
-E, --show-ends          在每行的结尾显示'$'字符。
-n, --number             对所有行编号，从1开始编号。
-s, --squeeze-blank      压缩连续的空行到一行。
-t                       等价于"-vT"组合选项。
-T, --show-tabs          使用"^I"表示TAB（制表符）。
-u                       POSIX兼容性选项，无意义。
-v, --show-nonprinting   使用"^"和"M-"符号显示控制字符，除了LFD（line feed，即换行符'\n'）和TAB（制表符）。

--help                   显示帮助信息并退出。
--version                显示版本信息并退出。
```

## 返回值

返回状态为成功除非给出了非法选项或非法参数。

## 例子

```shell
# 合并显示多个文件
cat ./1.log ./2.log ./3.log
# 显示文件中的非打印字符、tab、换行符
cat -A test.log
# 压缩文件的空行
cat -s test.log
# 显示文件并在所有行开头附加行号
cat -n test.log
# 显示文件并在所有非空行开头附加行号
cat -b test.log
# 将标准输入的内容和文件内容一并显示
echo '######' |cat - test.log
```

### 注意

1. 该命令是`GNU coreutils`包中的命令，相关的帮助信息请查看`man -s 1 cat`或`info coreutils 'cat invocation'`。
2. 当使用`cat`命令查看**体积较大的文件**时，文本在屏幕上迅速闪过（滚屏），用户往往看不清所显示的内容，为了控制滚屏，可以按`Ctrl+s`键停止滚屏；按`Ctrl+q`键恢复滚屏；按`Ctrl+c`（中断）键可以终止该命令的执行，返回Shell提示符状态。
3. 建议您查看**体积较大的文件**时使用`less`、`more`命令或`emacs`、`vi`等文本编辑器。

# tail

在屏幕上显示指定文件的末尾若干行

## 补充说明

**tail命令** 用于输入文件中的尾部内容。

- 默认在屏幕上显示指定文件的末尾10行。

- 处理多个文件时会在各个文件之前附加含有文件名的行。

- 如果没有指定文件或者文件名为`-`，则读取标准输入。

- 如果表示字节或行数的`NUM`值之前有一个`+`号，则从文件开头的第`NUM`项开始显示，而不是显示文件的最后`NUM`项。

- ```
  NUM
  ```

  值后面可以有后缀：

  - `b` : 512
  - `kB` : 1000
  - `k `: 1024
  - `MB` : 1000 * 1000
  - `M `: 1024 * 1024
  - `GB` : 1000 * 1000 * 1000
  - `G `: 1024 * 1024 * 1024
  - `T`、`P`、`E`、`Z`、`Y`等以此类推。

### 语法

```shell
tail (选项) (参数)
```

### 选项

```shell
-c, --bytes=NUM                 输出文件尾部的NUM（NUM为整数）个字节内容。
-f, --follow[={name|descript}]  显示文件最新追加的内容。“name”表示以文件名的方式监视文件的变化。
-F                              与 “--follow=name --retry” 功能相同。
-n, --line=NUM                  输出文件的尾部NUM（NUM位数字）行内容。
--pid=<进程号>                  与“-f”选项连用，当指定的进程号的进程终止后，自动退出tail命令。
-q, --quiet, --silent           当有多个文件参数时，不输出各个文件名。
--retry                         即是在tail命令启动时，文件不可访问或者文件稍后变得不可访问，都始终尝试打开文件。使用此选项时需要与选项“--follow=name”连用。
-s, --sleep-interal=<秒数>      与“-f”选项连用，指定监视文件变化时间隔的秒数。
-v, --verbose                   当有多个文件参数时，总是输出各个文件名。
--help                          显示指令的帮助信息。
--version                       显示指令的版本信息。
```

### 参数

文件列表：指定要显示尾部内容的文件列表。

### 实例

```shelltailf
tail file #（显示文件file的最后10行）
tail -n +20 file #（显示文件file的内容，从第20行至文件末尾）
tail -c 10 file #（显示文件file的最后10个字节）

tail -25 mail.log # 显示 mail.log 最后的 25 行
tail -f mail.log # 等同于--follow=descriptor，根据文件描述符进行追踪，当文件改名或被删除，追踪停止
tail -F mail.log # 等同于--follow=name --retry，根据文件名进行追踪，并保持重试，即该文件被删除或改名后，如果再次创建相同的文件名，会继续
```

# tailf

在屏幕上显示指定文件的末尾若干行内容，通常用于日志文件的跟踪输出

## 补充说明

tailf命令几乎等同于`tail -f`，严格说来应该与`tail --follow=name`更相似些。当文件改名之后它也能继续跟踪，特别适合于日志文件的跟踪（follow the growth of a log file）。与`tail -f`不同的是，如果文件不增长，它不会去访问磁盘文件。tailf特别适合那些便携机上跟踪日志文件，因为它能省电，因为减少了磁盘访问。tailf命令不是个脚本，而是一个用C代码编译后的二进制执行文件，某些Linux安装之后没有这个命令。

tailf和tail -f的区别

1. tailf 总是从文件开头一点一点的读， 而tail -f 则是从文件尾部开始读
2. tailf check文件增长时，使用的是文件名， 用stat系统调用；而tail -f 则使用的是已打开的文件描述符； 注：tail 也可以做到类似跟踪文件名的效果； 但是tail总是使用fstat系统调用，而不是stat系统调用；结果就是：默认情况下，当tail的文件被偷偷删除时，tail是不知道的，而tailf是知道的。

### 语法

```shell
tailf logfile # 动态跟踪日志文件logfile，最初的时候打印文件的最后10行内容。
```

### 选项

```shell
-n, --lines NUMBER  # 输出最后数行
-NUMBER             # 与NUMBER相同 `-n NUMBER'
-V, --version       # 输出版本信息并退出
-h, --help          # 显示帮助并退出
```

### 参数

目标：指定目标日志。

### 实例

```shell
tailf log/WEB.LOG 
tailf -n 5 log2014.log   # 显示文件最后5行内容
```

# pmap

报告进程的内存映射关系

## 补充说明

**pmap命令** 用于报告进程的内存映射关系，是Linux调试及运维一个很好的工具。

### 语法

```shell
pmap(选项)(参数)
```

### 选项

```shell
-x：显示扩展格式；
-d：显示设备格式；
-q：不显示头尾行；
-V：显示指定版本。
```

### 参数

进程号：指定需要显示内存映射关系的进程号，可以是多个进程号。

### 实例

```shell
pidof nginx
13312 5371

pmap -x 5371
5371:   nginx: worker process                
Address           Kbytes     RSS   Dirty Mode   Mapping
0000000000400000     564     344       0 r-x--  nginx
000000000068c000      68      68      60 rw---  nginx
000000000069d000      56      12      12 rw---    [ anon ]
000000000a0c8000    1812    1684    1684 rw---    [ anon ]
0000003ac0a00000     112      40       0 r-x--  ld-2.5.so
0000003ac0c1c000       4       4       4 r----  ld-2.5.so
0000003ac0c1d000       4       4       4 rw---  ld-2.5.so
0000003ac0e00000    1340     284       0 r-x--  libc-2.5.so
0000003ac0f4f000    2044       0       0 -----  libc-2.5.so
0000003ac114e000      16      16       8 r----  libc-2.5.so
0000003ac1152000       4       4       4 rw---  libc-2.5.so
0000003ac1153000      20      20      20 rw---    [ anon ]
0000003ac1200000       8       4       0 r-x--  libdl-2.5.so
0000003ac1202000    2048       0       0 -----  libdl-2.5.so
0000003ac1402000       4       4       4 r----  libdl-2.5.so
0000003ac1403000       4       4       4 rw---  libdl-2.5.so
0000003ac1600000      84       0       0 r-x--  libselinux.so.1
0000003ac1615000    2048       0       0 -----  libselinux.so.1
0000003ac1815000       8       8       8 rw---  libselinux.so.1
0000003ac1817000       4       4       4 rw---    [ anon ]
0000003ac1a00000     236       0       0 r-x--  libsepol.so.1
0000003ac1a3b000    2048       0       0 -----  libsepol.so.1
0000003ac1c3b000       4       4       4 rw---  libsepol.so.1
0000003ac1c3c000      40       0       0 rw---    [ anon ]
0000003ac1e00000      88      44       0 r-x--  libpthread-2.5.so
0000003ac1e16000    2048       0       0 -----  libpthread-2.5.so
0000003ac2016000       4       4       4 r----  libpthread-2.5.so
0000003ac2017000       4       4       4 rw---  libpthread-2.5.so
0000003ac2018000      16       4       4 rw---    [ anon ]
0000003ac2600000      80      52       0 r-x--  libz.so.1.2.3
0000003ac2614000    2044       0       0 -----  libz.so.1.2.3
0000003ac2813000       4       4       4 rw---  libz.so.1.2.3
0000003ac2a00000      36       4       0 r-x--  libcrypt-2.5.so
0000003ac2a09000    2044       0       0 -----  libcrypt-2.5.so
0000003ac2c08000       4       4       4 r----  libcrypt-2.5.so
0000003ac2c09000       4       4       4 rw---  libcrypt-2.5.so
0000003ac2c0a000     184       0       0 rw---    [ anon ]
0000003ac3600000       8       0       0 r-x--  libkeyutils-1.2.so
0000003ac3602000    2044       0       0 -----  libkeyutils-1.2.so
0000003ac3801000       4       4       4 rw---  libkeyutils-1.2.so
0000003ac3a00000      68       0       0 r-x--  libresolv-2.5.so
0000003ac3a11000    2048       0       0 -----  libresolv-2.5.so
0000003ac3c11000       4       4       4 r----  libresolv-2.5.so
0000003ac3c12000       4       4       4 rw---  libresolv-2.5.so
0000003ac3c13000       8       0       0 rw---    [ anon ]
0000003ac3e00000       8       0       0 r-x--  libcom_err.so.2.1
0000003ac3e02000    2044       0       0 -----  libcom_err.so.2.1
0000003ac4001000       4       4       4 rw---  libcom_err.so.2.1
0000003ac4200000    1204       8       0 r-x--  libcrypto.so.0.9.8e
0000003ac432d000    2044       0       0 -----  libcrypto.so.0.9.8e
0000003ac452c000     132      88      12 rw---  libcrypto.so.0.9.8e
0000003ac454d000      16      12      12 rw---    [ anon ]
0000003ac4600000     176       0       0 r-x--  libgssapi_krb5.so.2.2
0000003ac462c000    2048       0       0 -----  libgssapi_krb5.so.2.2
0000003ac482c000       8       8       8 rw---  libgssapi_krb5.so.2.2
0000003ac4a00000     144       0       0 r-x--  libk5crypto.so.3.1
0000003ac4a24000    2044       0       0 -----  libk5crypto.so.3.1
0000003ac4c23000       8       8       8 rw---  libk5crypto.so.3.1
0000003ac4e00000      32       0       0 r-x--  libkrb5support.so.0.1
0000003ac4e08000    2044       0       0 -----  libkrb5support.so.0.1
0000003ac5007000       4       4       4 rw---  libkrb5support.so.0.1
0000003ac5200000     580       0       0 r-x--  libkrb5.so.3.3
0000003ac5291000    2048       0       0 -----  libkrb5.so.3.3
0000003ac5491000      16      16      12 rw---  libkrb5.so.3.3
0000003ac5a00000     288       4       0 r-x--  libssl.so.0.9.8e
0000003ac5a48000    2048       0       0 -----  libssl.so.0.9.8e
0000003ac5c48000      24      16      12 rw---  libssl.so.0.9.8e
00002b5751808000       8       8       8 rw---    [ anon ]
00002b5751810000     108      36       0 r-x--  libpcre.so.1.2.0
00002b575182b000    2044       0       0 -----  libpcre.so.1.2.0
00002b5751a2a000       4       4       4 rw---  libpcre.so.1.2.0
00002b5751a2b000      28      28      28 rw---    [ anon ]
00002b5751a32000      40      20       0 r-x--  libnss_files-2.5.so
00002b5751a3c000    2044       0       0 -----  libnss_files-2.5.so
00002b5751c3b000       4       4       4 r----  libnss_files-2.5.so
00002b5751c3c000       4       4       4 rw---  libnss_files-2.5.so
00002b5751c3d000       4       4       4 rw-s-  zero (deleted)
00002b5751c3e000   20012   20000   20000 rw---    [ anon ]
00007fffbf2ce000      84      20      20 rw---    [ stack ]
00007fffbf35e000      12       0       0 r-x--    [ anon ]
ffffffffff600000    8192       0       0 -----    [ anon ]
----------------  ------  ------  ------
total kB           72880   22940   22000
```

# vmstat

显示虚拟内存状态

## 补充说明

**vmstat命令** 的含义为显示虚拟内存状态（“Viryual Memor Statics”），但是它可以报告关于进程、内存、I/O等系统整体运行状态。

### 语法

```shell
vmstat(选项)(参数)
```

### 选项

```shell
-a：显示活动内页；
-f：显示启动后创建的进程总数；
-m：显示slab信息；
-n：头信息仅显示一次；
-s：以表格方式显示事件计数器和内存状态；
-d：报告磁盘状态；
-p：显示指定的硬盘分区状态；
-S：输出信息的单位。
```

### 参数

- 事件间隔：状态信息刷新的时间间隔；
- 次数：显示报告的次数。

### 实例

```shell
vmstat 3
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0    320  42188 167332 1534368    0    0     4     7    1    0  0  0 99  0  0
 0  0    320  42188 167332 1534392    0    0     0     0 1002   39  0  0 100  0  0
 0  0    320  42188 167336 1534392    0    0     0    19 1002   44  0  0 100  0  0
 0  0    320  42188 167336 1534392    0    0     0     0 1002   41  0  0 100  0  0
 0  0    320  42188 167336 1534392    0    0     0     0 1002   41  0  0 100  0  0
```

**字段说明：**

Procs（进程）

- r: 运行队列中进程数量，这个值也可以判断是否需要增加CPU。（长期大于1）
- b: 等待IO的进程数量。

Memory（内存）

- swpd: 使用虚拟内存大小，如果swpd的值不为0，但是SI，SO的值长期为0，这种情况不会影响系统性能。
- free: 空闲物理内存大小。
- buff: 用作缓冲的内存大小。
- cache: 用作缓存的内存大小，如果cache的值大的时候，说明cache处的文件数多，如果频繁访问到的文件都能被cache处，那么磁盘的读IO bi会非常小。

Swap

- si: 每秒从交换区写到内存的大小，由磁盘调入内存。
- so: 每秒写入交换区的内存大小，由内存调入磁盘。

注意：内存够用的时候，这2个值都是0，如果这2个值长期大于0时，系统性能会受到影响，磁盘IO和CPU资源都会被消耗。有些朋友看到空闲内存（free）很少的或接近于0时，就认为内存不够用了，不能光看这一点，还要结合si和so，如果free很少，但是si和so也很少（大多时候是0），那么不用担心，系统性能这时不会受到影响的。

IO（现在的Linux版本块的大小为1kb）

- bi: 每秒读取的块数
- bo: 每秒写入的块数

注意：随机磁盘读写的时候，这2个值越大（如超出1024k)，能看到CPU在IO等待的值也会越大。

system（系统）

- in: 每秒中断数，包括时钟中断。
- cs: 每秒上下文切换数。

注意：上面2个值越大，会看到由内核消耗的CPU时间会越大。

CPU（以百分比表示）

- us: 用户进程执行时间百分比(user time)

us的值比较高时，说明用户进程消耗的CPU时间多，但是如果长期超50%的使用，那么我们就该考虑优化程序算法或者进行加速。

- sy: 内核系统进程执行时间百分比(system time)

sy的值高时，说明系统内核消耗的CPU资源多，这并不是良性表现，我们应该检查原因。

- wa: IO等待时间百分比

wa的值高时，说明IO等待比较严重，这可能由于磁盘大量作随机访问造成，也有可能磁盘出现瓶颈（块操作）。

- id: 空闲时间百分比

# systemctl

系统服务管理器指令

## 补充说明

**systemctl命令** 是系统服务管理器指令，它实际上将 service 和 chkconfig 这两个命令组合到一起。

| 任务                 | 旧指令                        | 新指令                                                       |
| -------------------- | ----------------------------- | ------------------------------------------------------------ |
| 使某服务自动启动     | chkconfig --level 3 httpd on  | systemctl enable httpd.service                               |
| 使某服务不自动启动   | chkconfig --level 3 httpd off | systemctl disable httpd.service                              |
| 检查服务状态         | service httpd status          | systemctl status httpd.service （服务详细信息） systemctl is-active httpd.service （仅显示是否 Active) |
| 显示所有已启动的服务 | chkconfig --list              | systemctl list-units --type=service                          |
| 启动服务             | service httpd start           | systemctl start httpd.service                                |
| 停止服务             | service httpd stop            | systemctl stop httpd.service                                 |
| 重启服务             | service httpd restart         | systemctl restart httpd.service                              |
| 重载服务             | service httpd reload          | systemctl reload httpd.service                               |

### 实例

```shell
systemctl start nfs-server.service . # 启动nfs服务
systemctl enable nfs-server.service # 设置开机自启动
systemctl enable nfs-server.service --now # 设置开机自启动，并立刻启动
systemctl disable nfs-server.service # 停止开机自启动
systemctl disable nfs-server.service --now # 停止开机自启动，并立刻停止
systemctl status nfs-server.service # 查看服务当前状态
systemctl restart nfs-server.service # 重新启动某服务
systemctl list-units --type=service # 查看所有已启动的服务
```

开启防火墙22端口

```shell
iptables -I INPUT -p tcp --dport 22 -j accept
```

如果仍然有问题，就可能是SELinux导致的

关闭SElinux：

修改`/etc/selinux/config`文件中的`SELINUX=""`为disabled，然后重启。

彻底关闭防火墙：

```shell
sudo systemctl status firewalld.service
sudo systemctl stop firewalld.service          
sudo systemctl disable firewalld.service
```

# nstat

nstat 是一个简单的监视内核的 SNMP 计数器和网络接口状态的实用工具。

## 补充说明

大多数命令行用户都熟悉 netstat ，这是 net-tools 软件包中的命令。目前新版本中 net-tools 软件包几乎完全被弃用，取而代之的是 ip 命令套件，而 nstat 属于新软件包。

### 语法

```s
nstat [OPTION] [ PATTERN [ PATTERN ] ]
```

### 选项

```shell
-h：显示帮助信息；
-V：显示指令版本信息；
-z：转储零计数器。默认情况下不显示它们；
-r：清零历史统计；
-n：不显示任何内容，仅更新历史；
-a：显示计数器的绝对值；
-d：以守护进程模式运行并收集统计数据
-s：不更新历史；
-j：JSON格式输出。
```

### 实例

直接输入以查询网络接口状态，以下展示了 IPv4，IPv6，TCP，UDP，ICMP 的统计数据：

```shell
nstat          
#kernel
IpInReceives                    769152             0.0
IpInAddrErrors                  1                  0.0
IpInDelivers                    769146             0.0
IpOutRequests                   764236             0.0
IpOutDiscards                   20                 0.0
IpOutNoRoutes                   1                  0.0
IcmpInMsgs                      92                 0.0
IcmpInDestUnreachs              92                 0.0
IcmpOutMsgs                     94                 0.0
IcmpOutDestUnreachs             94                 0.0
IcmpMsgInType3                  92                 0.0
IcmpMsgOutType3                 94                 0.0
TcpActiveOpens                  1786               0.0
TcpPassiveOpens                 142                0.0
TcpAttemptFails                 11                 0.0
TcpEstabResets                  72                 0.0
TcpInSegs                       756827             0.0
TcpOutSegs                      802908             0.0
TcpRetransSegs                  767                0.0
TcpOutRsts                      702                0.0
UdpInDatagrams                  12075              0.0
UdpNoPorts                      82                 0.0
UdpOutDatagrams                 7045               0.0
UdpIgnoredMulti                 70                 0.0
Ip6InReceives                   5005               0.0
Ip6InDelivers                   5005               0.0
Ip6OutRequests                  131                0.0
Ip6OutDiscards                  2                  0.0
Ip6OutNoRoutes                  959                0.0
Ip6InMcastPkts                  4999               0.0
Ip6OutMcastPkts                 125                0.0
Ip6InOctets                     797462             0.0
Ip6OutOctets                    16421              0.0
Ip6InMcastOctets                797030             0.0
Ip6OutMcastOctets               15949              0.0
Ip6InNoECTPkts                  5005               0.0
Icmp6InMsgs                     3                  0.0
Icmp6OutMsgs                    51                 0.0
Icmp6InNeighborAdvertisements   1                  0.0
Icmp6InMLDv2Reports             2                  0.0
Icmp6OutRouterSolicits          11                 0.0
Icmp6OutNeighborSolicits        4                  0.0
Icmp6OutMLDv2Reports            36                 0.0
Icmp6InType136                  1                  0.0
Icmp6InType143                  2                  0.0
Icmp6OutType133                 11                 0.0
Icmp6OutType135                 4                  0.0
Icmp6OutType143                 36                 0.0
Udp6InDatagrams                 4998               0.0
Udp6OutDatagrams                76                 0.0
TcpExtTW                        385                0.0
TcpExtPAWSEstab                 1                  0.0
TcpExtDelayedACKs               37133              0.0
TcpExtDelayedACKLocked          57                 0.0
TcpExtDelayedACKLost            456                0.0
TcpExtTCPHPHits                 417717             0.0
TcpExtTCPPureAcks               34186              0.0
TcpExtTCPHPAcks                 222980             0.0
TcpExtTCPSACKReorder            1                  0.0
TcpExtTCPLossUndo               194                0.0
TcpExtTCPLostRetransmit         169                0.0
TcpExtTCPSlowStartRetrans       1                  0.0
TcpExtTCPTimeouts               494                0.0
TcpExtTCPLossProbes             309                0.0
TcpExtTCPBacklogCoalesce        571                0.0
TcpExtTCPDSACKOldSent           281                0.0
TcpExtTCPDSACKRecv              281                0.0
TcpExtTCPAbortOnData            13                 0.0
TcpExtTCPAbortOnClose           30                 0.0
TcpExtTCPDSACKIgnoredOld        1                  0.0
TcpExtTCPDSACKIgnoredNoUndo     258                0.0
TcpExtTCPSackShiftFallback      1                  0.0
TcpExtTCPRcvCoalesce            18314              0.0
TcpExtTCPFastOpenActiveFail     2                  0.0
TcpExtTCPSpuriousRtxHostQueues  11                 0.0
TcpExtTCPAutoCorking            1684               0.0
TcpExtTCPFromZeroWindowAdv      2                  0.0
TcpExtTCPToZeroWindowAdv        2                  0.0
TcpExtTCPSynRetrans             479                0.0
TcpExtTCPOrigDataSent           359814             0.0
TcpExtTCPHystartTrainDetect     13                 0.0
TcpExtTCPHystartTrainCwnd       550                0.0
TcpExtTCPKeepAlive              18                 0.0
TcpExtTCPDelivered              361695             0.0
TcpExtTCPZeroWindowDrop         1                  0.0
TcpExtTcpTimeoutRehash          494                0.0
TcpExtTcpDuplicateDataRehash    2                  0.0
TcpExtTCPDSACKRecvSegs          281                0.0
IpExtInNoRoutes                 3                  0.0
IpExtInMcastPkts                5392               0.0
IpExtOutMcastPkts               221                0.0
IpExtInBcastPkts                70                 0.0
IpExtOutBcastPkts               10                 0.0
IpExtInOctets                   2100280442         0.0
IpExtOutOctets                  226760631          0.0
IpExtInMcastOctets              746608             0.0
IpExtOutMcastOctets             27565              0.0
IpExtInBcastOctets              5674               0.0
IpExtOutBcastOctets             778                0.0
IpExtInNoECTPkts                1885871            0.0
```

# telnet

登录远程主机和管理(测试ip端口是否连通)

## 补充说明

**telnet命令** 用于登录远程主机，对远程主机进行管理。telnet因为采用明文传送报文，安全性不好，很多Linux服务器都不开放telnet服务，而改用更安全的ssh方式了。但仍然有很多别的系统可能采用了telnet方式来提供远程登录，因此弄清楚telnet客户端的使用方式仍是很有必要的。

### 语法

```shell
telnet(选项)(参数)
```

### 选项

```shell
-8：允许使用8位字符资料，包括输入与输出；
-a：尝试自动登入远端系统；
-b<主机别名>：使用别名指定远端主机名称；
-c：不读取用户专属目录里的.telnetrc文件；
-d：启动排错模式；
-e<脱离字符>：设置脱离字符；
-E：滤除脱离字符；
-f：此参数的效果和指定"-F"参数相同；
-F：使用Kerberos V5认证时，加上此参数可把本地主机的认证数据上传到远端主机；
-k<域名>：使用Kerberos认证时，加上此参数让远端主机采用指定的领域名，而非该主机的域名；
-K：不自动登入远端主机；
-l<用户名称>：指定要登入远端主机的用户名称；
-L：允许输出8位字符资料；
-n<记录文件>：指定文件记录相关信息；
-r：使用类似rlogin指令的用户界面；
-S<服务类型>：设置telnet连线所需的ip TOS信息；
-x：假设主机有支持数据加密的功能，就使用它；
-X<认证形态>：关闭指定的认证形态。
```

### 参数

- 远程主机：指定要登录进行管理的远程主机；
- 端口：指定TELNET协议使用的端口号。

### 实例

```shell
$ telnet 192.168.2.10
Trying 192.168.2.10...
Connected to 192.168.2.10 (192.168.2.10).
Escape character is '^]'.

    localhost (Linux release 2.6.18-274.18.1.el5 #1 SMP Thu Feb 9 12:45:44 EST 2012) (1)

login: root
Password:
Login incorrect
```

一般情况下不允许root从远程登录，可以先用普通账号登录，然后再用su -切到root用户。

```shell
$ telnet 192.168.188.132
Trying 192.168.188.132...
telnet: connect to address 192.168.188.132: Connection refused
telnet: Unable to connect to remote host
```

处理这种情况方法：

1. 确认ip地址是否正确？
2. 确认ip地址对应的主机是否已经开机？
3. 如果主机已经启动，确认路由设置是否设置正确？（使用route命令查看）
4. 如果主机已经启动，确认主机上是否开启了telnet服务？（使用netstat命令查看，TCP的23端口是否有LISTEN状态的行）
5. 如果主机已经启动telnet服务，确认防火墙是否放开了23端口的访问？（使用iptables-save查看）

**启动telnet服务**

```shell
service xinetd restart
```

配置参数，通常的配置如下：

```shell
service telnet
{
    disable = no #启用
    flags = REUSE #socket可重用
    socket_type = stream #连接方式为TCP
    wait = no #为每个请求启动一个进程
    user = root #启动服务的用户为root
    server = /usr/sbin/in.telnetd #要激活的进程
    log_on_failure += USERID #登录失败时记录登录用户名
}
```

如果要配置允许登录的客户端列表，加入

```
only_from = 192.168.0.2 #只允许192.168.0.2登录
```

如果要配置禁止登录的客户端列表，加入

```
no_access = 192.168.0.{2,3,4} #禁止192.168.0.2、192.168.0.3、192.168.0.4登录
```

如果要设置开放时段，加入

```
access_times = 9:00-12:00 13:00-17:00 # 每天只有这两个时段开放服务（我们的上班时间：P）
```

如果你有两个IP地址，一个是私网的IP地址如192.168.0.2，一个是公网的IP地址如218.75.74.83，如果你希望用户只能从私网来登录telnet服务，那么加入

```
bind = 192.168.0.2
```

各配置项具体的含义和语法可参考xined配置文件属性说明（man xinetd.conf）

配置端口，修改services文件：

```shell
# vi /etc/services
```

找到以下两句

```shell
telnet 23/tcp
telnet 23/udp
```

如果前面有#字符，就去掉它。telnet的默认端口是23，这个端口也是黑客端口扫描的主要对象，因此最好将这个端口修改掉，修改的方法很简单，就是将23这个数字修改掉，改成大一点的数字，比如61123。注意，1024以下的端口号是internet保留的端口号，因此最好不要用，还应该注意不要与其它服务的端口冲突。

启动服务：

```
service xinetd restart
```

# ip

网络配置工具

## 补充说明

**ip命令** 用来显示或操纵Linux主机的路由、网络设备、策略路由和隧道，是Linux下较新的功能强大的网络配置工具。

### 语法

```shell
ip(选项)(对象)
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
```

### 对象

```shell
OBJECT := { link | address | addrlabel | route | rule | neigh | ntable |
       tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm |
       netns | l2tp | macsec | tcp_metrics | token }
       
-V：显示指令版本信息；
-s：输出更详细的信息；
-f：强制使用指定的协议族；
-4：指定使用的网络层协议是IPv4协议；
-6：指定使用的网络层协议是IPv6协议；
-0：输出信息每条记录输出一行，即使内容较多也不换行显示；
-r：显示主机时，不使用IP地址，而使用主机的域名。
```

### 选项

```shell
OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
        -h[uman-readable] | -iec |
        -f[amily] { inet | inet6 | ipx | dnet | bridge | link } |
        -4 | -6 | -I | -D | -B | -0 |
        -l[oops] { maximum-addr-flush-attempts } |
        -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] [filename] |
        -rc[vbuf] [size] | -n[etns] name | -a[ll] }
        
网络对象：指定要管理的网络对象；
具体操作：对指定的网络对象完成具体操作；
help：显示网络对象支持的操作命令的帮助信息。
```

### 实例

```shell
ip link show                    # 显示网络接口信息
ip link set eth0 up             # 开启网卡
ip link set eth0 down            # 关闭网卡
ip link set eth0 promisc on      # 开启网卡的混合模式
ip link set eth0 promisc offi    # 关闭网卡的混合模式
ip link set eth0 txqueuelen 1200 # 设置网卡队列长度
ip link set eth0 mtu 1400        # 设置网卡最大传输单元
ip addr show     # 显示网卡IP信息
ip addr add 192.168.0.1/24 dev eth0 # 为eth0网卡添加一个新的IP地址192.168.0.1
ip addr del 192.168.0.1/24 dev eth0 # 为eth0网卡删除一个IP地址192.168.0.1

ip route show # 显示系统路由
ip route add default via 192.168.1.254   # 设置系统默认路由
ip route list                 # 查看路由信息
ip route add 192.168.4.0/24  via  192.168.0.254 dev eth0 # 设置192.168.4.0网段的网关为192.168.0.254,数据走eth0接口
ip route add default via  192.168.0.254  dev eth0        # 设置默认网关为192.168.0.254
ip route del 192.168.4.0/24   # 删除192.168.4.0网段的网关
ip route del default          # 删除默认路由
ip route delete 192.168.1.0/24 dev eth0 # 删除路由
```

**用ip命令显示网络设备的运行状态**

```shell
[root@localhost ~]# ip link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 00:16:3e:00:1e:51 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 00:16:3e:00:1e:52 brd ff:ff:ff:ff:ff:ff
```

**显示更加详细的设备信息**

```shell
[root@localhost ~]# ip -s link list
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    RX: bytes  packets  errors  dropped overrun mcast   
    5082831    56145    0       0       0       0      
    TX: bytes  packets  errors  dropped carrier collsns
    5082831    56145    0       0       0       0      
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 00:16:3e:00:1e:51 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    3641655380 62027099 0       0       0       0      
    TX: bytes  packets  errors  dropped carrier collsns
    6155236    89160    0       0       0       0      
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 00:16:3e:00:1e:52 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    2562136822 488237847 0       0       0       0      
    TX: bytes  packets  errors  dropped carrier collsns
    3486617396 9691081  0       0       0       0     
```

**显示核心路由表**

```shell
[root@localhost ~]# ip route list 
112.124.12.0/22 dev eth1  proto kernel  scope link  src 112.124.15.130
10.160.0.0/20 dev eth0  proto kernel  scope link  src 10.160.7.81
192.168.0.0/16 via 10.160.15.247 dev eth0
172.16.0.0/12 via 10.160.15.247 dev eth0
10.0.0.0/8 via 10.160.15.247 dev eth0
default via 112.124.15.247 dev eth1
```

**显示邻居表**

```shell
[root@localhost ~]# ip neigh list
112.124.15.247 dev eth1 lladdr 00:00:0c:9f:f3:88 REACHABLE
10.160.15.247 dev eth0 lladdr 00:00:0c:9f:f2:c0 STALE
```

**获取主机所有网络接口**

```shell
ip link | grep -E '^[0-9]' | awk -F: '{print $2}'
```

# ss

比 netstat 好用的socket统计信息，iproute2 包附带的另一个工具，允许你查询 socket 的有关统计信息

## 补充说明

**ss命令** 用来显示处于活动状态的套接字信息。ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。

当服务器的socket连接数量变得非常大时，无论是使用netstat命令还是直接`cat /proc/net/tcp`，执行速度都会很慢。可能你不会有切身的感受，但请相信我，当服务器维持的连接达到上万个的时候，使用netstat等于浪费 生命，而用ss才是节省时间。

天下武功唯快不破。ss快的秘诀在于，它利用到了TCP协议栈中tcp_diag。tcp_diag是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了ss的快捷高效。当然，如果你的系统中没有tcp_diag，ss也可以正常运行，只是效率会变得稍慢。

### 语法

```shell
ss [参数]
ss [参数] [过滤]
```

### 选项

```shell
-h, --help      帮助信息
-V, --version   程序版本信息
-n, --numeric   不解析服务名称
-r, --resolve   解析主机名
-a, --all       显示所有套接字（sockets）
-l, --listening 显示监听状态的套接字（sockets）
-o, --options   显示计时器信息
-e, --extended  显示详细的套接字（sockets）信息
-m, --memory    显示套接字（socket）的内存使用情况
-p, --processes 显示使用套接字（socket）的进程
-i, --info      显示 TCP内部信息
-s, --summary   显示套接字（socket）使用概况
-4, --ipv4      仅显示IPv4的套接字（sockets）
-6, --ipv6      仅显示IPv6的套接字（sockets）
-0, --packet    显示 PACKET 套接字（socket）
-t, --tcp       仅显示 TCP套接字（sockets）
-u, --udp       仅显示 UCP套接字（sockets）
-d, --dccp      仅显示 DCCP套接字（sockets）
-w, --raw       仅显示 RAW套接字（sockets）
-x, --unix      仅显示 Unix套接字（sockets）
-f, --family=FAMILY  显示 FAMILY类型的套接字（sockets），FAMILY可选，支持  unix, inet, inet6, link, netlink
-A, --query=QUERY, --socket=QUERY
      QUERY := {all|inet|tcp|udp|raw|unix|packet|netlink}[,QUERY]
-D, --diag=FILE     将原始TCP套接字（sockets）信息转储到文件
 -F, --filter=FILE  从文件中都去过滤器信息
       FILTER := [ state TCP-STATE ] [ EXPRESSION ]
```

### 实例

```shell
ss -t -a    # 显示TCP连接
ss -s       # 显示 Sockets 摘要
ss -l       # 列出所有打开的网络连接端口
ss -pl      # 查看进程使用的socket
ss -lp | grep 3306  # 找出打开套接字/端口应用程序
ss -u -a    显示所有UDP Sockets
ss -o state established '( dport = :smtp or sport = :smtp )' # 显示所有状态为established的SMTP连接
ss -o state established '( dport = :http or sport = :http )' # 显示所有状态为Established的HTTP连接
ss -o state fin-wait-1 '( sport = :http or sport = :https )' dst 193.233.7/24  # 列举出处于 FIN-WAIT-1状态的源端口为 80或者 443，目标网络为 193.233.7/24所有 tcp套接字

# ss 和 netstat 效率对比
time netstat -at
time ss

# 匹配远程地址和端口号
# ss dst ADDRESS_PATTERN
ss dst 192.168.1.5
ss dst 192.168.119.113:http
ss dst 192.168.119.113:smtp
ss dst 192.168.119.113:443

# 匹配本地地址和端口号
# ss src ADDRESS_PATTERN
ss src 192.168.119.103
ss src 192.168.119.103:http
ss src 192.168.119.103:80
ss src 192.168.119.103:smtp
ss src 192.168.119.103:25
```

**将本地或者远程端口和一个数比较**

```shell
# ss dport OP PORT 远程端口和一个数比较；
# ss sport OP PORT 本地端口和一个数比较
# OP 可以代表以下任意一个:
# <= or le : 小于或等于端口号
# >= or ge : 大于或等于端口号
# == or eq : 等于端口号
# != or ne : 不等于端口号
# < or gt : 小于端口号
# > or lt : 大于端口号
ss  sport = :http
ss  dport = :http
ss  dport \> :1024
ss  sport \> :1024
ss sport \< :32000
ss  sport eq :22
ss  dport != :22
ss  state connected sport = :http
ss \( sport = :http or sport = :https \)
ss -o state fin-wait-1 \( sport = :http or sport = :https \) dst 192.168.1/24
```

**用TCP 状态过滤Sockets**

```shell
ss -4 state closing
# ss -4 state FILTER-NAME-HERE
# ss -6 state FILTER-NAME-HERE
# FILTER-NAME-HERE 可以代表以下任何一个：
# established、 syn-sent、 syn-recv、 fin-wait-1、 fin-wait-2、 time-wait、 closed、 close-wait、 last-ack、 listen、 closing、
# all : 所有以上状态
# connected : 除了listen and closed的所有状态
# synchronized :所有已连接的状态除了syn-sent
# bucket : 显示状态为maintained as minisockets,如：time-wait和syn-recv.
# big : 和bucket相反.
```

**显示ICP连接**

```shell
[root@localhost ~]# ss -t -a
State       Recv-Q Send-Q                            Local Address:Port                                Peer Address:Port
LISTEN      0      0                                             *:3306                                           *:*
LISTEN      0      0                                             *:http                                           *:*
LISTEN      0      0                                             *:ssh                                            *:*
LISTEN      0      0                                     127.0.0.1:smtp                                           *:*
ESTAB       0      0                                112.124.15.130:42071                              42.156.166.25:http
ESTAB       0      0                                112.124.15.130:ssh                              121.229.196.235:33398
```

**显示 Sockets 摘要**

```shell
[root@localhost ~]# ss -s
Total: 172 (kernel 189)
TCP:   10 (estab 2, closed 4, orphaned 0, synrecv 0, timewait 0/0), ports 5

Transport Total     ip        IPv6
*         189       -         -
RAW       0         0         0
UDP       5         5         0
TCP       6         6         0
INET      11        11        0
FRAG      0         0         0
```

列出当前的established, closed, orphaned and waiting TCP sockets

**列出所有打开的网络连接端口**

```shell
[root@localhost ~]# ss -l
Recv-Q Send-Q                                 Local Address:Port                                     Peer Address:Port
0      0                                                  *:3306                                                *:*
0      0                                                  *:http                                                *:*
0      0                                                  *:ssh                                                 *:*
0      0                                          127.0.0.1:smtp                                                *:*
```

**查看进程使用的socket**

```shell
[root@localhost ~]# ss -pl
Recv-Q Send-Q                                          Local Address:Port                                              Peer Address:Port
0      0                                                           *:3306                                                         *:*        users:(("mysqld",1718,10))
0      0                                                           *:http                                                         *:*        users:(("nginx",13312,5),("nginx",13333,5))
0      0                                                           *:ssh                                                          *:*        users:(("sshd",1379,3))
0      0                                                   127.0.0.1:smtp                                                         *:*        us
```

**找出打开套接字/端口应用程序**

```shell
[root@localhost ~]# ss -pl | grep 3306
0      0                            *:3306                          *:*        users:(("mysqld",1718,10))
```

**显示所有UDP Sockets**

```shell
[root@localhost ~]# ss -u -a
State       Recv-Q Send-Q                                     Local Address:Port                                         Peer Address:Port
UNCONN      0      0                                                      *:syslog                                                  *:*
UNCONN      0      0                                         112.124.15.130:ntp                                                     *:*
UNCONN      0      0                                            10.160.7.81:ntp                                                     *:*
UNCONN      0      0                                              127.0.0.1:ntp                                                     *:*
UNCONN      0      0                                                      *:ntp                                                     *:*
```

**出所有端口为 22（ssh）的连接**

```shell
[root@localhost ~]# ss state all sport = :ssh
Netid State      Recv-Q Send-Q     Local Address:Port                      Peer Address:Port
tcp   LISTEN     0      128                    *:ssh                                  *:*
tcp   ESTAB      0      0          192.168.0.136:ssh                      192.168.0.102:46540
tcp   LISTEN     0      128                   :::ssh                                 :::*
```

**查看TCP的连接状态**

```shell
[root@localhost ~]# ss  -tan|awk 'NR>1{++S[$1]}END{for (a in S) print a,S[a]}'
LISTEN 7
ESTAB 31
TIME-WAIT 28
```

# lsof

显示Linux系统当前已打开的所有文件列表 `lsof -p pid`

## 补充说明

**lsof命令** 用于查看你进程打开的文件，打开文件的进程，进程打开的端口(TCP、UDP)。找回/恢复删除的文件。是十分方便的系统监视工具，因为lsof命令需要访问核心内存和各种文件，所以需要root用户执行。

在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，因此通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的。

### 语法

```shell
lsof (选项)
```

### 选项

```shell
-a：列出打开文件存在的进程；
-c<进程名>：列出指定进程所打开的文件；
-g：列出GID号进程详情；
-d<文件号>：列出占用该文件号的进程；
+d<目录>：列出目录下被打开的文件；
+D<目录>：递归列出目录下被打开的文件；
-n<目录>：列出使用NFS的文件；
-i<条件>：列出符合条件的进程（协议、:端口、 @ip ）
-p<进程号>：列出指定进程号所打开的文件；
-u：列出UID号进程详情；
-h：显示帮助信息；
-v：显示版本信息
```

### 实例

```shell
lsof
command     PID USER   FD      type             DEVICE     SIZE       NODE NAME
init          1 root  cwd       DIR                8,2     4096          2 /
init          1 root  rtd       DIR                8,2     4096          2 /
init          1 root  txt       REG                8,2    43496    6121706 /sbin/init
init          1 root  mem       REG                8,2   143600    7823908 /lib64/ld-2.5.so
init          1 root  mem       REG                8,2  1722304    7823915 /lib64/libc-2.5.so
init          1 root  mem       REG                8,2    23360    7823919 /lib64/libdl-2.5.so
init          1 root  mem       REG                8,2    95464    7824116 /lib64/libselinux.so.1
init          1 root  mem       REG                8,2   247496    7823947 /lib64/libsepol.so.1
init          1 root   10u     FIFO               0,17                1233 /dev/initctl
migration     2 root  cwd       DIR                8,2     4096          2 /
migration     2 root  rtd       DIR                8,2     4096          2 /
migration     2 root  txt   unknown                                        /proc/2/exe
ksoftirqd     3 root  cwd       DIR                8,2     4096          2 /
ksoftirqd     3 root  rtd       DIR                8,2     4096          2 /
ksoftirqd     3 root  txt   unknown                                        /proc/3/exe
migration     4 root  cwd       DIR                8,2     4096          2 /
migration     4 root  rtd       DIR                8,2     4096          2 /
migration     4 root  txt   unknown                                        /proc/4/exe
ksoftirqd     5 root  cwd       DIR                8,2     4096          2 /
ksoftirqd     5 root  rtd       DIR                8,2     4096          2 /
ksoftirqd     5 root  txt   unknown                                        /proc/5/exe
events/0      6 root  cwd       DIR                8,2     4096          2 /
events/0      6 root  rtd       DIR                8,2     4096          2 /
events/0      6 root  txt   unknown                                        /proc/6/exe
events/1      7 root  cwd       DIR                8,2     4096          2 /
```

**lsof输出各列信息的意义如下：**

| 标识      | 说明                                 |
| --------- | ------------------------------------ |
| `COMMAND` | 进程的名称                           |
| `PID`     | 进程标识符                           |
| `PPID`    | 父进程标识符（需要指定-R参数）       |
| `USER`    | 进程所有者                           |
| `PGID`    | 进程所属组                           |
| `FD`      | 文件描述符，应用程序通过它识别该文件 |

文件描述符列表：

| 标识   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `cwd`  | 表示当前工作目录，即：应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改 |
| `txt`  | 该类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序 |
| `lnn`  | 库引用 (AIX);                                                |
| `er`   | FD 信息错误（参见名称栏）                                    |
| `jld`  | jail 目录 (FreeBSD);                                         |
| `ltx`  | 共享库文本（代码和数据）                                     |
| `mxx`  | 十六进制内存映射类型编号xx                                   |
| `m86`  | DOS合并映射文件                                              |
| `mem`  | 内存映射文件                                                 |
| `mmap` | 内存映射设备                                                 |
| `pd`   | 父目录                                                       |
| `rtd`  | 根目录                                                       |
| `tr`   | 内核跟踪文件 (OpenBSD)                                       |
| `v86`  | VP/ix 映射文件                                               |
| `0`    | 表示标准输出                                                 |
| `1`    | 表示标准输入                                                 |
| `2`    | 表示标准错误                                                 |

一般在标准输出、标准错误、标准输入后还跟着文件状态模式：

| 标识   | 说明                                      |
| ------ | ----------------------------------------- |
| `u`    | 表示该文件被打开并处于读取/写入模式       |
| `r`    | 表示该文件被打开并处于只读模式            |
| `w`    | 表示该文件被打开并处于写入模式            |
| `空格` | 表示该文件的状态模式为 unknow，且没有锁定 |
| `-`    | 表示该文件的状态模式为 unknow，且被锁定   |

同时在文件状态模式后面，还跟着相关的锁：

| 标识    | 说明                                     |
| ------- | ---------------------------------------- |
| `N`     | 对于未知类型的Solaris NFS锁              |
| `r`     | 用于部分文件的读取锁定                   |
| `R`     | 对整个文件进行读取锁定                   |
| `w`     | 对文件的一部分进行写锁定(文件的部分写锁) |
| `W`     | 对整个文件进行写锁定(整个文件的写锁)     |
| `u`     | 用于任何长度的读写锁                     |
| `U`     | 对于未知类型的锁                         |
| `x`     | 对于文件部分的SCO OpenServer Xenix锁     |
| `X`     | 对于整个文件的SCO OpenServer Xenix锁     |
| `space` | 如果没有锁                               |

**文件类型**

| 标识     | 说明                           |
| -------- | ------------------------------ |
| `DIR`    | 表示目录                       |
| `CHR`    | 表示字符类型                   |
| `BLK`    | 块设备类型                     |
| `UNIX`   | UNIX 域套接字                  |
| `FIFO`   | 先进先出 (FIFO) 队列           |
| `IPv4`   | 网际协议 (IP) 套接字           |
| `DEVICE` | 指定磁盘的名称                 |
| `SIZE`   | 文件的大小                     |
| `NODE`   | 索引节点（文件在磁盘上的标识） |
| `NAME`   | 打开文件的确切名称             |
| `REG`    | 常规文件                       |

列出指定进程号所打开的文件:

```shell
lsof -p $pid
```

获取端口对应的进程ID=>pid

```shell
lsof -i:9981 -P -t -sTCP:LISTEN
```

列出打开文件的进程:

```shell
lsof $filename
```

查看端口占用

```shell
lsof -i:$port
```

**查看所有打开的文件：**

```
lsof
```

**查看指定进程打开的文件：**

```
lsof -p <PID>
```

**查看指定用户打开的文件：**

```
lsof -u <username>
```

**查看指定文件名相关的进程：**

```
lsof <filename>
```

**查看网络连接相关的进程：**

```
lsof -i
```

**查看指定端口相关的进程：**

```
lsof -i :<port>
```

**查看正在使用某个目录的进程：**

```
lsof +D /path/to/directory
```

**查看被删除但仍然被某个进程打开的文件：**

```
lsof -u +L1
```

**查看某个文件系统上被打开的文件：**

```
lsof /mountpoint
```

**以列表形式显示结果：**

```
lsof -F
```

**显示结果中不包含主机名：**

```
lsof -n
```

**显示结果中不包含进程路径：**

```
lsof -b
```

**以逆序显示结果：**

```
lsof -r
```

**以特定间隔时间循环显示结果：**

```
lsof -r <interval>
```

**以持续模式显示结果：**

```
lsof -t <interval>
```

# kill

发送信号到进程。

## 目录

- [bash内建命令](https://wangchujiang.com/linux-command/c/kill.html#内建命令)
- [GNU coreutils中的命令](https://wangchujiang.com/linux-command/c/kill.html#外部命令)

## 内建命令

### 概要

```shell
kill [-s sigspec | -n signum | -sigspec] pid | jobspec ...
kill -l [sigspec]
```

### 主要用途

- 发送信号到作业或进程（可以为多个）。
- 列出信号。

### 选项

```shell
-s sig    信号名称。
-n sig    信号名称对应的数字。
-l        列出信号名称。如果在该选项后提供了数字那么假设它是信号名称对应的数字。
-L        等价于-l选项。
```

### 参数

pid：进程ID

jobspec：作业标识符

### 返回值

返回状态为成功除非给出了非法选项、执行出现错误。

### 例子

```shell
[user2@pc] kill -l 9
KILL

# 列出所有信号名称：
[user2@pc] kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL
 5) SIGTRAP      6) SIGABRT      7) SIGBUS       8) SIGFPE
 9) SIGKILL     10) SIGUSR1     11) SIGSEGV     12) SIGUSR2
13) SIGPIPE     14) SIGALRM     15) SIGTERM     16) SIGSTKFLT
17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU
25) SIGXFSZ     26) SIGVTALRM   27) SIGPROF     28) SIGWINCH
29) SIGIO       30) SIGPWR      31) SIGSYS      34) SIGRTMIN
35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3  38) SIGRTMIN+4
39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12
47) SIGRTMIN+13 48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14
51) SIGRTMAX-13 52) SIGRTMAX-12 53) SIGRTMAX-11 54) SIGRTMAX-10
55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7  58) SIGRTMAX-6
59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX

# 下面是常用的信号。
# 只有第9种信号(SIGKILL)才可以无条件终止进程，其他信号进程都有权利忽略。

HUP     1    终端挂断
INT     2    中断（同 Ctrl + C）
QUIT    3    退出（同 Ctrl + \）
KILL    9    强制终止
TERM   15    终止
CONT   18    继续（与STOP相反，fg/bg命令）
STOP   19    暂停（同 Ctrl + Z）
# 以下发送KILL信号的形式等价。当然还有更多的等价形式，在此不一一列举了。
[user2@pc] kill -s SIGKILL PID
[user2@pc] kill -s KILL PID
[user2@pc] kill -n 9 PID
[user2@pc] kill -9 PID

[user2@pc] sleep 90 &
[1] 178420

# 终止作业标识符为1的作业。
[user2@pc] kill -9 %1

[user2@pc] jobs -l
[1]+ 178420 KILLED                  ssh 192.168.1.4

[user2@pc] sleep 90 &
[1] 181357

# 发送停止信号。
[user2@pc] kill -s STOP 181357

[user2@pc] jobs -l
[1]+ 181537 Stopped (signal)        sleep 90

# 发送继续信号。
[user2@pc] kill -s CONT 181357

[user2@pc] jobs -l
[1]+ 181537 Running                 sleep 90 &
```

### 注意

1. `bash`的作业控制命令包括`bg fg kill wait disown suspend`。
2. 该命令是bash内建命令，相关的帮助信息请查看`help`命令。

## 外部命令

### 概要

```shell
kill [-signal|-s signal|-p] [-q value] [-a] [--] pid|name...
kill -l [number] | -L
```

### 主要用途

- 发送信号到进程（可以为多个）。
- 列出信号。

### 选项

```shell
-s, --signal signal    要发送的信号，可能是信号名称或信号对应的数字。
-l, --list [number]    打印信号名称或转换给定数字到信号名称。信号名称可参考文件（/usr/include/linux/signal.h）。
-L, --table            和'-l'选项类似，但是输出信号名称以及信号对应的数字。
-a, --all              不要限制“命令名到pid”的转换为具有与当前进程相同的UID的进程。
-p, --pid              打印目标进程的PID而不发送信号。
--verbose              打印信号以及接收信号的PID。
-q, --queue value      使用sigqueue(3)而不是kill(2)。参数value是信号对应的数字。
                           如果接收进程已为此信号安装了处理程序将SA_SIGINFO标记为sigaction(2)，则可以获取
                           该数据通过siginfo_t结构的si_sigval字段。
--help                 显示帮助信息并退出。
--version              显示版本信息并退出。
```

### 参数

接收信号的进程列表可以是PID以及name的混合组成。

PID：每一个PID可以是以下四种情况之一：

| 状态 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| n    | 当n大于0时，PID为n的进程接收信号。                           |
| 0    | 当前进程组中的所有进程均接收信号。                           |
| -1   | PID大于1的所有进程均接收信号。                               |
| -n   | 当n大于1时，进程组n中的所有进程接收信号。当给出了一个参数的形式为“-n”，想要让它表示一个进程组，那么必须首先指定一个信号，或参数前必须有一个“--”选项，否则它将被视为发送的信号。 |

name：使用此名称调用的所有进程将接收信号。

### 例子

```shell
> sleep 20 &

# 列出对应的PID。
> kill -p sleep
23021
```

### 返回值

- 0 成功。
- 1 失败。
- 64 部分成功（当指定了多个进程时）。

### 注意

1. 该命令是`GNU coreutils`包中的命令，相关的帮助信息请查看`man -s 1 kill`或`info coreutils 'kill invocation'`。
2. 启动或关闭内建命令请查看`enable`命令，关于同名优先级的问题请查看`builtin`命令的例子部分的相关讨论。
3. 与`kill`命令类似的有`xkill`，`pkill`,`killall`等，用于不同的目的和场景。

# killall

使用进程的名称来杀死一组进程

## 补充说明

**killall命令** 使用进程的名称来杀死进程，使用此指令可以杀死一组同名进程。我们可以使用kill命令杀死指定进程PID的进程，如果要找到我们需要杀死的进程，我们还需要在之前使用ps等命令再配合grep来查找进程，而killall把这两个过程合二为一，是一个很好用的命令。

### 语法

```shell
killall(选项)(参数)
```

### 选项

```shell
-e：对长名称进行精确匹配；
-l：忽略大小写的不同；
-p：杀死进程所属的进程组；
-i：交互式杀死进程，杀死进程前需要进行确认；
-l：打印所有已知信号列表；
-q：如果没有进程被杀死。则不输出任何信息；
-r：使用正规表达式匹配要杀死的进程名称；
-s：用指定的进程号代替默认信号“SIGTERM”；
-u：杀死指定用户的进程。
```

### 参数

进程名称：指定要杀死的进程名称。

### 实例

```shell
# 杀死所有同名进程
killall vi
# 指定向进程发送的信号
killall -9 vi
# 0信号表示不向进程发送信号, 可通过返回值判断进程是否存在, 0(存在)1(不存在)
killall -0 vi
echo $?
```

# ls

显示目录内容列表

## 补充说明

**ls命令** 就是list的缩写，用来显示目标列表，在Linux中是使用率较高的命令。ls命令的输出信息可以进行彩色加亮显示，以分区不同类型的文件。

### 语法

```shell
ls [选项] [文件名...]
   [-1abcdfgiklmnopqrstuxABCDFGLNQRSUX] [-w cols] [-T cols] [-I pattern] [--full-time] 
   [--format={long,verbose,commas,across,vertical,single-col‐umn}] 
   [--sort={none,time,size,extension}] [--time={atime,access,use,ctime,status}] 
   [--color[={none,auto,always}]] [--help] [--version] [--]
```

### 选项

```shell
-C     # 多列输出，纵向排序。
-F     # 每个目录名加 "/" 后缀，每个 FIFO 名加 "|" 后缀， 每个可运行名加“ * ”后缀。
-R     # 递归列出遇到的子目录。
-a     # 列出所有文件，包括以 "." 开头的隐含文件。
-c     # 使用“状态改变时间”代替“文件修改时间”为依据来排序（使用“-t”选项时）或列出（使用“-l”选项时）。
-d     # 将目录名像其它文件一样列出，而不是列出它们的内容。
-i     # 输出文件前先输出文件系列号（即 i 节点号: i-node number）。 -l  列出（以单列格式）文件模式
       # （file mode），文件的链接数，所有者名，组名，文件大小（以字节为单位），时间信息，及文件名。
       # 缺省时，时间信息显示最近修改时间；可以以选项“-c”和“-u”选择显示其它两种时间信息。对于设备文件，
       # 原先显示文件大小的区域通常显示的是主要和次要的信号（majorand minor device numbers）。
-q     # 将文件名中的非打印字符输出为问号。（对于到终端的输出这是缺省的。）
-r     # 逆序排列。
-t     # 按时间信息排序。
-u     # 使用最近访问时间代替最近修改时间为依据来排序（使用“-t”选项时）或列出（使用“-l”选项时）。
-1     # 单列输出。
-1, --format=single-column  # 一行输出一个文件（单列输出）。如标准输出不是到终端，此选项就是缺省选项。
-a, --all # 列出目录中所有文件，包括以“.”开头的文件。
-b, --escape # 把文件名中不可输出的字符用反斜杠加字符编号(就像在 C 语言里一样)的形式列出。
-c, --time=ctime, --time=status
      # 按文件状态改变时间（i节点中的ctime）排序并输出目录内
      # 容。如采用长格式输出（选项“-l”），使用文件的状态改
      # 变时间取代文件修改时间。【译注：所谓文件状态改变（i节
      # 点中以ctime标志），既包括文件被修改，又包括文件属性（ 如所有者、组、链接数等等）的变化】
-d, --directory
      # 将目录名像其它文件一样列出，而不是列出它们的内容。
-f    # 不排序目录内容；按它们在磁盘上存储的顺序列出。同时启 动“ -a ”选项，如果在“ -f ”之前存在“ -l”、
      # “ - -color ”或“ -s ”，则禁止它们。
-g    # 忽略，为兼容UNIX用。
-i, --inode
      # 在每个文件左边打印  i  节点号（也叫文件序列号和索引号:  file  serial  number and index num‐
      # ber）。i节点号在每个特定的文件系统中是唯一的。
-k, --kilobytes
      # 如列出文件大小，则以千字节KB为单位。
-l, --format=long, --format=verbose
      # 输出的信息从左到右依次包括文件名、文件类型、权限、硬链接数、所有者名、组名、大小（byte）
      # 、及时间信息（如未指明是其它时间即指修改时间）。对于6个月以上的文件或超出未来
      # 1小时的文件，时间信息中的时分将被年代取代。
      # 每个目录列出前，有一行“总块数”显示目录下全部文件所占的磁盘空间。块默认是1024字节；
      # 如果设置了 POSIXLY_CORRECT 的环境变量，除非用“-k”选项，则默认块大小是 512 字节。
      # 每一个硬链接都计入总块数（因此可能重复计数），这无 疑是个缺点。

# 列出的权限类似于以符号表示（文件）模式的规范。但是 ls
      # 在每套权限的第三个字符中结合了多位（ multiple bits ） 的信息，如下： s 如果设置了  setuid
      # 位或 setgid   位，而且也设置了相应的可执行位。 S 如果设置了 setuid 位或 setgid
      # 位，但是没有设置相应的可执行位。 t 如果设置了  sticky  位，而且也设置了相应的可执行位。  T
      # 如果设置了 sticky 位，但是没有设置相应的可执行位。              x
      # 如果仅仅设置了可执行位而非以上四种情况。 - 其它情况（即可执行位未设置）。
-m, --format=commas
      # 水平列出文件，每行尽可能多，相互用逗号和一个空格分隔。
-n, --numeric-uid-gid
      # 列出数字化的 UID 和 GID 而不是用户名和组名。
-o    #  以长格式列出目录内容，但是不显示组信息。等于使用“         --format=long          --no-group
      # ”选项。提供此选项是为了与其它版本的 ls 兼容。
-p    #  在每个文件名后附上一个字符以说明该文件的类型。类似“ -F ”选项但是不 标示可执行文件。
-q, --hide-control-chars
      # 用问号代替文件名中非打印的字符。这是缺省选项。
-r, --reverse
      # 逆序排列目录内容。
-s, --size
      # 在每个文件名左侧输出该文件的大小，以    1024   字节的块为单位。如果设置了   POSIXLY_CORRECT
      # 的环境变量，除非用“ -k ”选项，块大小是 512 字节。
-t, --sort=time
      # 按文件最近修改时间（ i 节点中的 mtime ）而不是按文件名字典序排序，新文件 靠前。
-u, --time=atime, --time=access, --time=use
      # 类似选项“    -t    ”，但是用文件最近访问时间（    i     节点中的     atime     ）取代文件修
      # 改时间。如果使用长格式列出，打印的时间是最近访问时间。
-w, --width cols
       # 假定屏幕宽度是      cols      （      cols     以实际数字取代）列。如未用此选项，缺省值是这
       # 样获得的：如可能先尝试取自终端驱动，否则尝试取自环境变量          COLUMNS          （如果设
       # 置了的话），都不行则取 80 。

-x, --format=across, --format=horizontal
       # 多列输出，横向排序。

-A, --almost-all
       # 显示除 "." 和 ".." 外的所有文件。

-B, --ignore-backups
       # 不输出以“ ~ ”结尾的备份文件，除非已经在命令行中给出。

-C, --format=vertical
       # 多列输出，纵向排序。当标准输出是终端时这是缺省项。使用命令名 dir 和 d 时， 则总是缺省的。

-D, --dired
       # 当采用长格式（“-l”选项）输出时，在主要输出后，额外打印一行：  //DIRED//  BEG1 END1 BEG2
       # END2 ...

# BEGn 和 ENDn 是无符号整数，记录每个文件名的起始、结束位置在输出中的位置（
#        字节偏移量）。这使得          Emacs          易于找到文件名，即使文件名包含空格或换行等非正
#        常字符也无需特异的搜索。
# 
# 如果目录是递归列出的（“ -R ”选项），每个子目录后列出类似一行：
       # //SUBDIRED//  BEG1 END1 ...  【译注：我测试了 TurboLinux4.0 和 RedHat6.1 ，发现它们都是在 “
       # //DIRED//     BEG1...     ”之后列出“     //SUBDIRED//     BEG1     ...      ”，也即只有一个
       # 而不是在每个子目录后都有。而且“ //SUBDIRED// BEG1 ... ”列出的是各个子目 录名的偏移。】

-F, --classify, --file-type
       # 在每个文件名后附上一个字符以说明该文件的类型。“  * ”表示普通的可执行文件； “ / ”表示目录；“
       # @ ”表示符号链接；“ | ”表示FIFOs；“ = ”表示套接字 (sockets) ；什么也没有则表示普通文件。

-G, --no-group
       # 以长格式列目录时不显示组信息。

-I, --ignorepattern
       # 除非在命令行中给定，不要列出匹配shell文件名匹配式（pattern ，不是指一般
       # 表达式）的文件。在shell中，文件名以"."起始的不与在文件名匹配式(pattern)
       # 开头的通配符匹配。

-L, --dereference
       # 列出符号链接指向的文件的信息，而不是符号链接本身。

-N, --literal
       # 不要用引号引起文件名。

-Q, --quote-name
       # 用双引号引起文件名，非打印字符以 C 语言的方法表示。

-R, --recursive
       # 递归列出全部目录的内容。

-S, --sort=size
       # 按文件大小而不是字典序排序目录内容，大文件靠前。

-T, --tabsize cols
       # 假定每个制表符宽度是 cols 。缺省为 8。为求效率， ls 可能在输出中使用制表符。  若 cols 为
       0，则不使用制表符。

-U, --sort=none
       # 不排序目录内容；按它们在磁盘上存储的顺序列出。（选项“-U”和“-f”的不
       # 同是前者不启动或禁止相关的选项。）这在列很大的目录时特别有用，因为不加排序
       # 能显著地加快速度。

-X, --sort=extension
       # 按文件扩展名（由最后的 "." 之后的字符组成）的字典序排序。没有扩展名的先列 出。

--color[=when]
       # 指定是否使用颜色区别文件类别。环境变量  LS_COLORS  指定使用的颜色。如何设置 这个变量见 dir‐
       # colors(1) 。 when 可以被省略，或是以下几项之一：
none # 不使用颜色，这是缺省项。
       # auto 仅当标准输出是终端时使用。 always 总是使用颜色。指定 --color 而且省略 when  时就等同于
       # --color=always 。

--full-time
       # 列出完整的时间，而不是使用标准的缩写。格式如同          date(1)          的缺省格式；此格式
       # 是不能改变的，但是你可以用 cut(1) 取出其中的日期字串并将结果送至命令 “ date -d ”。

# 输出的时间包括秒是非常有用的。（ Unix 文件系统储存文件的时间信息精确到秒，
       # 因此这个选项已经给出了系统所知的全部信息。）例如，当你有一个         Makefile          文件
       # 不能恰当地生成文件时，这个选项会提供帮助。
```

### 参数

目录：指定要显示列表的目录，也可以是具体的文件。

### 实例

```shell
$ ls       # 仅列出当前目录可见文件
$ ls -l    # 列出当前目录可见文件详细信息
$ ls -hl   # 列出详细信息并以可读大小显示文件大小
$ ls -al   # 列出所有文件（包括隐藏）的详细信息
$ ls --human-readable --size -1 -S --classify # 按文件大小排序
$ du -sh * | sort -h # 按文件大小排序(同上)
```

显示当前目录下包括隐藏文件在内的所有文件列表

```shell
[root@localhost ~]# ls -a
.   anaconda-ks.cfg  .bash_logout   .bashrc  install.log         .mysql_history  satools  .tcshrc   .vimrc
..  .bash_history    .bash_profile  .cshrc   install.log.syslog  .rnd            .ssh     .viminfo
```

输出长格式列表

```shell
[root@localhost ~]# ls -1

anaconda-ks.cfg
install.log
install.log.syslog
satools
```

显示文件的inode信息

索引节点（index inode简称为“inode”）是Linux中一个特殊的概念，具有相同的索引节点号的两个文本本质上是同一个文件（除文件名不同外）。

```shell
[root@localhost ~]# ls -i -l anaconda-ks.cfg install.log
2345481 -rw------- 1 root root   859 Jun 11 22:49 anaconda-ks.cfg
2345474 -rw-r--r-- 1 root root 13837 Jun 11 22:49 install.log
```

水平输出文件列表

```shell
[root@localhost /]# ls -m

bin, boot, data, dev, etc, home, lib, lost+found, media, misc, mnt, opt, proc, root, sbin, selinux, srv, sys, tmp, usr, var
```

修改最后一次编辑的文件

最近修改的文件显示在最上面。

```shell
[root@localhost /]# ls -t

tmp  root  etc  dev  lib  boot  sys  proc  data  home  bin  sbin  usr  var  lost+found  media  mnt  opt  selinux  srv  misc
```

显示递归文件

```shell
[root@localhost ~]# ls -R
.:
anaconda-ks.cfg  install.log  install.log.syslog  satools

./satools:
black.txt  freemem.sh  iptables.sh  lnmp.sh  mysql  php502_check.sh  ssh_safe.sh
```

打印文件的UID和GID

```shell
[root@localhost /]# ls -n

total 254
drwxr-xr-x   2 0 0  4096 Jun 12 04:03 bin
drwxr-xr-x   4 0 0  1024 Jun 15 14:45 boot
drwxr-xr-x   6 0 0  4096 Jun 12 10:26 data
drwxr-xr-x  10 0 0  3520 Sep 26 15:38 dev
drwxr-xr-x  75 0 0  4096 Oct 16 04:02 etc
drwxr-xr-x   4 0 0  4096 Jun 12 10:26 home
drwxr-xr-x  14 0 0 12288 Jun 16 04:02 lib
drwx------   2 0 0 16384 Jun 11 22:46 lost+found
drwxr-xr-x   2 0 0  4096 May 11  2011 media
drwxr-xr-x   2 0 0  4096 Nov  8  2010 misc
drwxr-xr-x   2 0 0  4096 May 11  2011 mnt
drwxr-xr-x   2 0 0  4096 May 11  2011 opt
dr-xr-xr-x 232 0 0     0 Jun 15 11:04 proc
drwxr-x---   4 0 0  4096 Oct 15 14:43 root
drwxr-xr-x   2 0 0 12288 Jun 12 04:03 sbin
drwxr-xr-x   2 0 0  4096 May 11  2011 selinux
drwxr-xr-x   2 0 0  4096 May 11  2011 srv
drwxr-xr-x  11 0 0     0 Jun 15 11:04 sys
drwxrwxrwt   3 0 0 98304 Oct 16 08:45 tmp
drwxr-xr-x  13 0 0  4096 Jun 11 23:38 usr
drwxr-xr-x  19 0 0  4096 Jun 11 23:38 var
```

列出文件和文件夹的详细信息

```shell
[root@localhost /]# ls -l

total 254
drwxr-xr-x   2 root root  4096 Jun 12 04:03 bin
drwxr-xr-x   4 root root  1024 Jun 15 14:45 boot
drwxr-xr-x   6 root root  4096 Jun 12 10:26 data
drwxr-xr-x  10 root root  3520 Sep 26 15:38 dev
drwxr-xr-x  75 root root  4096 Oct 16 04:02 etc
drwxr-xr-x   4 root root  4096 Jun 12 10:26 home
drwxr-xr-x  14 root root 12288 Jun 16 04:02 lib
drwx------   2 root root 16384 Jun 11 22:46 lost+found
drwxr-xr-x   2 root root  4096 May 11  2011 media
drwxr-xr-x   2 root root  4096 Nov  8  2010 misc
drwxr-xr-x   2 root root  4096 May 11  2011 mnt
drwxr-xr-x   2 root root  4096 May 11  2011 opt
dr-xr-xr-x 232 root root     0 Jun 15 11:04 proc
drwxr-x---   4 root root  4096 Oct 15 14:43 root
drwxr-xr-x   2 root root 12288 Jun 12 04:03 sbin
drwxr-xr-x   2 root root  4096 May 11  2011 selinux
drwxr-xr-x   2 root root  4096 May 11  2011 srv
drwxr-xr-x  11 root root     0 Jun 15 11:04 sys
drwxrwxrwt   3 root root 98304 Oct 16 08:48 tmp
drwxr-xr-x  13 root root  4096 Jun 11 23:38 usr
drwxr-xr-x  19 root root  4096 Jun 11 23:38 var
```

列出可读文件和文件夹详细信息

```shell
[root@localhost /]# ls -lh

total 254K
drwxr-xr-x   2 root root 4.0K Jun 12 04:03 bin
drwxr-xr-x   4 root root 1.0K Jun 15 14:45 boot
drwxr-xr-x   6 root root 4.0K Jun 12 10:26 data
drwxr-xr-x  10 root root 3.5K Sep 26 15:38 dev
drwxr-xr-x  75 root root 4.0K Oct 16 04:02 etc
drwxr-xr-x   4 root root 4.0K Jun 12 10:26 home
drwxr-xr-x  14 root root  12K Jun 16 04:02 lib
drwx------   2 root root  16K Jun 11 22:46 lost+found
drwxr-xr-x   2 root root 4.0K May 11  2011 media
drwxr-xr-x   2 root root 4.0K Nov  8  2010 misc
drwxr-xr-x   2 root root 4.0K May 11  2011 mnt
drwxr-xr-x   2 root root 4.0K May 11  2011 opt
dr-xr-xr-x 235 root root    0 Jun 15 11:04 proc
drwxr-x---   4 root root 4.0K Oct 15 14:43 root
drwxr-xr-x   2 root root  12K Jun 12 04:03 sbin
drwxr-xr-x   2 root root 4.0K May 11  2011 selinux
drwxr-xr-x   2 root root 4.0K May 11  2011 srv
drwxr-xr-x  11 root root    0 Jun 15 11:04 sys
drwxrwxrwt   3 root root  96K Oct 16 08:49 tmp
drwxr-xr-x  13 root root 4.0K Jun 11 23:38 usr
drwxr-xr-x  19 root root 4.0K Jun 11 23:38 var
```

显示文件夹信息

```shell
[root@localhost /]# ls -ld /etc/

drwxr-xr-x 75 root root 4096 Oct 16 04:02 /etc/
```

按时间列出文件和文件夹详细信息

```shell
[root@localhost /]# ls -lt

total 254
drwxrwxrwt   3 root root 98304 Oct 16 08:53 tmp
drwxr-xr-x  75 root root  4096 Oct 16 04:02 etc
drwxr-x---   4 root root  4096 Oct 15 14:43 root
drwxr-xr-x  10 root root  3520 Sep 26 15:38 dev
drwxr-xr-x  14 root root 12288 Jun 16 04:02 lib
drwxr-xr-x   4 root root  1024 Jun 15 14:45 boot
drwxr-xr-x  11 root root     0 Jun 15 11:04 sys
dr-xr-xr-x 232 root root     0 Jun 15 11:04 proc
drwxr-xr-x   6 root root  4096 Jun 12 10:26 data
drwxr-xr-x   4 root root  4096 Jun 12 10:26 home
drwxr-xr-x   2 root root  4096 Jun 12 04:03 bin
drwxr-xr-x   2 root root 12288 Jun 12 04:03 sbin
drwxr-xr-x  13 root root  4096 Jun 11 23:38 usr
drwxr-xr-x  19 root root  4096 Jun 11 23:38 var
drwx------   2 root root 16384 Jun 11 22:46 lost+found
drwxr-xr-x   2 root root  4096 May 11  2011 media
drwxr-xr-x   2 root root  4096 May 11  2011 mnt
drwxr-xr-x   2 root root  4096 May 11  2011 opt
drwxr-xr-x   2 root root  4096 May 11  2011 selinux
drwxr-xr-x   2 root root  4096 May 11  2011 srv
drwxr-xr-x   2 root root  4096 Nov  8  2010 misc
```

按修改时间列出文件和文件夹详细信息

```shell
[root@localhost /]# ls -ltr

total 254
drwxr-xr-x   2 root root  4096 Nov  8  2010 misc
drwxr-xr-x   2 root root  4096 May 11  2011 srv
drwxr-xr-x   2 root root  4096 May 11  2011 selinux
drwxr-xr-x   2 root root  4096 May 11  2011 opt
drwxr-xr-x   2 root root  4096 May 11  2011 mnt
drwxr-xr-x   2 root root  4096 May 11  2011 media
drwx------   2 root root 16384 Jun 11 22:46 lost+found
drwxr-xr-x  19 root root  4096 Jun 11 23:38 var
drwxr-xr-x  13 root root  4096 Jun 11 23:38 usr
drwxr-xr-x   2 root root 12288 Jun 12 04:03 sbin
drwxr-xr-x   2 root root  4096 Jun 12 04:03 bin
drwxr-xr-x   4 root root  4096 Jun 12 10:26 home
drwxr-xr-x   6 root root  4096 Jun 12 10:26 data
dr-xr-xr-x 232 root root     0 Jun 15 11:04 proc
drwxr-xr-x  11 root root     0 Jun 15 11:04 sys
drwxr-xr-x   4 root root  1024 Jun 15 14:45 boot
drwxr-xr-x  14 root root 12288 Jun 16 04:02 lib
drwxr-xr-x  10 root root  3520 Sep 26 15:38 dev
drwxr-x---   4 root root  4096 Oct 15 14:43 root
drwxr-xr-x  75 root root  4096 Oct 16 04:02 etc
drwxrwxrwt   3 root root 98304 Oct 16 08:54 tmp
```

按照特殊字符对文件进行分类

```shell
[root@localhost nginx-1.2.1]# ls -F

auto/  CHANGES  CHANGES.ru  conf/  configure*  contrib/  html/  LICENSE  Makefile  man/  objs/  README  src/
```

列出文件并标记颜色分类

```shell
[root@localhost nginx-1.2.1]# ls --color=auto

auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
```

## 扩展知识

### 不同颜色代表的文件类型

- `蓝色`：目录
- `绿色`：可执行文件
- `白色`：一般性文件，如文本文件，配置文件等
- `红色`：压缩文件或归档文件
- `浅蓝色`：链接文件
- 红色闪烁：链接文件存在问题
- 黄色：设备文件
- 青黄色：管道文件

# fishshell

比 bash 更好用的 shell

## 安装

```shell
# Ubuntu 和 Debian 的安装方法。
sudo apt-get install fish
# Mac 的安装方法。
brew install fish
```

## 启动与帮助

由于 `Fish` 的语法与 `Bash` 有很大差异，`Bash` 脚本一般不兼容。因此，建议不要将 `Fish` 设为默认 `Shell`，而是每次手动启动它。

```shell
# 安装完成后，就可以启动 Fish。
$ fish
# 使用过程中，如果需要帮助，可以输入 help 命令
$ help
```

## 彩色显示

```shell
# 无效命令为红色
$ mkd
# 有效命令为蓝色
$ mkdir
# 有效路径会有下划线。如果没有下划线，你就知道这个路径不存在。
$ cat ~/somefi 
```

## 自动建议

Fish 会自动在光标后面给出建议，表示可能的选项，颜色为灰色。如果采纳建议，可以按下 `→` 或 `Control + F` 。如果只采纳一部分，可以按下 `Alt + →`。

```shell
$ /bin/hostname # 命令建议
$ grep --ignore-case # 参数建议
$ ls node_modules # 路径建议
```

## 自动补全

输入命令时，`Fish` 会自动显示匹配的上一条历史记录。如果没有匹配的历史记录，`Fish` 会猜测可能的结果，自动补全各种输入。比如，输入 `pyt` 再按下 `Tab` ，就会自动补全为 `python` 命令。

`Fish` 还可以自动补全 `Git` 分支。

## 脚本语法

### if 语句

```shell
if grep fish /etc/shells
    echo Found fish
else if grep bash /etc/shells
    echo Found bash
else
    echo Got nothing
end
```

### switch 语句

```shell
switch (uname)
case Linux
    echo Hi Tux!
case Darwin
    echo Hi Hexley!
case FreeBSD NetBSD DragonFly
    echo Hi Beastie!
case '*'
    echo Hi, stranger!
end
```

### while 循环

```shell
while true
    echo "Loop forever"
end
```

### for 循环

```shell
for file in *.txt
    cp $file $file.bak
end
```

### 函数

`Fish` 的函数用来封装命令，或者为现有的命令起别名。

```shell
function ll
    ls -lhG $argv
end
```

上面代码定义了一个 `ll` 函数。命令行执行这个函数以后，就可以用 `ll` 命令替代 `ls -lhG`。其中，变量 `$argv` 表示函数的参数。

```shell
function ls
    command ls -hG $argv
end
```

上面的代码重新定义 `ls` 命令。注意，函数体内的 `ls` 之前，要加上 `command`，否则会因为无限循环而报错。

### 提示符

`fish_prompt` 函数用于定义命令行提示符（prompt）。

```shell
function fish_prompt
  set_color purple
  date "+%m/%d/%y"
  set_color FF0
  echo (pwd) '>'
  set_color normal
end
```

执行上面的函数以后，你的命令行提示符就会变成下面这样。

```
02/06/13
/home/tutorial > 
```

## 配置

Fish 的配置文件是 `~/.config/fish/config.fish`，每次 `Fish` 启动，就会自动加载这个文件。Fish 还提供 Web 界面配置该文件。

```shell
$ fish_config # 浏览器打开 Web 界面配置
```

Running Commands: 兼容 bash 等shell的命令执行方式
Getting Help: `help/man cmd -> browser/terminal`
Syntax Highlighting: 实时检查命令是否正确
Wildcards: 支持缩写 `*` 递归 匹配
Pipes and Redirections: 使用 `^` 代表 stderr
Autosuggestions: 自动建议, 可以使用 `Ctrl-f / ->` 来补全
Tab Completions: 更强大的 tab 补全
Variables: 使用 set 设置
Exit Status: 使用 `echo $status` 替代 `$?`
Exports (Shell Variables)
Lists: all variables in fish are really lists
Command Substitutions: 使用 `(cmd)` 来执行命令, 而不是 反引号、`$()`
Combiners (And, Or, Not): 不支持使用符合来表示逻辑运算
Functions：使用 `$argv` 替代 `$1`
Conditionals (If, Else, Switch) / Functions / Loops: 更人性化的写法(参考 py)
Prompt: `function fish_prompt` 实现
Startup (Where's .bashrc?): `~/.config/fish/config.fish`，更好的方式是 autoloading-function、universal-variables
Autoloading Functions: `~/.config/fish/functions/.`
Universal Variables：a variable whose value is shared across all instances of fish

```shell
set name 'czl' # 设置变量，替代 name=czl
echo $name
echo $status # exit status，替代 $?
env # 环境变量
set -x MyVariable SomeValue # 替代 export
set -e MyVariable
set PATH $PATH /usr/local/bin # 使用 lists 记录 PATH
set -U fish_user_paths /usr/local/bin $fish_user_paths # 永久生效
touch "testing_"(date +%s)".txt" # command subtitution，替代 `date +%s`
cp file.txt file.txt.bak; and echo 'back success'; or echo 'back fail' # combiner
functions # 列出 fish 下定义的函数
```

# vi

功能强大的纯文本编辑器

## 补充说明

**vi命令** 是UNIX操作系统和类UNIX操作系统中最通用的全屏幕纯文本编辑器。Linux中的vi编辑器叫vim，它是vi的增强版（vi Improved），与vi编辑器完全兼容，而且实现了很多增强功能。

vi编辑器支持编辑模式和命令模式，编辑模式下可以完成文本的编辑功能，命令模式下可以完成对文件的操作命令，要正确使用vi编辑器就必须熟练掌握着两种模式的切换。默认情况下，打开vi编辑器后自动进入命令模式。从编辑模式切换到命令模式使用“esc”键，从命令模式切换到编辑模式使用“A”、“a”、“O”、“o”、“I”、“i”键。

vi编辑器提供了丰富的内置命令，有些内置命令使用键盘组合键即可完成，有些内置命令则需要以冒号“：”开头输入。常用内置命令如下：

```shell
Ctrl+u：向文件首翻半屏；
Ctrl+d：向文件尾翻半屏；
Ctrl+f：向文件尾翻一屏；
Ctrl+b：向文件首翻一屏；
Esc：从编辑模式切换到命令模式；
ZZ：命令模式下保存当前文件所做的修改后退出vi；
:行号：光标跳转到指定行的行首；
:$：光标跳转到最后一行的行首；
x或X：删除一个字符，x删除光标后的，而X删除光标前的；
D：删除从当前光标到光标所在行尾的全部字符；
dd：删除光标行正行内容；
ndd：删除当前行及其后n-1行；
nyy：将当前行及其下n行的内容保存到寄存器？中，其中？为一个字母，n为一个数字；
p：粘贴文本操作，用于将缓存区的内容粘贴到当前光标所在位置的下方；
P：粘贴文本操作，用于将缓存区的内容粘贴到当前光标所在位置的上方；
/字符串：文本查找操作，用于从当前光标所在位置开始向文件尾部查找指定字符串的内容，查找的字符串会被加亮显示；
?字符串：文本查找操作，用于从当前光标所在位置开始向文件头部查找指定字符串的内容，查找的字符串会被加亮显示；
a，bs/F/T：替换文本操作，用于在第a行到第b行之间，将F字符串换成T字符串。其中，“s/”表示进行替换操作；
a：在当前字符后添加文本；
A：在行末添加文本；
i：在当前字符前插入文本；
I：在行首插入文本；
o：在当前行后面插入一空行；
O：在当前行前面插入一空行；
:wq：在命令模式下，执行存盘退出操作；
:w：在命令模式下，执行存盘操作；
:w!：在命令模式下，执行强制存盘操作；
:q：在命令模式下，执行退出vi操作；
:q!：在命令模式下，执行强制退出vi操作；
:e文件名：在命令模式下，打开并编辑指定名称的文件；
:n：在命令模式下，如果同时打开多个文件，则继续编辑下一个文件；
:f：在命令模式下，用于显示当前的文件名、光标所在行的行号以及显示比例；
:set number：在命令模式下，用于在最左端显示行号；
:set nonumber：在命令模式下，用于在最左端不显示行号；
```

### 语法

```shell
vi(选项)(参数)
```

### 选项

```shell
+<行号>：从指定行号的行开始显示文本内容；
-b：以二进制模式打开文件，用于编辑二进制文件和可执行文件；
-c<指令>：在完成对第一个文件编辑任务后，执行给出的指令；
-d：以diff模式打开文件，当多个文件编辑时，显示文件差异部分；
-l：使用lisp模式，打开“lisp”和“showmatch”；
-m：取消写文件功能，重设“write”选项；
-M：关闭修改功能；
-n：不实用缓存功能；
-o<文件数目>：指定同时打开指定数目的文件；
-R：以只读方式打开文件；
-s：安静模式，不显示指令的任何错误信息。
```

### 参数

文件列表：指定要编辑的文件列表。多个文件之间使用空格分隔开。

## 知识扩展

vi编辑器有三种工作方式：命令方式、输入方式和ex转义方式。通过相应的命令或操作，在这三种工作方式之间可以进行转换。

**命令方式**

在Shell提示符后输入命令vi，进入vi编辑器，并处于vi的命令方式。此时，从键盘上输入的任何字符都被作为编辑命令来解释，例如，a(append）表示附加命令，i(insert）表示插入命令，x表示删除字符命令等。如果输入的字符不是vi的合法命令，则机器发出“报警声”，光标不移动。另外，在命令方式下输入的字符（即vi命令）并不在屏幕上显示出来，例如，输入i，屏幕上并无变化，但通过执行i命令，编辑器的工作方式却发生变化：由命令方式变为输入方式。

**输入方式**

通过输入vi的插入命令（i）、附加命令（a）、打开命令（o）、替换命令（s）、修改命令(c）或取代命令（r）可以从命令方式进入输入方式。在输入方式下，从键盘上输入的所有字符都被插入到正在编辑的缓冲区中，被当做该文件的正文。进入输入方式后，输入的可见字符都在屏幕上显示出来，而编辑命令不再起作用，仅作为普通字母出现。例如，在命令方式下输入字母i，进到输入方式，然后再输入i，就在屏幕上相应光标处添加一个字母i。

由输入方式回到命令方式的办法是按下Esc键。如果已在命令方式下，那么按下Esc键就会发出“嘟嘟”声。为了确保用户想执行的vi命令是在命令方式下输入的，不妨多按几下Esc键，听到嘟声后再输入命令。

**ex转义方式**

vi和ex编辑器的功能是相同的，二者的主要区别是用户界面。在vi中，命令通常是单个字母，如a,x,r等。而在ex中，命令是以Enter；键结束的命令行。vi有一个专门的“转义”命令，可访问很多面向行的ex命令。为使用ex转义方式，可输入一个冒号（:）。作为ex命令提示符，冒号出现在状态行（通常在屏幕最下一行）。按下中断键（通常是Del键），可终止正在执行的命令。多数文件管理命令都是在ex转义方式下执行的（例如，读取文件，把编辑缓冲区的内容写到文件中等）。转义命令执行后，自动回到命令方式。例如：

```shell
:1,$s/I/i/g 按Enter键
```

# rm

用于删除给定的文件和目录

## 补充说明

**rm** **命令** 可以删除一个目录中的一个或多个文件或目录，也可以将某个目录及其下属的所有文件及其子目录均删除掉。对于链接文件，只是删除整个链接文件，而原有文件保持不变。

注意：使用rm命令要格外小心。因为一旦删除了一个文件，就无法再恢复它。所以，在删除文件之前，最好再看一下文件的内容，确定是否真要删除。rm命令可以用-i选项，这个选项在使用文件扩展名字符删除多个文件时特别有用。使用这个选项，系统会要求你逐一确定是否要删除。这时，必须输入y并按Enter键，才能删除文件。如果仅按Enter键或其他字符，文件不会被删除。

### 语法

```shell
rm (选项)(参数)
```

### 选项

```shell
-d：直接把欲删除的目录的硬连接数据删除成0，删除该目录；
-f：强制删除文件或目录；
-i：删除已有文件或目录之前先询问用户；
-r或-R：递归处理，将指定目录下的所有文件与子目录一并处理；
--preserve-root：不对根目录进行递归操作；
-v：显示指令的详细执行过程。
```

### 参数

文件：指定被删除的文件列表，如果参数中含有目录，则必须加上`-r`或者`-R`选项。

### 实例

交互式删除当前目录下的文件test和example

```shell
rm -i test example
Remove test ?n（不删除文件test)
Remove example ?y（删除文件example)
```

删除当前目录下除隐含文件外的所有文件和子目录

```shell
# rm -r *
```

应注意，这样做是非常危险的!

**删除当前目录下的 package-lock.json 文件**

```shell
find .  -name "package-lock.json" -exec rm -rf {} \;
```

**查找 \*.html 结尾的文件并删除**

```shell
find ./docs -name "*.html" -exec rm -rf {} \;
```

**删除当前项目下 \*.html 结尾的文件**

```shell
rm -rf *.html
```

**删除当前目录下的 node_modules 目录**

```shell
find . -name 'node_modules' -type d -prune -exec rm -rf '{}' +
```

**删除文件**

```shell
# rm 文件1 文件2 ...
rm testfile.txt
```

**删除目录**

> rm -r [目录名称] -r 表示递归地删除目录下的所有文件和目录。 -f 表示强制删除

```shell
rm -rf testdir
rm -r testdir
```

**删除操作前有确认提示**

> rm -i [文件/目录]

```shell
rm -r -i testdir
```

**批量删除 `icons` 文件夹中的子文件夹中的 data 文件夹**

```shell
rm -rf icons/**/data
```

**rm 忽略不存在的文件或目录**

> -f 选项（LCTT 译注：即 “force”）让此次操作强制执行，忽略错误提示

```shell
rm -f [文件...]
```

**仅在某些场景下确认删除**

> 选项 -I，可保证在删除超过 3 个文件时或递归删除时（LCTT 译注： 如删除目录）仅提示一次确认。

```shell
rm -I file1 file2 file3
```

**删除根目录**

> 当然，删除根目录（/）是 Linux 用户最不想要的操作，这也就是为什么默认 rm 命令不支持在根目录上执行递归删除操作。 然而，如果你非得完成这个操作，你需要使用 --no-preserve-root 选项。当提供此选项，rm 就不会特殊处理根目录（/）了。

```shell
不给示例了，操作系统都被你删除了，你太坏了😆
```

**rm 显示当前删除操作的详情**

```shellrmdir
rm -v [文件/目录]
```

# rmdir

用来删除空目录

## 补充说明

**rmdir命令** 用来删除空目录。当目录不再被使用时，或者磁盘空间已到达使用限定值，就需要删除失去使用价值的目录。利用rmdir命令可以从一个目录中删除一个或多个空的子目录。该命令从一个目录中删除一个或多个子目录，其中dirname佬表示目录名。如果dirname中没有指定路径，则删除当前目录下由dirname指定的目录；如dirname中包含路径，则删除指定位置的目录。删除目录时，必须具有对其父目录的写权限。

注意：子目录被删除之前应该是空目录。就是说，该目录中的所有文件必须用rm命令全部，另外，当前工作目录必须在被删除目录之上，不能是被删除目录本身，也不能是被删除目录的子目录。

虽然还可以用带有`-r`选项的rm命令递归删除一个目录中的所有文件和该目录本身，但是这样做存在很大的危险性。

### 语法

```shell
rmdir(选项)(参数)
```

### 选项

```shell
-p或--parents：删除指定目录后，若该目录的上层目录已变成空目录，则将其一并删除；
--ignore-fail-on-non-empty：此选项使rmdir命令忽略由于删除非空目录时导致的错误信息；
-v或-verboes：显示命令的详细执行过程；
--help：显示命令的帮助信息；
--version：显示命令的版本信息。
```

### 参数

目录列表：要删除的空目录列表。当删除多个空目录时，目录名之间使用空格隔开。

### 实例

将工作目录下，名为 `www` 的子目录删除 :

```shell
rmdir www
```

在工作目录下的 www 目录中，删除名为 Test 的子目录。若 Test 删除后，www 目录成为空目录，则 www 亦予删除。

```shell
rmdir -p www/Test
```

下面命令等价于 `rmdir a/b/c`, `rmdir a/b`, `rmdir a`

```shell
rmdir -p a/b/c
```

# rmmod

从运行的内核中移除指定的内核模块

## 补充说明

**rmmod命令** 用于从当前运行的内核中移除指定的内核模块。执行rmmod指令，可删除不需要的模块。Linux操作系统的核心具有模块化的特性，应此在编译核心时，务须把全部的功能都放如核心。你可以将这些功能编译成一个个单独的模块，待有需要时再分别载入它们。

### 语法

```shell
rmmod(选项)(参数)
```

### 选项

```shell
-v：显示指令执行的详细信息；
-f：强制移除模块，使用此选项比较危险；
-w：等待着，直到模块能够被除时在移除模块；
-s：向系统日志（syslog）发送错误信息。
```

### 参数

模块名：要移除的模块名称。

### 实例

用rmmod命令主要用于卸载正在使用的Linux内核模块，与`modprobe -r`命令相似，如下所示：

```shell
[root@localhost boot]# lsmod | grep raid1
raid1                  25153  0

[root@localhost boot]# rmmod raid1
[root@localhost boot]# lsmod | grep raid1
```

# nohup

将程序以忽略挂起信号的方式运行起来

## 补充说明

**nohup命令** 可以将程序以忽略挂起信号的方式运行起来，被运行的程序的输出信息将不会显示到终端。

无论是否将 nohup 命令的输出重定向到终端，输出都将附加到当前目录的 nohup.out 文件中。如果当前目录的 nohup.out 文件不可写，输出重定向到`$HOME/nohup.out`文件中。如果没有文件能创建或打开以用于追加，那么 command 参数指定的命令不可调用。如果标准错误是一个终端，那么把指定的命令写给标准错误的所有输出作为标准输出重定向到相同的文件描述符。

### 语法

```shell
nohup(选项)(参数)
```

### 选项

```shell
--help：在线帮助；
--version：显示版本信息。
```

### 参数

程序及选项：要运行的程序及选项。

### 实例

使用nohup命令提交作业，如果使用nohup命令提交作业，那么在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中，除非另外指定了输出文件：

```shell
nohup command > myout.file 2>&1 &
```

在上面的例子中，输出被重定向到myout.file文件中。

该指令表示不做挂断操作，后台下载

```shell
nohup wget site.com/file.zip
```

下面命令，会在同一个目录下生成一个名称为 `nohup.out` 的文件，其中包含了正在运行的程序的输出内容

```shell
nohup ping -c 10 baidu.com
```

最简单的后台运行

```shell
nohup command &
```

输出默认重定向到当前目录下 nohup.out 文件

```shell
nohup python main.py &
```

自定义输出文件(标准输出和错误输出合并到 main.log)

```shell
nohup python main.py >> main.log 2>&1 &
```

与上一个例子相同作用的简写方法

```shell
nohup python main.py &> main.log &
```

不记录输出信息

```shell
nohup python main.py &> /dev/null &
```

不记录输出信息并将程序的进程号写入 pidfile.txt 文件中，方便后续杀死进程

```shell
nohup python main.py &> /dev/null & echo $! > pidfile.txt
```

# scp

加密的方式在本地主机和远程主机之间复制文件

## 补充说明

**scp命令** 用于在Linux下进行远程拷贝文件的命令，和它类似的命令有cp，不过cp只是在本机进行拷贝不能跨服务器，而且scp传输是加密的。可能会稍微影响一下速度。当你服务器硬盘变为只读read only system时，用scp可以帮你把文件移出来。另外，scp还非常不占资源，不会提高多少系统负荷，在这一点上，rsync就远远不及它了。虽然 rsync比scp会快一点，但当小文件众多的情况下，rsync会导致硬盘I/O非常高，而scp基本不影响系统正常使用。

### 语法

```shell
scp(选项)(参数)
```

### 选项

```shell
-1：使用ssh协议版本1；
-2：使用ssh协议版本2；
-4：使用ipv4；
-6：使用ipv6；
-B：以批处理模式运行；
-C：使用压缩；
-F：指定ssh配置文件；
-i：identity_file 从指定文件中读取传输时使用的密钥文件（例如亚马逊云pem），此参数直接传递给ssh；
-l：指定宽带限制；
-o：指定使用的ssh选项；
-P：指定远程主机的端口号；
-p：保留文件的最后修改时间，最后访问时间和权限模式；
-q：不显示复制进度；
-r：以递归方式复制。
```

### 参数

- 源文件：指定要复制的源文件。
- 目标文件：目标文件。格式为`user@host：filename`（文件名为目标文件的名称）。

### 实例

从远程复制到本地的scp命令与上面的命令雷同，只要将从本地复制到远程的命令后面2个参数互换顺序就行了。

**从远程机器复制文件到本地目录**

```shell
scp root@10.10.10.10:/opt/soft/nginx-0.5.38.tar.gz /opt/soft/
```

从10.10.10.10机器上的`/opt/soft/`的目录中下载nginx-0.5.38.tar.gz 文件到本地`/opt/soft/`目录中。

**从亚马逊云复制OpenVPN到本地目录**

```shell
scp -i amazon.pem ubuntu@10.10.10.10:/usr/local/openvpn_as/etc/exe/openvpn-connect-2.1.3.110.dmg openvpn-connect-2.1.3.110.dmg
```

从10.10.10.10机器上下载openvpn安装文件到本地当前目录来。

**从远程机器复制到本地**

```shell
scp -r root@10.10.10.10:/opt/soft/mongodb /opt/soft/
```

从10.10.10.10机器上的`/opt/soft/`中下载mongodb目录到本地的`/opt/soft/`目录来。

**上传本地文件到远程机器指定目录**

```shell
scp /opt/soft/nginx-0.5.38.tar.gz root@10.10.10.10:/opt/soft/scptest
# 指定端口 2222
scp -rp -P 2222 /opt/soft/nginx-0.5.38.tar.gz root@10.10.10.10:/opt/soft/scptest
```

复制本地`/opt/soft/`目录下的文件nginx-0.5.38.tar.gz到远程机器10.10.10.10的`opt/soft/scptest`目录。

**上传本地目录到远程机器指定目录**

```shell
scp -r /opt/soft/mongodb root@10.10.10.10:/opt/soft/scptest
```

上传本地目录`/opt/soft/mongodb`到远程机器10.10.10.10上`/opt/soft/scptest`的目录中去。

# telnet

登录远程主机和管理(测试ip端口是否连通)

## 补充说明

**telnet命令** 用于登录远程主机，对远程主机进行管理。telnet因为采用明文传送报文，安全性不好，很多Linux服务器都不开放telnet服务，而改用更安全的ssh方式了。但仍然有很多别的系统可能采用了telnet方式来提供远程登录，因此弄清楚telnet客户端的使用方式仍是很有必要的。

### 语法

```shell
telnet(选项)(参数)
```

### 选项

```shell
-8：允许使用8位字符资料，包括输入与输出；
-a：尝试自动登入远端系统；
-b<主机别名>：使用别名指定远端主机名称；
-c：不读取用户专属目录里的.telnetrc文件；
-d：启动排错模式；
-e<脱离字符>：设置脱离字符；
-E：滤除脱离字符；
-f：此参数的效果和指定"-F"参数相同；
-F：使用Kerberos V5认证时，加上此参数可把本地主机的认证数据上传到远端主机；
-k<域名>：使用Kerberos认证时，加上此参数让远端主机采用指定的领域名，而非该主机的域名；
-K：不自动登入远端主机；
-l<用户名称>：指定要登入远端主机的用户名称；
-L：允许输出8位字符资料；
-n<记录文件>：指定文件记录相关信息；
-r：使用类似rlogin指令的用户界面；
-S<服务类型>：设置telnet连线所需的ip TOS信息；
-x：假设主机有支持数据加密的功能，就使用它；
-X<认证形态>：关闭指定的认证形态。
```

### 参数

- 远程主机：指定要登录进行管理的远程主机；
- 端口：指定TELNET协议使用的端口号。

### 实例

```shell
$ telnet 192.168.2.10
Trying 192.168.2.10...
Connected to 192.168.2.10 (192.168.2.10).
Escape character is '^]'.

    localhost (Linux release 2.6.18-274.18.1.el5 #1 SMP Thu Feb 9 12:45:44 EST 2012) (1)

login: root
Password:
Login incorrect
```

一般情况下不允许root从远程登录，可以先用普通账号登录，然后再用su -切到root用户。

```shell
$ telnet 192.168.188.132
Trying 192.168.188.132...
telnet: connect to address 192.168.188.132: Connection refused
telnet: Unable to connect to remote host
```

处理这种情况方法：

1. 确认ip地址是否正确？
2. 确认ip地址对应的主机是否已经开机？
3. 如果主机已经启动，确认路由设置是否设置正确？（使用route命令查看）
4. 如果主机已经启动，确认主机上是否开启了telnet服务？（使用netstat命令查看，TCP的23端口是否有LISTEN状态的行）
5. 如果主机已经启动telnet服务，确认防火墙是否放开了23端口的访问？（使用iptables-save查看）

**启动telnet服务**

```shell
service xinetd restart
```

配置参数，通常的配置如下：

```shell
service telnet
{
    disable = no #启用
    flags = REUSE #socket可重用
    socket_type = stream #连接方式为TCP
    wait = no #为每个请求启动一个进程
    user = root #启动服务的用户为root
    server = /usr/sbin/in.telnetd #要激活的进程
    log_on_failure += USERID #登录失败时记录登录用户名
}
```

如果要配置允许登录的客户端列表，加入

```
only_from = 192.168.0.2 #只允许192.168.0.2登录
```

如果要配置禁止登录的客户端列表，加入

```
no_access = 192.168.0.{2,3,4} #禁止192.168.0.2、192.168.0.3、192.168.0.4登录
```

如果要设置开放时段，加入

```
access_times = 9:00-12:00 13:00-17:00 # 每天只有这两个时段开放服务（我们的上班时间：P）
```

如果你有两个IP地址，一个是私网的IP地址如192.168.0.2，一个是公网的IP地址如218.75.74.83，如果你希望用户只能从私网来登录telnet服务，那么加入

```
bind = 192.168.0.2
```

各配置项具体的含义和语法可参考xined配置文件属性说明（man xinetd.conf）

配置端口，修改services文件：

```shell
# vi /etc/services
```

找到以下两句

```shell
telnet 23/tcp
telnet 23/udp
```

如果前面有#字符，就去掉它。telnet的默认端口是23，这个端口也是黑客端口扫描的主要对象，因此最好将这个端口修改掉，修改的方法很简单，就是将23这个数字修改掉，改成大一点的数字，比如61123。注意，1024以下的端口号是internet保留的端口号，因此最好不要用，还应该注意不要与其它服务的端口冲突。

启动服务：

```
service xinetd restart
```

# chmod

用来变更文件或目录的权限

## 概要

```shell
chmod [OPTION]... MODE[,MODE]... FILE...
chmod [OPTION]... OCTAL-MODE FILE...
chmod [OPTION]... --reference=RFILE FILE...
```

## 主要用途

- 通过符号组合的方式更改目标文件或目录的权限。
- 通过八进制数的方式更改目标文件或目录的权限。
- 通过参考文件的权限来更改目标文件或目录的权限。

## 参数

mode：八进制数或符号组合。

file：指定要更改权限的一到多个文件。

## 选项

```shell
-c, --changes：当文件的权限更改时输出操作信息。
--no-preserve-root：不将'/'特殊化处理，默认选项。
--preserve-root：不能在根目录下递归操作。
-f, --silent, --quiet：抑制多数错误消息的输出。
-v, --verbose：无论文件是否更改了权限，一律输出操作信息。
--reference=RFILE：使用参考文件或参考目录RFILE的权限来设置目标文件或目录的权限。
-R, --recursive：对目录以及目录下的文件递归执行更改权限操作。
--help：显示帮助信息并退出。
--version：显示版本信息并退出。
```

## 返回值

返回状态为成功除非给出了非法选项或非法参数。

## 例子

> 参考`man chmod`文档的`DESCRIPTION`段落得知：
>
> - `u`符号代表当前用户。
> - `g`符号代表和当前用户在同一个组的用户，以下简称组用户。
> - `o`符号代表其他用户。
> - `a`符号代表所有用户。
> - `r`符号代表读权限以及八进制数`4`。
> - `w`符号代表写权限以及八进制数`2`。
> - `x`符号代表执行权限以及八进制数`1`。
> - `X`符号代表如果目标文件是可执行文件或目录，可给其设置可执行权限。
> - `s`符号代表设置权限suid和sgid，使用权限组合`u+s`设定文件的用户的ID位，`g+s`设置组用户ID位。
> - `t`符号代表只有目录或文件的所有者才可以删除目录下的文件。
> - `+`符号代表添加目标用户相应的权限。
> - `-`符号代表删除目标用户相应的权限。
> - `=`符号代表添加目标用户相应的权限，删除未提到的权限。

```shell
linux文件的用户权限说明：

# 查看当前目录（包含隐藏文件）的长格式。
ls -la
  -rw-r--r--   1 user  staff   651 Oct 12 12:53 .gitmodules

# 第1位如果是d则代表目录，是-则代表普通文件。
# 更多详情请参阅info coreutils 'ls invocation'（ls命令的info文档）的'-l'选项部分。
# 第2到4位代表当前用户的权限。
# 第5到7位代表组用户的权限。
# 第8到10位代表其他用户的权限。
# 添加组用户的写权限。
chmod g+w ./test.log
# 删除其他用户的所有权限。
chmod o= ./test.log
# 使得所有用户都没有写权限。
chmod a-w ./test.log
# 当前用户具有所有权限，组用户有读写权限，其他用户只有读权限。
chmod u=rwx, g=rw, o=r ./test.log
# 等价的八进制数表示：
chmod 764 ./test.log
# 将目录以及目录下的文件都设置为所有用户拥有读写权限。
# 注意，使用'-R'选项一定要保留当前用户的执行和读取权限，否则会报错！
chmod -R a=rw ./testdir/
# 根据其他文件的权限设置文件权限。
chmod --reference=./1.log  ./test.log
```

### 注意

1. 该命令是`GNU coreutils`包中的命令，相关的帮助信息请查看`man chmod`或`info coreutils 'chmod invocation'`。
2. 符号连接的权限无法变更，如果用户对符号连接修改权限，其改变会作用在被连接的原始文件。
3. 使用`-R`选项一定要保留当前用户的执行和读取权限，否则会报错！

# chown

用来变更文件或目录的拥有者或所属群组

## 补充说明

**chown命令** 改变某个文件或目录的所有者和所属的组，该命令可以向某个用户授权，使该用户变成指定文件的所有者或者改变文件所属的组。用户可以是用户或者是用户D，用户组可以是组名或组id。文件名可以使由空格分开的文件列表，在文件名中可以包含通配符。

只有文件主和超级用户才可以使用该命令。

### 语法

```shell
chown(选项)(参数)
```

### 选项

```shell
-c或——changes：效果类似“-v”参数，但仅回报更改的部分；
-f或--quite或——silent：不显示错误信息；
-h或--no-dereference：只对符号连接的文件作修改，而不更改其他任何相关文件；
-R或——recursive：递归处理，将指定目录下的所有文件及子目录一并处理；
-v或——verbose：显示指令执行过程；
--dereference：效果和“-h”参数相同；
--help：在线帮助；
--reference=<参考文件或目录>：把指定文件或目录的拥有者与所属群组全部设成和参考文件或目录的拥有者与所属群组相同；
--version：显示版本信息。
```

### 参数

用户：组：指定所有者和所属工作组。当省略“：组”，仅改变文件所有者；
文件：指定要改变所有者和工作组的文件列表。支持多个文件和目标，支持shell通配符。

### 实例

将目录`/usr/meng`及其下面的所有文件、子目录的文件主改成 liu：

```shell
chown -R liu /usr/meng
```

# useradd

创建的新的系统用户

## 补充说明

**useradd命令** 用于Linux中创建的新的系统用户。useradd可用来建立用户帐号。帐号建好之后，再用passwd设定帐号的密码．而可用userdel删除帐号。使用useradd指令所建立的帐号，实际上是保存在`/etc/passwd`文本文件中。

在Slackware中，adduser指令是个script程序，利用交谈的方式取得输入的用户帐号资料，然后再交由真正建立帐号的useradd命令建立新用户，如此可方便管理员建立用户帐号。在Red Hat Linux中， **adduser命令** 则是useradd命令的符号连接，两者实际上是同一个指令。

### 语法

```shell
useradd(选项)(参数)
```

### 选项

```shell
-b, --base-dir BASE_DIR  # 如果未指定 -d HOME_DIR，则系统的默认基本目录。如果未指定此选项，useradd 将使用 /etc/default/useradd 中的 HOME 变量指定的基本目录，或默认使用 /home。
-c, --comment COMMENT    # 加上备注文字。任何文本字符串。它通常是对登录名的简短描述，目前用作用户全名的字段。
-d, --home HOME_DIR      # 将使用 HOME_DIR 作为用户登录目录的值来创建新用户。 
-D, --defaults           # 变更预设值。
-e, --expiredate EXPIRE_DATE # 用户帐户将被禁用的日期。 日期以 YYYY-MM-DD 格式指定。
-f, --inactive INACTIVE      # 密码过期后到帐户被永久禁用的天数。
-g, --gid GROUP   # 用户初始登录组的组名或编号。组名必须存在。组号必须引用已经存在的组。
-G, --groups GROUP1[,GROUP2,...[,GROUPN]]] # 用户也是其成员的补充组列表。每个组用逗号隔开，中间没有空格。
-h, --help # 显示帮助信息并退出。
-k, --skel SKEL_DIR # 骨架目录，其中包含要在用户的主目录中复制的文件和目录，当主目录由 useradd 创建时。
-K, --key KEY=VALUE # 覆盖 /etc/login.defs 默认值（UID_MIN、UID_MAX、UMASK、PASS_MAX_DAYS 等）。
-l, --no-log-init   # 不要将用户添加到 lastlog 和 faillog 数据库。
-m, --create-home   # 如果用户的主目录不存在，则创建它。
-M                  # 不要创建用户的主目录，即使 /etc/login.defs (CREATE_HOME) 中的系统范围设置设置为 yes。
-N, --no-user-group # 不要创建与用户同名的组，而是将用户添加到由 -g 选项或 /etc/default/useradd 中的 GROUP 变量指定的组中。
-o, --non-unique    # 允许创建具有重复（非唯一）UID 的用户帐户。 此选项仅在与 -o 选项结合使用时有效。
-p, --password PASSWORD # crypt(3) 返回的加密密码。 默认是禁用密码。
-r, --system        # 创建一个系统帐户。
-s, --shell SHELL   # 用户登录 shell 的名称。
-u, --uid UID       # 用户 ID 的数值。
-U, --user-group    # 创建一个与用户同名的组，并将用户添加到该组。
-Z, --selinux-user SEUSER # 用户登录的 SELinux 用户。 默认情况下将此字段留空，这会导致系统选择默认的 SELinux 用户。

# 更改默认值
# 当仅使用 -D 选项调用时，useradd 将显示当前默认值。 当使用 -D 和其他选项调用时，useradd 将更新指定选项的默认值。 有效的默认更改选项是：
```

### 参数

用户名：要创建的用户名。

### 退出值

`useradd` 命令以以下值退出：

```shell
0 成功
1 无法更新密码文件
2 无效的命令语法
3 选项的无效参数
4 UID 已经在使用（并且没有 -o）
6 指定的组不存在
9 用户名已被使用
10 无法更新组文件
12 无法创建主目录
13 无法创建邮件假脱机
14 无法更新 SELinux 用户映射
```

### 文件

```shell
/etc/passwd # 用户帐户信息。
/etc/shadow # 保护用户帐户信息。
/etc/group  # 组帐户信息。
/etc/gshadow # 保护组帐户信息。
/etc/default/useradd # 帐户创建的默认值。
/etc/skel/                                 # 包含默认文件的目录。
/etc/login.defs # 影子密码套件配置。
```

### 实例

新建用户加入组：

```shell
useradd –g sales jack –G company,employees    # -g：加入主要组、-G：加入次要组
```

建立一个新用户账户，并设置ID：

```shell
useradd caojh -u 544
```

需要说明的是，设定ID值时尽量要大于500，以免冲突。因为Linux安装后会建立一些特殊用户，一般0到499之间的值留给bin、mail这样的系统账号。

新建一个普通用户：

```shell
useradd lutixia
```

新建一个系统用户,系统用户一般用于管理服务，无需登录，所以分配nologin，限制其登录系统：

```shell
useradd -r -s /sbin/nologin mq
```

修改创建用户的默认参数，设置密码过期后到永久禁用的不活动时间为30天:

```shell
useradd -D -f 30
```

# yum

基于RPM的软件包管理器

## 补充说明

**yum命令** 是在Fedora和RedHat以及SUSE中基于rpm的软件包管理器，它可以使系统管理人员交互和自动化地更新与管理RPM软件包，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。

yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

### 语法

```shell
yum(选项)(参数)
```

### 选项

```shell
-h：显示帮助信息；
-y：对所有的提问都回答“yes”；
-c：指定配置文件；
-q：安静模式；
-v：详细模式；
-d：设置调试等级（0-10）；
-e：设置错误等级（0-10）；
-R：设置yum处理一个命令的最大等待时间；
-C：完全从缓存中运行，而不去下载或者更新任何头文件。
```

### 参数

```shell
install：安装rpm软件包；
update：更新rpm软件包；
check-update：检查是否有可用的更新rpm软件包；
remove：删除指定的rpm软件包；
list：显示软件包的信息；
search：检查软件包的信息；
info：显示指定的rpm软件包的描述信息和概要信息；
clean：清理yum过期的缓存；
shell：进入yum的shell提示符；
resolvedep：显示rpm软件包的依赖关系；
localinstall：安装本地的rpm软件包；
localupdate：显示本地rpm软件包进行更新；
deplist：显示rpm软件包的所有依赖关系；
provides：查询某个程序所在安装包。
```

### 实例

部分常用的命令包括：

- 自动搜索最快镜像插件：`yum install yum-fastestmirror`
- 安装yum图形窗口插件：`yum install yumex`
- 查看可能批量安装的列表：`yum grouplist`

**安装**

```shell
yum install              #全部安装
yum install package1     #安装指定的安装包package1
yum groupinsall group1   #安装程序组group1
```

**更新和升级**

```shell
yum update               #全部更新
yum update package1      #更新指定程序包package1
yum check-update         #检查可更新的程序
yum upgrade package1     #升级指定程序包package1
yum groupupdate group1   #升级程序组group1
```

**查找和显示**

```shell
# 检查 MySQL 是否已安装
yum list installed | grep mysql
yum list installed mysql*

yum info package1      #显示安装包信息package1
yum list               #显示所有已经安装和可以安装的程序包
yum list package1      #显示指定程序包安装情况package1
yum groupinfo group1   #显示程序组group1信息yum search string 根据关键字string查找安装包
```

**删除程序**

```shell
yum remove &#124; erase package1   #删除程序包package1
yum groupremove group1             #删除程序组group1
yum deplist package1               #查看程序package1依赖情况
```

**清除缓存**

```shell
yum clean packages       # 清除缓存目录下的软件包
yum clean headers        # 清除缓存目录下的 headers
yum clean oldheaders     # 清除缓存目录下旧的 headers
```

**更多实例**

```shelldu
# yum
/etc/yum.repos.d/       # yum 源配置文件
vi /etc/yum.repos.d/nginx.repo # 举个栗子: nginx yum源
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch/
gpgcheck=0
enabled=1

# yum mirror
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
wget https://mirror.tuna.tsinghua.edu.cn/help/centos/
yum makecache

# 添加中文语言支持
LANG=C # 原始语言
LANG=zh_CN.utf8 # 切换到中文
yum groupinstall "Chinese Support" # 添加中文语言支持
```

# du

显示每个文件和目录的磁盘使用空间

## 补充说明

**du命令** 也是查看使用空间的，但是与df命令不同的是Linux du命令是对文件和目录磁盘使用的空间的查看，还是和df命令有一些区别的。

### 语法

```shell
du [选项][文件]
```

### 选项

```shell
-a, --all                              显示目录中个别文件的大小。
-B, --block-size=大小                  使用指定字节数的块
-b, --bytes                            显示目录或文件大小时，以byte为单位。
-c, --total                            除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
-D, --dereference-args                 显示指定符号链接的源文件大小。
-d, --max-depth=N                      限制文件夹深度
-H, --si                               与-h参数相同，但是K，M，G是以1000为换算单位。
-h, --human-readable                   以K，M，G为单位，提高信息的可读性。
-k, --kilobytes                        以KB(1024bytes)为单位输出。
-l, --count-links                      重复计算硬件链接的文件。
-m, --megabytes                        以MB为单位输出。
-L<符号链接>, --dereference<符号链接>  显示选项中所指定符号链接的源文件大小。
-P, --no-dereference                   不跟随任何符号链接(默认)
-0, --null                             将每个空行视作0 字节而非换行符
-S, --separate-dirs                    显示个别目录的大小时，并不含其子目录的大小。
-s, --summarize                        仅显示总计，只列出最后加总的值。
-x, --one-file-xystem                  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
-X<文件>, --exclude-from=<文件>        在<文件>指定目录或文件。
--apparent-size                        显示表面用量，而并非是磁盘用量；虽然表面用量通常会小一些，但有时它会因为稀疏文件间的"洞"、内部碎片、非直接引用的块等原因而变大。
--files0-from=F                        计算文件F中以NUL结尾的文件名对应占用的磁盘空间如果F的值是"-"，则从标准输入读入文件名
--exclude=<目录或文件>                 略过指定的目录或文件。
--max-depth=N                          显示目录总计(与--all 一起使用计算文件)当N为指定数值时计算深度为N，等于0时等同--summarize
--si                                   类似-h，但在计算时使用1000 为基底而非1024
--time                                 显示目录或该目录子目录下所有文件的最后修改时间
--time=WORD                            显示WORD时间，而非修改时间：atime，access，use，ctime 或status
--time-style=样式                      按照指定样式显示时间(样式解释规则同"date"命令)：full-iso，long-iso，iso，+FORMAT
--help                                 显示此帮助信息并退出
--version                              显示版本信息并退出
```

### 实例

文件从大到小排序

```
ubuntu@VM-0-14-ubuntu:~/git-work/linux-command$ du -sh * |sort -rh
2.9M    command
1.9M    assets
148K    template
72K     package-lock.json
52K     dist
28K     build
16K     README.md
4.0K    renovate.json
4.0K    package.json
4.0K    LICENSE
```

只显示当前目录下子目录的大小。

```shell
ubuntu@VM-0-14-ubuntu:~/git-work/linux-command$ du -sh ./*/
1.9M    ./assets/
28K     ./build/
2.9M    ./command/
52K     ./dist/
148K    ./template/
```

查看指定目录下文件所占的空间：

```shell
ubuntu@VM-0-14-ubuntu:~/git-work/linux-command/assets$ du ./*
144     ./alfred.png
452     ./chrome-extensions.gif
4       ./dash-icon.png
1312    ./Linux.gif
16      ./qr.png
```

只显示总和的大小:

```shell
ubuntu@VM-0-14-ubuntu:~/git-work/linux-command/assets$ du -s .
1932    .
```

显示总和的大小且易读:

```shell
ubuntu@VM-0-14-ubuntu:~/git-work/linux-command/assets$ du -sh .
1.9M    .
```

# df

显示磁盘的相关信息

## 补充说明

**df命令** 用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

### 语法

```shell
df(选项)(参数)
```

### 选项

```shell
-a或--all：包含全部的文件系统；
--block-size=<区块大小>：以指定的区块大小来显示区块数目；
-h或--human-readable：以可读性较高的方式来显示信息；
-H或--si：与-h参数相同，但在计算时是以1000 Bytes为换算单位而非1024 Bytes；
-i或--inodes：显示inode的信息；
-k或--kilobytes：指定区块大小为1024字节；
-l或--local：仅显示本地端的文件系统；
-m或--megabytes：指定区块大小为1048576字节；
--no-sync：在取得磁盘使用信息前，不要执行sync指令，此为预设值；
-P或--portability：使用POSIX的输出格式；
--sync：在取得磁盘使用信息前，先执行sync指令；
-t<文件系统类型>或--type=<文件系统类型>：仅显示指定文件系统类型的磁盘信息；
-T或--print-type：显示文件系统的类型；
-x<文件系统类型>或--exclude-type=<文件系统类型>：不要显示指定文件系统类型的磁盘信息；
--help：显示帮助；
--version：显示版本信息。
```

### 参数

文件：指定文件系统上的文件。

### 大小格式

显示值以 `--block-size` 和 `DF_BLOCK_SIZE`，`BLOCK_SIZE` 和 `BLOCKSIZE` 环境变量中的第一个可用 `SIZE` 为单位。 否则，单位默认为 `1024` 个字节（如果设置 `POSIXLY_CORRECT`，则为`512`）。

SIZE是一个整数和可选单位（例如：10M是10 * 1024 * 1024）。 单位是K，M，G，T，P，E，Z，Y（1024的幂）或KB，MB，...（1000的幂）。

### 实例

查看系统磁盘设备，默认是KB为单位：

```shell
[root@LinServ-1 ~]# df
文件系统               1K-块        已用     可用 已用% 挂载点
/dev/sda2            146294492  28244432 110498708  21% /
/dev/sda1              1019208     62360    904240   7% /boot
tmpfs                  1032204         0   1032204   0% /dev/shm
/dev/sdb1            2884284108 218826068 2518944764   8% /data1
```

使用`-h`选项以KB以上的单位来显示，可读性高：

```shell
[root@LinServ-1 ~]# df -h
文件系统              容量  已用 可用 已用% 挂载点
/dev/sda2             140G   27G  106G  21% /
/dev/sda1             996M   61M  884M   7% /boot
tmpfs                1009M     0 1009M   0% /dev/shm
/dev/sdb1             2.7T  209G  2.4T   8% /data1
```

查看全部文件系统：

```shell
[root@LinServ-1 ~]# df -a
文件系统               1K-块        已用     可用 已用% 挂载点
/dev/sda2            146294492  28244432 110498708  21% /
proc                         0         0         0   -  /proc
sysfs                        0         0         0   -  /sys
devpts                       0         0         0   -  /dev/pts
/dev/sda1              1019208     62360    904240   7% /boot
tmpfs                  1032204         0   1032204   0% /dev/shm
/dev/sdb1            2884284108 218826068 2518944764   8% /data1
none                         0         0         0   -  /proc/sys/fs/binfmt_misc
```

显示 `public` 目录中的可用空间量，如以下输出中所示：

```shell
df public
# Filesystem     1K-blocks     Used Available Use% Mounted on
# /dev/loop0      18761008 15246924   2554392  86% /d Avail
```