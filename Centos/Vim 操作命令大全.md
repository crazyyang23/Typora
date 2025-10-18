# Vim 操作命令大全

## 目录

1. 基础操作
2. 光标移动
3. 文本编辑
4. 查找替换
5. 多文件操作
6. 窗口管理
7. 高级功能
8. 配置示例

## 基础操作

### 模式切换

text

复制下载

```
i         进入插入模式（当前光标前）
I         行首插入模式
a         进入插入模式（当前光标后）
A         行尾插入模式
o         在下方新建行并进入插入模式
O         在上方新建行并进入插入模式
v         进入可视模式
V         进入行可视模式
Ctrl+v    进入块可视模式
Esc       返回普通模式
```



### 保存退出

text

复制下载

```
:w        保存文件
:q        退出
:q!       强制退出（不保存）
:wq       保存并退出
:x        保存并退出（等同于:wq）
ZZ        保存并退出
ZQ        强制退出（不保存）
```



## 光标移动

### 基本移动

text

复制下载

```
h         左移
j         下移
k         上移
l         右移
w         移动到下一个单词开头
e         移动到当前或下一个单词结尾
b         移动到上一个单词开头
0         移动到行首
^         移动到行首第一个非空字符
$         移动到行尾
gg        移动到文件开头
G         移动到文件末尾
:n        移动到第n行（如:10）
```



### 屏幕滚动

text

复制下载

```
Ctrl+f    向下翻页
Ctrl+b    向上翻页
Ctrl+d    向下半页
Ctrl+u    向上半页
zt        当前行移动到屏幕顶部
zz        当前行移动到屏幕中间
zb        当前行移动到屏幕底部
```



### 搜索移动

text

复制下载

```
/text     向前搜索text
?text     向后搜索text
n         重复上一次搜索
N         反向重复上一次搜索
*         搜索当前光标下的单词
#         反向搜索当前光标下的单词
f{char}   在当前行向前查找字符char
F{char}   在当前行向后查找字符char
```



## 文本编辑

### 删除操作

text

复制下载

```
x         删除当前字符
dd        删除当前行
dw        删除到单词末尾
d$        删除到行尾
d0        删除到行首
dG        删除到文件末尾
dgg       删除到文件开头
:n,md     删除第n到m行
```



### 复制粘贴

text

复制下载

```
yy        复制当前行
yw        复制当前单词
y$        复制到行尾
p         在光标后粘贴
P         在光标前粘贴
"ayy      复制到寄存器a
"ap       从寄存器a粘贴
```



### 撤销重做

text

复制下载

```
u         撤销
Ctrl+r    重做
U         撤销整行修改
```



### 缩进调整

text

复制下载

```
>>        向右缩进当前行
<<        向左缩进当前行
>%        缩进匹配的括号区域
>G        从当前行缩进到文件末尾
=         自动缩进
==        自动缩进当前行
```



## 查找替换

### 基本替换

text

复制下载

```
:s/old/new         替换当前行第一个old为new
:s/old/new/g       替换当前行所有old为new
:%s/old/new/g      替换全文所有old为new
:%s/old/new/gc     替换全文所有old为new（需确认）
:n,ms/old/new/g    替换第n到m行的所有old为new
```



### 正则表达式替换

text

复制下载

```
:%s/\s\+$//g       删除行尾空白字符
:%s/^\s\+//g       删除行首空白字符
:%s/\n/\r/g        将Unix换行符转换为Windows格式
```



## 多文件操作

### 缓冲区管理

text

复制下载

```
:e file    打开新文件
:bn        下一个缓冲区
:bp        上一个缓冲区
:bd        关闭当前缓冲区
:ls        列出所有缓冲区
:b n       切换到第n个缓冲区
```



### 标签页操作

text

复制下载

```
:tabnew    新建标签页
:tabn      下一个标签页
:tabp      上一个标签页
:tabc      关闭当前标签页
:tabo      关闭其他所有标签页
gt         切换到下一个标签页
gT         切换到上一个标签页
```



## 窗口管理

### 窗口分割

text

复制下载

```
:sp        水平分割窗口
:vsp       垂直分割窗口
Ctrl+w h   切换到左窗口
Ctrl+w j   切换到下窗口
Ctrl+w k   切换到上窗口
Ctrl+w l   切换到右窗口
Ctrl+w =   所有窗口等宽等高
Ctrl+w +   增加窗口高度
Ctrl+w -   减少窗口高度
```



## 高级功能

### 宏录制

text

复制下载

```
qa        开始录制宏到寄存器a
q         停止录制
@a        执行寄存器a中的宏
@@        重复上一次执行的宏
```



### 标记功能

text

复制下载

```
ma        设置标记a（a可以是任意字母）
'a        跳转到标记a
`.        跳转到最后修改的位置
''        跳转到上次跳转前的位置
:marks    查看所有标记
```



## 配置示例

### 基础配置文件 (~/.vimrc)

vim

复制下载

```
" 基本设置
set nocompatible        " 不使用vi兼容模式
set number              " 显示行号
set relativenumber      " 显示相对行号
set autoindent          " 自动缩进
set smartindent         " 智能缩进
set tabstop=4           " Tab宽度为4空格
set shiftwidth=4        " 自动缩进宽度为4空格
set expandtab           " 将Tab转换为空格
set softtabstop=4       " 退格键删除4个空格
set cursorline          " 高亮当前行
set showmatch           " 显示匹配的括号
set ignorecase          " 搜索时忽略大小写
set smartcase           " 搜索时智能大小写
set incsearch           " 实时搜索
set hlsearch            " 高亮搜索结果
set backspace=indent,eol,start  " 退格键正常工作

" 键位映射
let mapleader = ","     " 设置leader键为逗号

" 快速保存
nmap <leader>w :w!<cr>

" 快速退出
nmap <leader>q :q!<cr>

" 清除搜索高亮
nmap <silent> <leader>/ :nohlsearch<CR>

" 窗口切换
map <C-j> <C-W>j
map <C-k> <C-W>k
map <C-h> <C-W>h
map <C-l> <C-W>l

" 插件管理 (需要安装Vundle)
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

Plugin 'VundleVim/Vundle.vim'
Plugin 'scrooloose/nerdtree'        " 文件浏览器
Plugin 'vim-airline/vim-airline'    " 状态栏增强
Plugin 'tpope/vim-fugitive'         " Git集成
Plugin 'airblade/vim-gitgutter'     " Git差异显示
Plugin 'preservim/nerdcommenter'    " 快速注释

call vundle#end()
filetype plugin indent on

" NERDTree配置
map <C-n> :NERDTreeToggle<CR>
let NERDTreeShowHidden=1

" 主题设置
syntax enable
set background=dark
colorscheme desert

" 自动命令
autocmd BufWritePre * :%s/\s\+$//e  " 保存时删除行尾空格
autocmd FileType python setlocal expandtab shiftwidth=4 softtabstop=4
autocmd FileType javascript setlocal expandtab shiftwidth=2 softtabstop=2
```



### 实用插件配置

vim

复制下载

```
" vim-airline配置
let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#tabline#formatter = 'default'

" nerdcommenter配置
let g:NERDSpaceDelims = 1
let g:NERDCompactSexyComs = 1
let g:NERDDefaultAlign = 'left'

" gitgutter配置
set updatetime=100
let g:gitgutter_max_signs = 500
```



### 实用技巧配置

vim

复制下载

```
" 快速编辑.vimrc
nnoremap <leader>ev :vsplit $MYVIMRC<cr>
nnoremap <leader>sv :source $MYVIMRC<cr>

" 快速运行当前文件
autocmd FileType python nnoremap <buffer> <leader>r :exec '!python' shellescape(@%, 1)<cr>
autocmd FileType javascript nnoremap <buffer> <leader>r :exec '!node' shellescape(@%, 1)<cr>

" 代码折叠设置
set foldmethod=indent
set foldlevel=99
nnoremap <space> za
```



## 实用案例

### 案例1：批量重命名变量

vim

复制下载

```
" 将所有的foo重命名为bar，每次替换前确认
:%s/foo/bar/gc
```



### 案例2：多行注释

vim

复制下载

```
" 使用nerdcommenter插件
<leader>cc    " 注释选中行
<leader>cu    " 取消注释选中行
```



### 案例3：代码格式化

vim

复制下载

```
" 格式化整个文件
gg=G

" 格式化选中区域
1. 进入可视模式 (v)
2. 选择要格式化的代码
3. 按 = 自动格式化
```



### 案例4：多文件搜索替换

vim

复制下载

```
" 在多个文件中搜索替换
:vimgrep /pattern/ **/*.py
:copen      " 打开quickfix窗口
:cfdo %s/pattern/replacement/g | update
```



这个Vim命令大全涵盖了从基础到高级的常用操作，配合配置文件示例可以帮助你打造一个高效的开发环境。建议先从基础命令开始练习，逐步掌握更高级的功能。