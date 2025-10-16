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