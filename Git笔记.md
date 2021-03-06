[TOC]

**此笔记是读[ProGit](https://www.progit.cn/)之后写的，大部分内容（包括图片）都是直接从书中复制下来的。**

# Git起步

## Git基础

### 不同版本控制系统使用的存储方式以及各自支持的管理方式

![image-20200416193228176](.assets/image-20200416193228176.png)

- 分布式的版本控制系统：每个人的电脑就是一个仓库。先写到本地的仓库，然后推到网上的仓库。即便网上的仓库崩了本地也有，反之亦然。Git就是这种实现.
- 集中式版本控制系统：必须通过网络连接到中心服务器，无法直接在本地更改。SVN就是这种实现

### 直接记录快照，而非差异比较

Git和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于Git对待数据的方法。概念上来区分，其它大部分系统以文件变更列表的方式存储信息。这类系统（CVS、Subversion、Perforce、Bazaar 等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。

![image-20200416164520305](.assets/image-20200416164520305.png)

Git不按照以上方式对待或保存数据。反之，Git 更像是把数据看作是对小型文件系统的一组快照。每次你提交更新，或在Git中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。为了高效，如果文件没有修改，Git不再重新存储该文件，而是只保留一个链接指向之前存储的文件。Git对待数据更像是一个 快照流。

![image-20200416164556205](.assets/image-20200416164556205.png)

### 近乎所有操作都是本地执行

Git不是直接操作服务器上的仓库，Git的绝大多数操作都只需要访问本地文件和资源。

### 保证完整性

Git用以SHA-1算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个SHA-1哈希值，作为指纹字符串。该字符串由40位16进制字符组成，看起来就像是：

`24b9da6552252987aa493b52f8696cd6d3b00373`

Git数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。

### Git一般只添加数据

你执行的 Git 操作，几乎只往 Git 数据库中增加数据。很难让 Git 执行任何不可逆操作，或者让它以任何方式清
除数据。同别的 VCS 一样，未提交更新时有可能丢失或弄乱修改的内容；但是一旦你提交快照到 Git 中，就难以
再丢失数据，特别是如果你定期的推送数据库到其它仓库的话。

### 三种状态

对于任何一个文件，在Git 内都只有三种状态：已提交（committed），已修改（modified）和已暂存（staged）。

![image-20200416165346322](.assets/image-20200416165346322.png)

- 已提交表示该文件已经被安全地保存在本地数据库中了；
- 已修改表示修改了文件，但还没保存到数据库中
- 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。

![image-20200416193500668](.assets/image-20200416193500668-1587224107909.png)

- 工作目录是对项目的某个版本独立提取出来的内容。这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

- 暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。有时候也被称作`‘索引’'，不过一般说法还是叫暂存区域。
- Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。

**基本的Git 工作流程如下所示**

1. 在工作目录中修改某些文件。
2. 对这些修改了的文件作快照，并保存到暂存区域。
3. 提交更新，将保存在暂存区域的文件快照转储到git目录中。

我们可以从文件所处的位置来判断状态：如果是git目录中保存着的特定版本文件，就属于已提交状态；如果作了修改并已放入暂存区域，就属于已暂存状态；如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。

## 安装Git

**RPM包安装**

```BASH
[10.208.3.20 root@test-3:~]# yum install git -y
```

**如果采用源码安装，可以使用下面这个技巧**

如果是源码安装的话就进入都contrib/completion目录，找到git-completion.bash文件，将其复制到/etc/bash_completion.d目录中。这个文件可以实现<TAB><TAB>自动补全的功能

```BASH
[10.208.3.20 root@test-3:~/testapp]# rpm -qf /etc/bash_completion.d/git
git-1.8.3.1-21.el7_7.x86_64
```

## 初次运行Git前的配置

Git 提供了一个叫做git config 的工具（实际是git-config 命令，只不过可以通过git 加一个名字来呼叫此命令），专门用来配置或读取相应的工作环境变量。而正是由这些环境变量，决定了Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

- /etc/gitconfig文件：系统中对所有用户都普遍适用的配置。若使用git config 时用--system 选项，读写的就是这个文件。
- ~/.gitconfig文件：用户目录下的配置文件只适用于该用户。若使用git config 时用--global 选项，读写的就是这个文件。
- 当前项目的git 目录中的配置文件（也就是工作目录中的.git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以.git/config 里的配置会覆盖/etc/gitconfig 中的同名变量。

**用户信息（必须）**

第一个要配置的是你个人的用户名称和电子邮件地址。这两条配置很重要，每次Git 提交时都会引用这两条信息，说明是谁提交了更新，所以会随更新内容一起被永久纳入历史记录：

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

如果用了--global 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。如果要在某个特定的项目中使用其他名字或者电邮，只要去掉--global选项重新配置即可，新的设定保存在当前项目的.git/config文件里。

**文本编辑器**

接下来要设置的是默认使用的文本编辑器。Git 需要你输入一些额外消息的时候，会自动调用一个外部文本编辑器给你用。默认会使用操作系统指定的默认编辑器，一般可能会是vi或者vim。如果你有其他偏好，比如Emacs的话，可以重新设置：

```bash
$ git config --global core.editor emacs
```

Git 可以理解kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和opendiff 等合并工具的输出信息。也可以开发自己的工具。

**查看配置信息**

```bash
$ git config --list
user.name=Scott Chacon
user.email=schacon@gmail.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...
```

有时候会看到重复的变量名，那就说明它们来自不同的配置文件（比如/etc/gitconfig和~/.gitconfig），不过最终Git 实际采用的是最后一个。你可以通过输入 git config <key> 来检查 Git 的某一项配置：

```bash
$ git config user.name
Scott Chacon
```

# Git基础

## 获取Git仓库

有两种取得 Git 项目仓库的方法。

- 在现有项目或目录下导入所有文件到 Git 中。
- 从一个服务器克隆一个现有的 Git 仓库。

### 初始化Git仓库

如果你打算使用Git 来对现有的项目进行管理，你只需要进入该项目目录并输入`git init`，该命令将创建一个名为 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是Git 仓库的骨干。

```BASH
[10.208.3.20 root@test-3:~]# mkdir testapp
[10.208.3.20 root@test-3:~]# cd testapp/
[10.208.3.20 root@test-3:~/testapp]# git init
[10.208.3.20 root@test-3:~/testapp]# tree -aF ./        ##执行git init之后，会创建一个.git的隐藏目录，该目录的目录结构如下
./
└── .git/
    ├── branches/
    ├── config        ##当前仓库的配置文件，除了此文件还有全局配置文件~/.gitconfig和系统配置文件/etc/gitconfig
    ├── description   ##提供GitWeb程序使用
    ├── HEAD        ##指示目前被检出的分支（指向当前分支的最新一个commit）
    ├── hooks/      ##客户端或服务端的钩子脚本
    │   ├── applypatch-msg.sample*
    │   ├── commit-msg.sample*
    │   ├── post-update.sample*
    │   ├── pre-applypatch.sample*
    │   ├── pre-commit.sample*
    │   ├── prepare-commit-msg.sample*
    │   ├── pre-push.sample
    │   ├── pre-rebase.sample*
    │   └── update.sample*
    ├── info/    ##包含一个全局性排除文件，用以放置那些不希望被记录在.gitignore文件中的忽略模式
    │   └── exclude    ##git ls-lists默认读取的忽略文件列表
    ├── objects/        ##存储所有数据内容
    │   ├── info/
    │   └── pack/
    └── refs/            ##存储指向数据（分支）的提交对象的指针
        ├── heads/
        └── tags/
     此时未创建的index文件	  ##保存暂存区信息
```

如果指定--bare选项，可以创建一个裸存储库（没有.git目录，所有内容都直接放到指定的目录下面）

```BASH
[10.208.3.20 root@test-3:~]# git init --bare myapp
Initialized empty Git repository in /root/myapp/
[10.208.3.20 root@test-3:~]# ll -a myapp/
total 16
drwxr-xr-x   7 root root  119 2020-04-10 19:59:00 .
dr-xr-x---. 19 root root 4096 2020-04-10 19:59:00 ..
drwxr-xr-x   2 root root    6 2020-04-10 19:59:00 branches
-rw-r--r--   1 root root   66 2020-04-10 19:59:00 config
-rw-r--r--   1 root root   73 2020-04-10 19:59:00 description
-rw-r--r--   1 root root   23 2020-04-10 19:59:00 HEAD
drwxr-xr-x   2 root root  242 2020-04-10 19:59:00 hooks
drwxr-xr-x   2 root root   21 2020-04-10 19:59:00 info
drwxr-xr-x   4 root root   30 2020-04-10 19:59:00 objects
drwxr-xr-x   4 root root   31 2020-04-10 19:59:00 refs
[10.208.3.20 root@test-3:~/testapp]# cat .git/config
[core]
    repositoryformatversion = 0
    filemode = true
    bare = true
    logallrefupdates = true
[user]
    name = tianbao3
```

### 克隆现有仓库

如果你想获得一份已经存在了的 Git 仓库的拷贝，就要用到 git clone 命令。

克隆仓库的命令格式是`git clone [url]`。比如，要克隆 Git 的可链接库 libgit2，可以用下面的命令：

```BASH
$ git clone https://github.com/libgit2/libgit2
```

这会在当前目录下创建一个名为 “libgit2” 的目录，并在这个目录下初始化一个 .git 文件夹，从远程仓库拉取下所有数据放入 .git 文件夹，然后从中读取最新版本的文件的拷贝。如果你进入到这个新建的 libgit2 文件夹，你会发现所有的项目文件已经在里面了，准备就绪等待后续的开发和使用。如果你想在克隆远程仓库的时候，自定义本地仓库的名字，你可以使用如下命令：

```BASH
$ git clone https://github.com/libgit2/libgit2 mylibgit
```

这将执行与上一个命令相同的操作，不过在本地创建的仓库名字变为 mylibgit。
Git 支持多种数据传输协议。上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用SSH传输协议，比如 user@server:path/to/repo.git 。

## 添加文件到到暂存区

`$ git add <filename>`

```BASH
[10.208.3.20 root@test-3:~/testapp]# cp /etc/passwd ./
[10.208.3.20 root@test-3:~/testapp]# git add /etc/passwd
[10.208.3.20 root@test-3:~/testapp]# git ls-files -s
100644 f8c3c5e14b617e7c4613d4cbcf5b3ffb229adcb8 0    passwd
```

filename支持通配符，也可以使用`git add . `来表示当前目录及其子目录的所有文件。

**查看暂存区的文件**

`$ git ls-files -s`

```BASH
[10.208.3.20 root@test-3:~/testapp]# git ls-files -s
100644 f8c3c5e14b617e7c4613d4cbcf5b3ffb229adcb8 0    passwd
```

**查看工作目录中有哪些未见未被加入到暂存区中**

`$ git ls-files -o`

**查看对象库中的文件**

`git cat-file -p <object>` 

```BASH
[10.208.3.20 root@test-3:~/testapp]# git cat-file -p f8c3c5e14b617e7c4613d4cbcf5b3ffb229adcb8 | head -n1
root:x:0:0:root:/root:/bin/bash
[10.208.3.20 root@test-3:~/testapp]# git cat-file -p f8c3c5e14 | head -n1
root:x:0:0:root:/root:/bin/bash
[10.208.3.20 root@test-3:~/testapp]# git cat-file -p f8c3c5e | head -n1
root:x:0:0:root:/root:/bin/bash
[10.208.3.20 root@test-3:~/testapp]# git cat-file -p f8c3c | head -n1
root:x:0:0:root:/root:/bin/bash
[10.208.3.20 root@test-3:~/testapp]# git cat-file -p f8c3 | head -n1
root:x:0:0:root:/root:/bin/bash
[10.208.3.20 root@test-3:~/testapp]# git cat-file -p f8c | head -n1
fatal: Not a valid object name f8c
```

支持使用短格式的hash码来查看文件（基本上表示只要唯一标识就可以，但是也不能太短）

**计算文件的hash码**

`git hash-object`

```BASH
[10.208.3.20 root@test-3:~/testapp]# git hash-object /etc/hosts82fc0291e78c3f0a681577330c31ad70c009da5e
```

**查看工作目录状态**

这回对比工作目录中的文件与暂存区、HEAD指向的commit的区别

```BASH
[10.208.3.20 root@test-3:~/testapp]# git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#    new file:   passwd
#
```

## 提交到仓库

`git commit -m 'xxx'`

如果不指定-m，git会使用文件编辑器打来一个文件，让用户在这个文件数输入消息，保存退出后才可以提交称成功。

```BASH
[10.208.3.20 root@test-3:~/testapp]# git commit -m "v0.1"
[master (root-commit) dd5b5e8] v0.1
1 file changed, 28 insertions(+)
create mode 100644 passwd
[10.208.3.20 root@test-3:~/testapp]# git status
# On branch master
nothing to commit, working directory clean
```

## 告诉Git哪些文件不需要提交到仓库

这可以通过在工作区中添加`.gitignore`文件来实现

**.gitignore文件配置语法**

- 以斜杠/开头表示目录；

- 以星号*通配多个字符；

- 以问号?通配单个字符

- 以方括号[]包含单个字符的匹配列表；

- 以叹号!表示不忽略(跟踪)匹配到的文件或目录；

此外，git 对于 .ignore 配置文件是按行从上到下进行规则匹配的，意味着如果前面的规则匹配的范围更大，则后面的规则将不会生效；

**配置示例**

我的python项目中，`__pycache__`不需要提交到仓库中

```bash
[10.208.3.3 root@ntp:~/alien_invasion]# git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	__pycache__/
nothing added to commit but untracked files present (use "git add" to track)
```

此时我在python项目的工作区中，添加一个.gitignore文件

```BASH
[10.208.3.3 root@ntp:~/alien_invasion]# cat .ig^C
[10.208.3.3 root@ntp:~/alien_invasion]# ll -a
total 36
drwxr-xr-x   5 root root  192 2020-06-05 14:49:50 .
dr-xr-x--x. 15 root root 4096 2020-06-05 14:49:50 ..
-rw-r--r--   1 root root  783 2020-06-05 11:06:55 alien_invasion.py
-rw-r--r--   1 root root 1190 2020-06-05 12:05:29 game_functions.py
drwxr-xr-x   8 root root  166 2020-06-05 14:48:17 .git
-rw-r--r--   1 root root   14 2020-06-05 14:45:19 .gitignore		#这就是添加的文件
drwxr-xr-x   2 root root   40 2020-06-04 18:59:16 images
-rw-r--r--   1 root root  369 2020-06-04 19:13:27 maliao.py
drwxr-xr-x   2 root root  130 2020-06-05 12:09:46 __pycache__
-rw-r--r--   1 root root  263 2020-06-05 14:24:20 pygame_test.py
-rw-r--r--   1 root root  293 2020-06-05 10:45:32 settings.py
-rw-r--r--   1 root root 1326 2020-06-05 12:09:45 ship.py
[10.208.3.3 root@ntp:~/alien_invasion]# cat .gitignore 				#文件内容如下
/__pycache__/		#这表示不会将__pycache__目录添加到仓库中
```

此时再次使用git status就看不到`__pycache__`目录了，但是会看到.gitignore的提示，将这个文件提交到版本库对其进行版本控制即可

```BASH
[10.208.3.3 root@ntp:~/alien_invasion]# git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	.gitignore
nothing added to commit but untracked files present (use "git add" to track)
[10.208.3.3 root@ntp:~/alien_invasion]# git add .
[10.208.3.3 root@ntp:~/alien_invasion]# git commit -m 202006051454
[master 2245a2c] 202006051454
 1 file changed, 1 insertion(+)
 create mode 100644 .gitignore
```

## 删除Git中的文件

使用系统的rm命令只能删除工作目录下得文件，但是无法删除索引中的记录，对于git中的文件需要使用下面的命令进行删除：

**删除工作目录中的文件和暂存区中索引里的条目**

`git rm <filename>`

**删除暂存区中索引里的条目**

`git rm --cached <filename>`

**git rm 命令后面可以列出文件或者目录的名字，也可以使用 glob 模式。比方说：**

```BASH
$ git rm log/\*.log
$ git rm \*~
```

**使用`git rm -f <filename>`删除文件**

对文件执行了git add之后，如果没有执行git commit。对工作目录中文件内容改动后，就不允许使用git rm删除文件。需要先对工作目录中改动了的文件执行git add后才可以删除。或者使用`git rm -f XXX`强制移除。这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被 Git 恢复。

```BASH
[10.208.3.20 root@test-3:~/testapp]# echo 456 >> password4
[10.208.3.20 root@test-3:~/testapp]# git rm password4
error: 'password4' has staged content different from both the file and the HEAD
(use -f to force removal)
[10.208.3.20 root@test-3:~/testapp]# git rm -f password4
rm 'password4'
```

## 对文件改名

`git mv <oldname> <newname>`

```BASH
[10.208.3.20 root@test-3:~/testapp]# git mv password3 passwd2
[10.208.3.20 root@test-3:~/testapp]# git ls-files -s
100644 190a18037c64c43e6b11489df4bf0b9eb6d2c9bf 0    hellow/1/3/le
100644 190a18037c64c43e6b11489df4bf0b9eb6d2c9bf 0    hellow/1/4/le
100644 f8c3c5e14b617e7c4613d4cbcf5b3ffb229adcb8 0    passwd
100644 f8c3c5e14b617e7c4613d4cbcf5b3ffb229adcb8 0    passwd2
[10.208.3.20 root@test-3:~/testapp]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#    renamed:    password3 -> passwd2
#
[10.208.3.20 root@test-3:~/testapp]# git commit -m "v0.1.7"
[master f6c0544] v0.1.7
1 file changed, 0 insertions(+), 0 deletions(-)
rename password3 => passwd2 (100%)
```

**上面执行的git mv就相当于运行了下面三条命令**

```BASH
$ mv password3 passwd2
$ git rm password3 
$ git add passwd2
```

## 查看提交日志

查看指定分支的提交日志，如果不指定<branchname>，默认查看当前分支的

`git log <branchname>`

**git log的部分选项**

- -p 按补丁格式显示每个更新之间的差异。

- --stat 显示每次更新的文件修改统计信息。
- --shortstat 只显示--stat 中最后的行数修改添加移除统计。
- --name-only 仅在提交信息后显示已修改的文件清单。
- --name-status 显示新增、修改、删除的文件清单。
- --abbrev-commit 仅显示SHA-1 的前几个字符，而非所有的40 个字符。
- --relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）。
- --graph 显示ASCII 图形表示的分支合并历史。
- --pretty 使用其他格式显示历史提交信息。可用的选项包括oneline，short，full，fuller 和format（后跟指定格式）。

**限制 git log 输出的选项**

- -(n) 仅显示最近的 n 条提交
- --since, --after 仅显示指定时间之后的提交。
- --until, --before 仅显示指定时间之前的提交。
- --author 仅显示指定作者相关的提交。
- --committer 仅显示指定提交者相关的提交。
- --grep 仅显示含指定关键字的提交
- -S 仅显示添加或移除了某个关键字的提交

**常用的格式占位符写法及其代表的意义**

- %H 提交对象（commit）的完整哈希字串
- %h 提交对象的简短哈希字串
- %T 树对象（tree）的完整哈希字串
- %t 树对象的简短哈希字串
- %P 父对象（parent）的完整哈希字串
- %p 父对象的简短哈希字串
- %an 作者（author）的名字
- %ae 作者的电子邮件地址
- %ad 作者修订日期（可以用-date= 选项定制格式）
- %ar 作者修订日期，按多久以前的方式显示
- %cn 提交者(committer)的名字
- %ce 提交者的电子邮件地址
- %cd 提交日期
- %cr 提交日期，按多久以前的方式显示
- %s 提交说明

**查看当前分支的提交日志**

```bash
[10.208.3.20 root@test-3:~/testapp]# git log
commit dd5b5e888b060758f265d5459db495116a4b069a
Author: tianbao3 <tianbao1@example.com>
Date:   Sun Apr 12 20:14:17 2020 +0800
    v0.1
```

**使用-p选项，展开显示每次提交的内容差异，用-2仅显示最近两次更新。**

在做代码审查，或者要快速浏览其他协作者提交的更新都作了哪些改动时，就可以用这个选项。

```bash
[10.208.3.20 root@test-3:~/testapp]# git log -p -2
commit bbc1db8131aabece95e8af530b6fb2ca8109df7f
Author: tianbao3 <tianbao1@example.com>
Date:   Tue Apr 14 20:25:29 2020 +0800

    0.3.2

diff --git a/passwd b/passwd
index f8c3c5e..b2713d1 100644
--- a/passwd
+++ b/passwd
@@ -26,3 +26,4 @@ jumpserver_service_admin:x:1000:1000::/home/jumpserver_service_admin:/bin/bash
 tianbao1:x:1001:1001::/home/tianbao1:/bin/bash
 nginx:x:993:991::/home/nginx:/sbin/nologin
 tianbao3:x:1002:1002::/home/tianbao3:/bin/bash
+123

commit e01d1d77c424e87833a57372d0a578cefa6ffd75
Author: tianbao3 <tianbao1@example.com>
Date:   Mon Apr 13 15:40:29 2020 +0800

    0.3.1

diff --git a/passwd2 b/passwd2
deleted file mode 100644
index f51cd94..0000000
--- a/passwd2
+++ /dev/null
@@ -1,29 +0,0 @@
-root:x:0:0:root:/root:/bin/bash
-bin:x:1:1:bin:/bin:/sbin/nologin
-daemon:x:2:2:daemon:/sbin:/sbin/nologin
-adm:x:3:4:adm:/var/adm:/sbin/nologin
-lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
-sync:x:5:0:sync:/sbin:/bin/sync
-shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
-halt:x:7:0:halt:/sbin:/sbin/halt
-mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
-operator:x:11:0:operator:/root:/sbin/nologin
-games:x:12:100:games:/usr/games:/sbin/nologin
-ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
-nobody:x:99:99:Nobody:/:/sbin/nologin
-systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
-dbus:x:81:81:System message bus:/:/sbin/nologin
-polkitd:x:999:997:User for polkitd:/:/sbin/nologin
-postfix:x:89:89::/var/spool/postfix:/sbin/nologin
-sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
-chrony:x:998:996::/var/lib/chrony:/sbin/nologin
-tcpdump:x:72:72::/:/sbin/nologin
-mysql:x:997:995::/home/mysql:/sbin/nologin
-openvpn:x:996:994:OpenVPN:/etc/openvpn:/sbin/nologin
-redis:x:995:993:Redis Database Server:/var/lib/redis:/sbin/nologin
-zabbix:x:994:992:Zabbix Monitoring System:/var/lib/zabbix:/sbin/nologin
-jumpserver_service_admin:x:1000:1000::/home/jumpserver_service_admin:/bin/bash
-tianbao1:x:1001:1001::/home/tianbao1:/bin/bash
-nginx:x:993:991::/home/nginx:/sbin/nologin
-tianbao3:x:1002:1002::/home/tianbao3:/bin/bash
-test123
```

**--stat，仅显示简要的增改行数统计**

每个提交都列出了修改过的文件，以及其中添加和移除的行数，并在最后列出所有增减行数小计。

```bash
[10.208.3.20 root@test-3:~/testapp]# git log --stat
commit bbc1db8131aabece95e8af530b6fb2ca8109df7f
Author: tianbao3 <tianbao1@example.com>
Date:   Tue Apr 14 20:25:29 2020 +0800

    0.3.2

 passwd | 1 +
 1 file changed, 1 insertion(+)

commit e01d1d77c424e87833a57372d0a578cefa6ffd75
Author: tianbao3 <tianbao1@example.com>
Date:   Mon Apr 13 15:40:29 2020 +0800

    0.3.1

 passwd2 | 29 -----------------------------
 1 file changed, 29 deletions(-)

```

**--pretty 选项，可以指定使用完全不同于默认格式的方式展示提交历史。比如用oneline 将每个提交放在一行显示，这在提交数很大时非常有用。此外还有short，full和fuller可以用**

```bash
[10.208.3.20 root@test-3:~/testapp]# git log --pretty=oneline
bbc1db8131aabece95e8af530b6fb2ca8109df7f 0.3.2
e01d1d77c424e87833a57372d0a578cefa6ffd75 0.3.1
5e8cf89c4f2a3972f611bbe4a84d896339621b6c Merge branch 'master' of github.com:AllenLTB/testapp
8b47c3815aae2330cb74669d1f0a22ca74185703 Merge branch 'develop'
1277176adc1f2d3bfae575a6a782ea5525a1eddb v0.1.10
a43d4dd503f42800f3533f645cddef4427dea2a1 v0.2.2
34c4b0638c69575d00df5a53f4cecf3891f05208 v0.2.1
e575f3105c83c30b6e6d23be16af699907253c64 v0.1.8
f6c0544fc0a4cc22bb2255bde75afe6cc9d227c3 v0.1.7
29bef5122413c65cae2432070e4dac98eb350f4b v0.1.5
007587c304af1745e84440956ef657eba709e155 v0.1.4
052e7966f4c8be449b2cf7f60019aaa8b15a51ae v0.1.3
c48eb87a2de80f42956ccad5d31d72b12a61cbe4 v0.1
dd5b5e888b060758f265d5459db495116a4b069a v0.1
ce1d4661aacb5e9dc42690843e11cca9d8156cff Initial commit
```

**用oneline 或format 时结合--graph 选项，可以看到开头多出一些ASCII 字符串表示的简单图形，形象地展示了每个提交所在的分支及其分化衍合情况。**

```bash
[10.208.3.20 root@test-3:~/testapp]# git log --graph --pretty=oneline
* bbc1db8131aabece95e8af530b6fb2ca8109df7f 0.3.2
* e01d1d77c424e87833a57372d0a578cefa6ffd75 0.3.1
*   5e8cf89c4f2a3972f611bbe4a84d896339621b6c Merge branch 'master' of github.com:AllenLTB/testapp
|\  
| * ce1d4661aacb5e9dc42690843e11cca9d8156cff Initial commit
*   8b47c3815aae2330cb74669d1f0a22ca74185703 Merge branch 'develop'
|\  
| * a43d4dd503f42800f3533f645cddef4427dea2a1 v0.2.2
| * 34c4b0638c69575d00df5a53f4cecf3891f05208 v0.2.1
* | 1277176adc1f2d3bfae575a6a782ea5525a1eddb v0.1.10
|/  
* e575f3105c83c30b6e6d23be16af699907253c64 v0.1.8
* f6c0544fc0a4cc22bb2255bde75afe6cc9d227c3 v0.1.7
* 29bef5122413c65cae2432070e4dac98eb350f4b v0.1.5
* 007587c304af1745e84440956ef657eba709e155 v0.1.4
* 052e7966f4c8be449b2cf7f60019aaa8b15a51ae v0.1.3
* c48eb87a2de80f42956ccad5d31d72b12a61cbe4 v0.1
* dd5b5e888b060758f265d5459db495116a4b069a v0.1
```

**format，可以定制要显示的记录格式，这样的输出便于后期编程提取分析**

```bash
[10.208.3.20 root@test-3:~/testapp]# git log --pretty=format:"%h - %an, %ar : %s"
bbc1db8 - tianbao3, 13 minutes ago : 0.3.2
e01d1d7 - tianbao3, 29 hours ago : 0.3.1
5e8cf89 - tianbao3, 29 hours ago : Merge branch 'master' of github.com:AllenLTB/testapp
8b47c38 - tianbao3, 2 days ago : Merge branch 'develop'
1277176 - tianbao3, 2 days ago : v0.1.10
a43d4dd - tianbao3, 2 days ago : v0.2.2
34c4b06 - tianbao3, 2 days ago : v0.2.1
e575f31 - tianbao3, 2 days ago : v0.1.8
f6c0544 - tianbao3, 2 days ago : v0.1.7
29bef51 - tianbao3, 2 days ago : v0.1.5
007587c - tianbao3, 2 days ago : v0.1.4
052e796 - tianbao3, 2 days ago : v0.1.3
c48eb87 - tianbao3, 2 days ago : v0.1
dd5b5e8 - tianbao3, 2 days ago : v0.1
ce1d466 - AllenLTB, 3 days ago : Initial commit
```

**如果要查看 Git 仓库中，2008 年 10 月期间，Junio Hamano 提交的但未合并的测试文件，可以用下面的查询命令**

```bash
$ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link
HEAD
d1a43f2 - reset --hard/read-tree --reset -u: remove unmerged new paths
51a94af - Fix "checkout --track -b newbranch" on detached HEAD
b0ad11e - pull: allow "git pull origin $something:$current_branch" into an
unborn branch
```

## 撤销操作

**重新提交**

`git commit --amend`

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。此时，可以运行带有 --amend 选
项的提交命令尝试重新提

```bash
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

上面的三条命令最终得到一个提交，第二个提交命令修正了第一个的提交内容。

**清除操作**

删除工作区中未跟踪的目录和文件，有时候我们错误的将一个目录解压到工作目录的时候，可以yoghurt

`git clean -d -f` 

**恢复操作**

`git reset --{soft|mixed|hard} <commit>`

注意：不管是用哪种模式，这个命令都会将当前提交到恢复提交之间的所有提交删除。此后就看不到了。

- soft：只恢复提交到指定版本，不恢复索引和工作目录（将HEAD引用指向给定的提交，但不影响索引和工作目录）。比如对之前的提交不满意，将这个提交少了，要重新提交。

- mixed：只恢复提交和索引和指定版本，不恢复工作目录（将HEAD引用指向给定的提交，并将索引内容改变为指定提交的快照；但不改变工作目录；）。在此种情况下想要将新版本重现（继续变为最新，未恢复之前），只能先执行add，在执行提交。

- hard：版本、索引和工作目录都恢复到指定版本（将HEAD引用指向给定的提交、将索引内容改变为指定提交的快照，并改变工作目录中的内容反映指定提交的内容；）。在此种情况下，无法重现。

![image-20200416205214088](.assets/image-20200416205214088.png)



**git暂存区工作区撤销和删除**

1、撤掉

a、如果还没 git add file ，使用该指令进行撤销：  git checkout -- fileName

b、如果已经git add file  ， 但是没有 git commit -m ""  分两步操作：

- git reset HEAD readme.txt

- git  status

- git checkout -- file

c、如果已经git add file 并且已经 git commit ,那么回退版本办法是：

- 通过 git log 或者 git log --pretty=oneline 、git reflog
- 找到对应的commit id进行回退：git reset --hard 1094a

**2、删除**

现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令git rm删掉，并且git commit：

- git rm test.txt
- git commit -m "remove test.txt"

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：

git checkout -- test.txt


## 对比不同

查看工作目录和索引的区别

`git diff`

索引和已提交的版本进行比较，如果不指定版本默认就是HEAD（当前分支的最近的一个版本）

`git diff --cached  <commit>`

比较两个版本之间的区别

`git diff <commit> <commit>`

比较工作目录和指定版本之间的区别，如果指定为HEAD就是比较工作目录和当前分支的最近一次提交

`git diff <commit>`

![image-20200419010918795](.assets/image-20200419010918795.png)

## 远程仓库的使用

**添加一个新的远程 Git 仓库**

`git remote add <shortname> <url>`

```BASH
[10.208.3.20 root@test-3:~/testapp]# git remote add testapp git@github.com:AllenLTB/testapp.git
```

**获取远程仓库名字**

`git remote`

```BASH
[10.208.3.20 root@test-3:~/testapp]# git remote
testapp
```

**指定选项 -v可以额外显示URL信息**

`git remote -v`

```BASH
[10.208.3.20 root@test-3:~/testapp]# git remote -v
testapp	git@github.com:AllenLTB/testapp.git (fetch)
testapp	git@github.com:AllenLTB/testapp.git (push)
```

**查看远端仓库**

```
git remote show 
 [10.208.3.20 root@test-3:~/testapp]# git remote show testapp
 * remote testapp
   Fetch URL: git@github.com:AllenLTB/testapp.git
   Push  URL: git@github.com:AllenLTB/testapp.git
   HEAD branch: master
   Remote branches:
     develop tracked
     master  tracked
   Local refs configured for 'git push':
     develop pushes to develop (up to date)
     master  pushes to master  (local out of date)
```

**远端仓库的移除**

```
git remote remobe 
```

**远端仓库的重命名**

```
git remote rename  
```

**从远程仓库中抓取和拉取**

- git fetch 命令会将数据拉取到你的本地仓库，它并不会自动合并或修改你当前的工作。当准备好时你必须手动将其合并入你的工作。
- git pull 命令来自动的抓取然后合并远程分支到当前分支
  - pull=fetch+merge

- git clone 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 master 分支（或不管是什么名字的默认分支）。如果你使用 clone 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。
  - clone=fetch+checkout

**将远程仓库的所有分支拉下来**

`git fetch <remote-name>`

**将远程仓库的指定拉下来**

`git fetch <remote-name> <remotebranch>`

**将远程仓库的指定分支拉下来作为一个新分支**

`git fetch <remote-name> <remotebranch>:<refspec>`

**把远程仓库的所有分支拉下来后合并** 

`git pull <remote-name>`

**取回远端服务器指定分支，然后与本地的指定分支合并**

`git pull <remote-name> <remotebranch>:<localbranch>` 

## 克隆Git仓库

clone=fetch+checkout

**克隆远端仓库**

`git clone <url> [dirname]`

```BASH
$ git clone https://github.com/libgit2/libgit2
```

这将执行与上一个命令相同的操作，不过在本地创建的仓库名字变为 mylibgit。

```bash
$ git clone https://github.com/libgit2/libgit2 mylibgit
```

## 标签

**列出标签**

`git tag`

```BASH
[10.208.3.20 root@test-3:~/testapp]# git tag
202004122153
202004122157
202004131711
test1
```

**指定-l可以使用glob匹配**

```BASH
$ git tag -l 'v1.8.5*'
```

**为指定提交创建标签**

`git tag <tagname> [commit|object]`

如果不指定commit，则为当前commit创建tag

**查看tag的信息**

`git show <tagname>`

**共享标签**

默认情况下，git push 命令并不会传送标签到远程仓库服务器上。你可以运行下面这个命令进行推送

`git push origin <tagname>`

如果想要一次性推送很多标签，也可以使用带有 --tags 选项的 git push 命令。这将会把所有不在远程仓库
服务器上的标签全部传送到那里。

**检出标签**

用特定的标签上创建一个新分支

`git checkout -b [branchname] [tagname]`

# Git分支

## 分支简介

现在，Git 仓库中有五个对象：三个 blob 对象（保存着文件快照）、一个树对象（记录着目录结构和 blob 对象
索引）以及一个提交对象（包含着指向前树对象的指针和所有提交信息）。

![image-20200416184040960](.assets/image-20200416184040960.png)



做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。

![image-20200416184125689](.assets/image-20200416184125689.png)

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。Git 的默认分支名字是 master。在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 master 分支。它会在每次的提交操作中自动向前移动。

![image-20200416184233141](.assets/image-20200416184233141.png)

> Note：Git 的 “master” 分支并不是一个特殊分支。它就跟其它分支完全没有区别。之所以几乎每一个仓库都有 master 分支，是因为 git init 命令默认创建它，并且大多数人都懒得去改动它。



## 创建分支

**Git分支的命名法则**

- 可以使用/，但不能以/结尾；
- 不能以-开头；
- 以位于/后面的组件，不能以.开头；
- 不能使用连续的...；
- 不能使用空白字符；
- 不能使用^, ~, ?, *，[等；

Git创建新分支，其实就是创建了一个可移动的新的指针。下面创建一个叫做testing的分支：

```bash
$ git branch testing
```

这会在当前所在提交对象上创建一个指针

![image-20200416184911624](.assets/image-20200416184911624.png)

那么，Git 又是怎么知道当前在哪一个分支上呢？也很简单，它有一个名为 HEAD 的特殊指针。请注意它和许多其它版本控制系统（如 Subversion 或 CVS）里的 HEAD 概念完全不同。在 Git 中，它是一个指针，指向当前所在的本地分支（译注：将 HEAD 想象为当前分支的别名）。在本例中，你仍然在 master 分支上。因为 git branch 命令仅仅 创建 一个新分支，并不会自动切换到新分支中去。

![image-20200416185133197](.assets/image-20200416185133197.png)

**使用git log的--decorate选项查看各分支当前所指的对象**

```BASH
$ git log --oneline --decorate
f30ab (HEAD, master, testing) add feature #32 - ability to add new
34ac2 fixed bug #1328 - stack overflow under certain conditions
98ca9 initial commit of my project
```

括号里的就是分支名称，那些没有括号的就只之前的版本（旧commit）

## 切换分支

**这条命令做了两件事。一是使 HEAD 指回 master 分支，二是将工作目录恢复成 master 分支所指向的快照内容**

`git checkout <branchname>`

**使用-b选项，创建一个分支，并切换过去**

`git checkout -b iss53`

等价于

```BASH
$ git branch iss53
$ git checkout iss53
```

**获取远端的master分支，到本地作为iss53分支，执行成功就切换到iss53分支上**

`git checkout -b iss53 origin/master`

注意：

- 分支切换会改变你工作目录中的文件。因此在切换分支时，一定要注意你工作目录里的文件会被改变。如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。如果 Git 不能干净利落地完成这个任务，它将禁止切换分支（对于各自分中已提交的文件，如果更改了，就必须执行commit后才可以切换）
- 对于没有提交的内容，branch之间是共用工作目录和暂存区的

## 删除分支

删除指定分支，如果它还包含未合并的工作，则会删除失败，除非使用-D选项强制删除

`git branch -d <branchname>`

`git branch -D <branchname>`

## 分支合并

**将指定分支合并到当前分支中**

`git merge <branchname>`

**示例：合并master与hotfix分支**

![image-20200416190214330](.assets/image-20200416190214330.png)

```BASH
$ git checkout master
$ git merge hotfix
```

这次合并也叫作**快进（fast-forward）**。由于当前 master 分支所指向的提交是你当前提交（hotfix）的直接上游，所以 Git 只是简单的将指针向前移动。换句话说，当你试图合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候，只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧，这就叫做 “快进（fast-forward）”。

**合并master与iss53分支**

![image-20200416212635344](.assets/image-20200416212635344.png)

```BASH
$ git checkout master
$ git merge iss53
```

和之前将分支指针向前推进所不同的是，Git 将此次三方合并的结果做了一个新的快照并且自动创建一个新的提
交指向它。这个被称作一次合并提交，它的特别之处在于他有不止一个父提交。

![image-20200416212946552](.assets/image-20200416212946552.png)

需要指出的是，Git 会自行决定选取哪一个提交作为最优的共同祖先，并以此作为合并的基础；这和更加古老的CVS 系统或者 Subversion （1.5 版本之前）不同，在这些古老的版本管理系统中，用户需要自己选择最佳的合并基础。Git 的这个优势使其在合并操作上比其他系统要简单很多。

既然你的修改已经合并进来了，你已经不再需要 iss53 分支了。现在你可以在任务追踪系统中关闭此项任务，
并删除这个分支。

## 遇到冲突时的分支合并

有时候合并操作不会如此顺利。如果你在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改，Git 就没法干净的合并它们。如果你对 #53 问题的修改和有关 hotfix 的修改都涉及到同一个文件的同一处，在合并它们的时候就会产生合并冲突：

```BASH
$ git merge iss53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

此时 Git 做了合并，但是没有自动地创建一个新的合并提交。Git 会暂停下来，等待你去解决合并产生的冲突。你可以在合并冲突后的任意时刻使用 git status 命令来查看那些因包含合并冲突而处于未合并（unmerged）状态的文件：

```BASH
$ git status
On branch master
You have unmerged paths.
(fix conflicts and run "git commit")
Unmerged paths:
(use "git add <file>..." to mark resolution)
both modified: index.html
no changes added to commit (use "git add" and/or "git commit -a")
```

任何因包含合并冲突而有待解决的文件，都会以未合并状态标识出来。Git 会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的文件然后手动解决冲突。出现冲突的文件会包含一些特殊区段，看起来像下面这个样子：

```BASH
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

这表示 HEAD 所指示的版本（也就是你的 master 分支所在的位置，因为你在运行 merge 命令的时候已经检出到了这个分支）在这个区段的上半部分（======= 的上半部分），而 iss53 分支所指示的版本在 ======= 的下半部分。为了解决冲突，你必须选择使用由 ======= 分割的两部分中的一个，或者你也可以自行合并这些内容。例如，你可以通过把这段内容换成下面的样子来解决冲突：

```
<div id="footer">
please contact us at email.support@github.com
</div>
```

上述的冲突解决方案仅保留了其中一个分支的修改，并且 <<<<<<< , ======= , 和 >>>>>>> 这些行被完全删除了。在你解决了所有文件里的冲突之后，对每个文件使用 git add 命令来将其标记为冲突已解决。一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。

如果你对结果感到满意，并且确定之前有冲突的的文件都已经暂存了，这时你可以输入 git commit 来完成合
并提交。默认情况下提交信息看起来像下面这个样子：

```BASH
Merge branch 'iss53'
Conflicts:
index.html
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
# .git/MERGE_HEAD
# and try again.
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# All conflicts fixed but you are still merging.
#
# Changes to be committed:
# modified: index.html
#
```

## 分支管理

**查看分支列表**

分支前有*字符，表示这个分支时当前分支（HEAD指向的分支）

`git branch --list`

**查看每个分支的最后一次提交**

```BASH
[10.208.3.20 root@test-3:~/testapp]# git branch -v
  develop      a167f3b 0.2.3
* master       456bcd4 Merge branch 'master' of github.com:AllenLTB/testapp
  testchanges  cec3c1d 0.3.7
  testchanges2 2364e9d Merge branch 'master' of github.com:AllenLTB/testapp into testchanges2
```

-vv查看更多信息

```BASH
[10.208.3.20 root@test-3:~/testapp]# git branch -vv
  develop      a167f3b [testapp/develop] 0.2.3
* master       456bcd4 Merge branch 'master' of github.com:AllenLTB/testapp
  master2      456bcd4 Merge branch 'master' of github.com:AllenLTB/testapp
  master3      456bcd4 Merge branch 'master' of github.com:AllenLTB/testapp
  testchanges  cec3c1d 0.3.7
  testchanges2 2364e9d Merge branch 'master' of github.com:AllenLTB/testapp into testchanges2
```

**查看追踪的远端分支**

`git branch -r`

**查看本地和追踪的远端的分支**

`git branch -a`

**查看哪些分支已经合并到当前分支中**

`git branch --merged`

**查看哪些分支未合并到当前分支中**

`git branch --no-merged`

## 远程分支

### 简介

远程引用是对远程仓库的引用（指针），包括分支、标签等等。你可以通过`git ls-remote [remote-name] [refs]`来显式地获得远程引用的完整列表，或者通过`git remote show [remote-name]`获得远程分支的更多信息。然而，一个更常见的做法是利用远程跟踪分支。

远程跟踪分支是远程分支状态的引用。它们是你不能移动的本地引用，当你做任何网络通信操作时，它们会自动移动。远程跟踪分支像是你上次连接到远程仓库时，那些分支所处状态的书签。

它们以 (remote)/(branch) 形式命名。例如，如果你想要看你最后一次与远程仓库 origin 通信时 master分支的状态，你可以查看 origin/master 分支。你与同事合作解决一个问题并且他们推送了一个 iss53 分支，你可能有自己的本地 iss53 分支；但是在服务器上的分支会指向 origin/iss53 的提交。

这可能有一点儿难以理解，让我们来看一个例子。假设你的网络里有一个在 git.ourcompany.com的Git 服务器。如果你从这里克隆，Git 的 clone 命令会为你自动将其命名为 origin，拉取它的所有数据，创建一个指向它的 master分支的指针，并且在本地将其命名为 origin/master。Git 也会给你一个与 origin 的 master 分支在指向同一个地方的本地 master 分支，这样你就有工作的基础。

注意：远程仓库名字 “origin” 与分支名字 “master” 一样，在 Git 中并没有任何特别的含义。“master” 是当你运行 git init 时默认的起始分支名字，原因仅仅是它的广泛使用，“origin” 是当你运行 git clone 时默认的远程仓库名字。如果你运行 git clone -o booyah，那么你默认的远程分支名字将会是 booyah/master。

**在客户机上克隆远程分支时，远端仓库和本地仓库**

![image-20200416214646563](.assets/image-20200416214646563.png)

**如果你在本地的 master 分支做了一些工作，然而在同一时间，其他人推送提交到 git.ourcompany.com 并更新了它的 master 分支，那么你的提交历史将向不同的方向前进。也许，只要你不与 origin 服务器连接，你的 origin/master 指针就不会移动。**

![image-20200416214741012](.assets/image-20200416214741012.png)

**更新你的远程仓库引用**

![image-20200416214910387](.assets/image-20200416214910387.png)

**为了演示有多个远程仓库与远程分支的情况，我们假定你有另一个内部 Git 服务器，仅用于你的 sprint 小组的开发工作。这个服务器位于 git.team1.ourcompany.com。你可以运行 git remote add 命令添加一个新的远程仓库引用到当前的项目，这个命令我们会在 Git 基础 中详细说明。将这个远程仓库命名为 teamone，将其作为整个 URL 的缩写。**

![image-20200416215115593](.assets/image-20200416215115593.png)

**添加另一个远程仓库**

![image-20200416215327738](.assets/image-20200416215327738.png)

### 推送分支

**将本地的serverfix推送到origin的serverfix分支中**

两条命令一样的含义

```bash
$ git push origin serverfix
$ git push origin serverfix:serverfix`
```

如果并不想让远程仓库上的分支叫做 serverfix，可以运行`git push origin serverfix:awesomebranch`将本地的 serverfix 分支推送到远程仓库上的 awesomebranch 分支

注意：如果你正在使用 HTTPS URL 来推送，Git 服务器会询问用户名与密码。默认情况下它会在终端中提示服务器是否允许你进行推送。如果不想在每一次推送时都输入用户名与密码，你可以设置一个 “credential cache”。最简单的方式就是将其保存在内存中几分钟，可以简单地运行`git config --global credential.helper cache`来设置它。

下一次其他协作者从服务器上抓取数据时，他们会在本地生成一个远程分支 origin/serverfix，指向服务器
的 serverfix 分支的引用：

```BASH
$ git fetch origin
```

抓取到新的远程跟踪分支时，本地不会自动生成一份可编辑的副本（拷贝）。换一句话说，这种情况下，不会有一个新的 serverfix 分支,只有一个不可以修改的 origin/serverfix 指针。

可以运行 git merge origin/serverfix 将这些工作合并到当前所在的分支。如果想要在自己的serverfix分支上工作，可以将其建立在远程跟踪分支之上：

```bash
$ git checkout -b serverfix origin/serverfix
```

### 跟踪分支

从一个远程跟踪分支检出一个本地分支会自动创建一个叫做 “跟踪分支”（有时候也叫做 “上游分支”）。跟踪分支是与远程分支有直接关系的本地分支。如果在一个跟踪分支上输入 git pull，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。

当克隆一个仓库时，它通常会自动地创建一个跟踪origin/master 的 master 分支（不会自动跟踪其他分支）。然而，如果你想设置其他的跟踪分支。最简单的就是之前看到的例子，运行`git checkout -b [branch] [remotename]/[branch]`。

这是一个十分常用的操作所以 Git 提供了 --track 快捷方式：

```bash
$ git checkout --track origin/serverfix
```

如果想要将本地分支与远程分支设置为不同名字，你可以轻松地增加一个不同名字的本地分支的上一个命令：

```bash
$ git checkout -b sf origin/serverfix
```

现在，本地分支 sf 会自动从 origin/serverfix 拉取。

设置已有的本地分支跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的上游分支，你可以在任意时间
使用 -u 或 --set-upstream-to 选项运行 git branch 来显式地设置。

```bash
$ git branch -u origin/serverfix
```

如果想要查看设置的所有跟踪分支，可以使用 git branch 的 -vv 选项。这会将所有的本地分支列出来并且包
含更多的信息，如每一个分支正在跟踪哪个远程分支与本地分支是否是领先、落后或是都有。

```BASH
$ git branch -vv
iss53 7e424c3 [origin/iss53: ahead 2] forgot the brackets
master 1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this
should do it
testing 5ea463a trying something new
```

这里可以看到 iss53 分支正在跟踪 origin/iss53 并且 “ahead” 是 2，意味着本地有两个提交还没有推送到服务器上。也能看到 master 分支正在跟踪 origin/master 分支并且是最新的。接下来可以看到serverfix 分支正在跟踪 teamone 服务器上的 server-fix-good 分支并且领先 2 落后 1，意味着服务器上有一次提交还没有合并入同时本地有三次提交还没有推送。最后看到 testing 分支并没有跟踪任何远程分支。

需要重点注意的一点是这些数字的值来自于你从每个服务器上最后一次抓取的数据。这个命令并没有连接服务器，它只会告诉你关于本地缓存的服务器数据。如果想要统计最新的领先与落后数字，需要在运行此命令前抓取所有的远程仓库。可以像这样做：`git fetch --all; git branch -vv`

在检出分支的时候也可以看到这样的信息

```BASH
[10.208.3.20 root@test-3:/tmp/testapp]# git checkout master 
Switched to branch 'master'
Your branch is behind 'testapp/master' by 5 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)
```

**注意：当设置好跟踪分支后，可以通过 @{upstream} 或 @{u} 快捷方式来引用它。所以在 master分支且它正在跟踪 origin/master 时，如果愿意的话可以使用 git merge @{u} 来取代 git merge origin/master。**

### 拉取分支

git fetch命令会从服务器删抓取本地没有的数据，但是它不会修改工作目录中的内容。

**将远程仓库的所有分支拉下来**

`git fetch <remote-name>`

**将远程仓库的指定拉下来**

`git fetch <remote-name> <remotebranch>`

**将远程仓库的指定分支拉下来作为一个新分支**

`git fetch <remote-name> <remotebranch>:<refspec>`

### 删除远程分支

**方式1**

```BASH
$ git push origin --delete serverfix
To https://github.com/schacon/simplegit
- [deleted] serverfix
```

**方式2**

```BASH
[10.208.3.20 root@test-3:/tmp/testapp]# git push testapp :master2
To git@github.com:AllenLTB/testapp.git
 - [deleted]         master2
```

## 变基

> 无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。

分支如下面这样

![image-20200416223603047](.assets/image-20200416223603047.png)

现在需要将experiment和master整合。如果使用merge的话，是这样：

![image-20200416223654219](.assets/image-20200416223654219.png)

把两个分支的最新快照（C3 和 C4）以及二者最近的共同祖先（C2）进行三方合并，合并的结果是生成一个新的快照（并提交）。

变基的方法：提取在 C4 中引入的补丁和修改，然后在 C3 的基础上再应用一次。你可以使用 rebase 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。

```BASH
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

它的原理是首先找到这两个分支（即当前分支 experiment、变基操作的目标基底分支 master）的最近共同祖先 C2，然后对比当前分支相对于该祖先的历次提交，提取相应的修改（不同）并存为临时文件，然后将当前分支指向目标基底 C3, 最后以此将之前另存为临时文件的修改依序应用。（译注：写明了 commit id，以便理解，下同）

![image-20200416223908042](.assets/image-20200416223908042.png)

回到master分支，尽心一次快进合并

```BASH
$ git checkout master
$ git merge experiment
```

![image-20200416223954863](.assets/image-20200416223954863.png)

此时，C4' 指向的快照就和上面使用 merge 命令的例子中 C5 指向的快照一模一样了。这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是先后串行的一样，提交历史是一条直线没有分叉。

一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁——例如向某个别人维护的项目贡献代码时。在这种情况下，你首先在自己的分支里进行开发，当开发完成时你需要先将你的代码变基到origin/master 上，然后再向主项目提交修改。这样的话，该项目的维护者就不再需要进行整合工作，只需要快进合并便可。

**总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史，从不对已推送至别处的提交执行变基操作**

# 服务器上的Git

>  Git 可以使用四种主要的协议来传输数据：本地传输，SSH 协议，Git 协议和HTTP协议。但除了HTTP 协议之外，其他所有协议都要求在服务器端安装并运行Git 。

## 本地协议

```bash
$ git clone /opt/git/project.git
```

或者

```bash
$ git clone file:///opt/git/project.git
```

如果你在URL的开头明确的使用file:// ，那么Git 会以一种略微不同的方式运行。如果你只给出路径，Git 会尝试使用硬链接或者直接复制它需要的文件。如果使用了file:// ，Git会调用它平时通过网络来传输数据的工序，而这种方式的效率相对很低。使用file:// 前缀的主要原因是当你需要一个不包含无关引用或对象的干净仓库副本的时候（一般是从其他版本控制系统的导入之后或者类似的情形）。我们这里使用普通路径，因为通常这样总是更快。

添加一个本地仓库到现有的Git工程

```bash
$ git remote add local_proj /opt/git/project.git
```

## SSH协议

SSH 也是唯一一个同时便于读和写操作的网络协议。另外两个网络协议（HTTP 和Git）通常都是只读的，所以虽然二者对大多数人都可用，但执行写操作时还是需要SSH。SSH 同时也是一个验证授权的网络协议；而因为其普遍性，通常也很容易架设和使用。
通过SSH 克隆一个Git 仓库，你可以像下面这样给出ssh:// 的URL：

```bash
$ git clone ssh://user@server:project.git
```

或者不指明某个协议时Git 会默认使用SSH ：

```bash
$ git clone user@server:project.git
```

也可以不指明用户，Git 会默认使用你当前登录的用户。

**优点**

SSH 架设相对比较简，且通过SSH 进行访问是安全的，所有数据传输都是加密和授权的。类似Git 和本地协议，SSH 很高效，会在传输之前尽可能的压缩数据。

**缺点**

SSH 的限制在于你不能通过它实现仓库的匿名访问。即使仅为读取数据，人们也必须在能通过SSH 访问主机的前提下才能访问仓库，这使得SSH不利于开源的项目。如果你仅仅在公司网络里使用，SSH 可能是你唯一需要使用的协议。如果想允许对项目的匿名只读访问，那么除了为自己推送而架设SSH 协议之外，还需要其他协议来让别人获取数据。

## Git协议

这是一个包含在Git 软件包中的特殊守护进程； 它会监听一个提供类似于SSH 服务的特定端口（9418），而无需任何授权。用Git 协议运营仓库，你需要创建git-export-daemon-ok 文件——它是协议进程提供仓库服务的必要条件——但除此之外该服务没有什么安全措施。要么所有人都能克隆Git 仓库，要么谁也不能。这也意味着该协议通常不能用来进行推送。你可以允许推送操作；然而由于没有授权机制，一旦允许该操作，网络上任何一个知道项目URL 的人将都有推送权限。不用说，这是十分罕见的情况。

一般做法是，使用Git协议的同时提供SSH 接口，让几个开发者拥有推送（写）权限，其他人通过git:// 拥有只读权限。

**优点**

Git 协议是现存最快的传输协议。如果你的项目有很大的访问量，或者你的项目很庞大并且不需要为写进行用户授权，架设 Git 守护进程来提供服务是不错的选择。它使用与 SSH 相同的数据传输机制，但是省去了加密和授权的开销。

**缺点**

是缺少授权机制。

## HTTP/S协议

Git 通过 HTTP 通信有两种模式。在 Git 1.6.6 版本之前只有一个方式可用，十分简单并且通常是只读模式的。Git 1.6.6 版本引入了一种新的、更智能的协议，让 Git 可以像通过 SSH 那样智能的协商和传输数据。新版本的 HTTP 协议一般被称为“智能” HTTP 协议，旧版本的一般被称为“哑” HTTP 协议。

**智能HTTP协议**

运行方式和SSH及Git协议类似，只运行在标注的HTTP/S端口上，可以使用各种HTTP验证机制，比如用户名/密码方式，相比SSH可以免去设置公钥。支持匿名服务，并且推送和抓取是可以使用同一个URL。

**优点**
仅针对智能HTTP协议

不同的访问方式只需要一个 URL 以及服务器只在需要授权时提示输入授权信息。相比 SSH 协议，可以使用用户名／密码授权是一个很大的优势，这样用户就不必须在使用Git 之前先在本地生成 SSH 密钥对再把公钥上传到服务器。与 SSH 协议类似，HTTP 协议也非常快和高效。

你也可以在 HTTPS 协议上提供只读版本库的服务，如此你在传输数据的时候就可以加密数据；或者，你甚至可以让客户端使用指定的 SSL 证书。

**缺点**
架设 HTTP/S 协议的服务端会比 SSH 协议的麻烦一些。

如果你在 HTTP 上使用需授权的推送，管理凭证会比使用 SSH 密钥认证麻烦一些。然而，你可以选择使用凭证存储工具，比如 OSX 的 Keychain 或者 Windows 的凭证管理器。

## 架设Git服务器

### 使用SSH为仓库提供服务

如果需要团队里的每个人都对仓库有写权限，又不能给每个人在服务器上建立账户，那么提供 SSH 连接就是唯一的选择了。

有几个方法可以使你给团队每个成员提供访问权：

- 第一个就是给团队里的每个人创建账号，这种方法很直接但也很麻烦。你需要为每个人运行一次useradd并且设置密码。关于写权限可以通过设置facl来实现。
- 第二个办法是在主机上建立一个 git 账户，让每个需要执行git操作的人发送他的SSH公钥，然后将其加入 git 账户的~/.ssh/authorized_keys 文件。这样一来，所有人都将通过 git 账户访问主机。这一点也不会影响提交的数据（访问主机用的身份不会影响提交对象的提交者信息）。相对于第一种方法，权限更加无法控制。

如果想要防止用户使用统一的git账户或者个人独立的账户在SSH服务器上做其他于git无关的操作，可以将做Git相关操作用户的shell更改为git-shell，命令：`usermod -s /usr/bin/git-shell gituser1`。更改完shell之后，仍然可以执行git相关操作，但直接ssh登录这个用户会报错。

**实例**

开始架设Git服务器的时候，需要把一个现存的仓库导出为新的纯仓库（不包含当前工作目录的仓库）。这可以使用clone命令的--bare选项做到。纯仓库的目录名以.git 结尾：

```bash
[10.208.3.20 root@test-3:~/testapp]# mkdir /opt/gitrepo/
[10.208.3.20 root@test-3:~/testapp]# git clone --bare /tmp/testapp/ /opt/gitrepo/testapp2.git
Cloning into bare repository '/opt/gitrepo/testapp2.git'...
done.
[10.208.3.20 root@test-3:~/testapp]# ll -a /opt/gitrepo/testapp2.git
total 16
drwxr-xr-x   7 root root 138 2020-04-15 11:45:32 .
drwxrwxrwt. 12 root root 257 2020-04-15 11:46:07 ..
drwxr-xr-x   2 root root   6 2020-04-15 11:45:32 branches
-rw-r--r--   1 root root 105 2020-04-15 11:45:32 config
-rw-r--r--   1 root root  73 2020-04-15 11:45:32 description
-rw-r--r--   1 root root  23 2020-04-15 11:45:32 HEAD
drwxr-xr-x   2 root root 242 2020-04-15 11:45:32 hooks
drwxr-xr-x   2 root root  21 2020-04-15 11:45:32 info
drwxr-xr-x  22 root root 210 2020-04-15 11:45:32 objects
-rw-r--r--   1 root root 282 2020-04-15 11:45:32 packed-refs
drwxr-xr-x   4 root root  31 2020-04-15 11:45:32 refs
```

这个命令其实就等于

```bash
$ cp -Rf /tmp/testapp/.git /opt/gitrepo/testapp2.git
```

将如果Git服务器不是当前服务器，则执行下面的命令将目录拷贝到到指定的服务器上

```bash
$ scp -r /opt/gitrepo/testapp2.git root@x.x.x.x:/opt/gitrepo/testapp2.git
```

仓库服务器是10.208.3.20，其上有两个用户gituser1和gituser2，现在10.208.3.21上使用这两个用户克隆仓库

```bash
[10.208.3.21 root@test-4:~]# git clone gituser1@10.208.3.20:/opt/gitrepo/testapp2.git
Cloning into 'testapp2'...
gittest1@10.208.3.20's password: 
remote: Counting objects: 57, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 57 (delta 23), reused 38 (delta 12)
Receiving objects: 100% (57/57), 141.96 KiB | 0 bytes/s, done.
Resolving deltas: 100% (23/23), done.
[10.208.3.21 root@test-4:~]# ll -a testapp2/
total 692
drwxr-xr-x   4 root root    135 2020-04-15 11:52:04 .
dr-xr-x---. 24 root root   4096 2020-04-15 11:51:59 ..
drwxr-xr-x   8 root root    163 2020-04-15 11:52:04 .git
drwxr-xr-x   3 root root     15 2020-04-15 11:52:04 hellow
-rw-r--r--   1 root root    206 2020-04-15 11:52:04 hosts
-rw-r--r--   1 root root     23 2020-04-15 11:52:04 issue
-rw-r--r--   1 root root  11357 2020-04-15 11:52:04 LICENSE
-rw-r--r--   1 root root   1330 2020-04-15 11:52:04 passwd
-rw-r--r--   1 root root   1338 2020-04-15 11:52:04 passwd2
-rw-r--r--   1 root root     27 2020-04-15 11:52:04 README.md
-rw-r--r--   1 root root 670293 2020-04-15 11:52:04 services
```

对10.208.3.20具有SSH访问权限，且对/tmp/目录有写入权限的用户可以用以下命令进行推送

```bash
[10.208.3.21 root@test-4:~]# cd testapp2/
[10.208.3.21 root@test-4:~/testapp2]# echo issueissueissue >> issue
[10.208.3.21 root@test-4:~/testapp2]# git add .
[10.208.3.21 root@test-4:~/testapp2]# git config user.email "gituser1@jzbgltb.com"
[10.208.3.21 root@test-4:~/testapp2]# git config user.name "gituser1"
[10.208.3.21 root@test-4:~/testapp2]# git commit -m "0.1.11"
[10.208.3.21 root@test-4:~/testapp2]# git push origin master
gituser1@10.208.3.20's password: 
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 279 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To root@10.208.3.20:/tmp/testapp2.git
   5e8cf89..422a07b  master -> master
```

执行push操作时，如果使用的用户不具有写入权限，则会报错

```bash
[10.208.3.21 root@test-4:~/testapp2]# git push origin master
gituser1@10.208.3.20's password: 
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 280 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: error: insufficient permission for adding an object to repository database ./objects
remote: fatal: failed to write object
error: unpack failed: unpack-objects abnormal exit
To gituser1@10.208.3.20:/opt/gitrepo/testapp2.git
 ! [remote rejected] master -> master (unpacker error)
error: failed to push some refs to 'gituser1@10.208.3.20:/opt/gitrepo/testapp2.git'
```

### SSH+权限管理器Gitosis

>  把所有用户的公钥保存在统一用户的authorized_keys文件的做法只能暂时奏效。当用户数量到了几百人的时候，它会变成一种痛苦。每一次都必须进入服务器的shell，而且缺少对连接的限制（文件里的每个人都对所有项目拥有读写权限）
>
>  现在，是时候向广泛使用的软件Gitosis求救了。Gitosis 简单的说就是一套用来管理authorized_keys文件和实现简单连接限制的脚本。最有意思的是，该软件用来添加用户和设定权限的界面不是网页，而是一个特殊的Git仓库。推送设定好的某个项目后，Gitosis 就会随之改变服务器设定。

#### 部署Gitosis的步骤

**安装Gitosis需要使用的Python工具**

```bash
[10.208.3.20 root@test-3:/opt/gitrepo]# yum install -y python-setuptools
```

**克隆并安装Gitosis**

```bash
[10.208.3.20 root@test-3:/]# git clone https://github.com/res0nat0r/gitosis.git
[10.208.3.20 root@test-3:/]# cd /gitosis/
[10.208.3.20 root@test-3:/gitosis]# sudo python setup.py install
```

**为Gitosis创建一个用户**

因为之前已经创建过一个gituser1用户了，现在用这个用户就行。只不过需要将这个用户的shell改为bash

```bash
[10.208.3.20 root@test-3:/gitosis]#  usermod -s /bin/bash gituser1
###Gitosis将会为我们管理公钥，所以需要将之前添加的authorized_keys文件删除
[10.208.3.20 root@test-3:/gitosis]# rm -rf /root/.ssh/authorized_keys
```

**初始化Gitosis**

使用上面创建的gituser1用户，并通过管理员的公钥来运行gitosis-init。Gitosis会将它的仓库放到gituser1家目录下的repositories目录中

```bash
[10.208.3.20 root@test-3:/gitosis]# sudo -H -u gituser1 gitosis-init < /tmp/id_rsa.pub
Initialized empty Git repository in /home/gituser1/repositories/gitosis-admin.git/
Reinitialized existing Git repository in /home/gituser1/repositories/gitosis-admin.git/
```

只用该公钥的拥有者才能修改gitosis-admin仓库。

**将gitosis-admin仓库中的post-update加上执行权限**

```bash
[10.208.3.20 root@test-3:/gitosis]# chmod 755 /home/gituser1/repositories/gitosis-admin.git/hooks/post-update
```

**拥有管理员公钥的用户来登录SSH服务器，结果如下**

```bash
[10.208.3.21 root@test-4:/gitosis]# ssh gituser1@10.208.3.20
PTY allocation request failed on channel 0
ERROR:gitosis.serve.main:Need SSH_ORIGINAL_COMMAND in environment.
Connection to 10.208.3.20 closed.
```

因为没有运行任何Git命令所以它切断了连接。

**克隆Gitosis仓库**

因为是使用管理员公钥，所以可以克隆成功。

```bash
[10.208.3.21 root@test-4:/tmp]# git clone gituser1@10.208.3.20:gitosis-admin.git
Cloning into 'gitosis-admin'...
remote: Counting objects: 37, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 37 (delta 8), reused 4 (delta 0)
Receiving objects: 100% (37/37), done.
Resolving deltas: 100% (8/8), done.
```

克隆得到了一个名为gitosis-admin的目录，内容主要分为两部分（其实还有.git隐藏目录，这个目录也不需要我们改动，我下面没有展示出来）

```bash
[10.208.3.21 root@test-4:/tmp/gitosis-admin]# tree -F ./
./
├── gitosis.conf
└── keydir/
    └── test4.pub
```

gitosis.conf文件是用来设置用户、仓库和权限的控制文件。keydir目录则是保存所有具有访问权限的用户公钥的地方（每个人一个公钥）。对于初始化得来的这个test4.pub文件名，是Gitosis使用gitosis-init脚本导入的公钥尾部的描述中获取的，文件内容就是管理员公钥。

gitosis.conf文件的内容如下，现在只包含了gitosis-admin仓库相关的信息

```bash
[10.208.3.21 root@test-4:/tmp/gitosis-admin]# cat gitosis.conf 
[gitosis]
[group gitosis-admin]
members = test4
writable = gitosis-admin
```

它显示用户test4（初始化Gitosis 公钥的拥有者）是唯一能访问gitosis-admin项目的人。

#### 示例：新建项目，配置Gitosis管理用户权限

现在我们添加一个新的项目testapp。项目中有两个开发者共同工作，一个人只能获取，另一个既可以获取，也可以写入。

项目的仓库目录已经存放在/home/gituser1/repositories/下面了，名为testapp2.git。

**添加用户公钥**

要和他人共同工作在一个项目上，就需要添加他们的公钥文件到keydir目录中。文件的命名决定gitosis.conf文件中用户的称呼。现在先添加test5的公钥

```bash
[10.208.3.21 root@test-4:/tmp/gitosis-admin]#  cp /tmp/test5.pub /tmp/gitosis-admin/keydir/
```

此时我们有两个用户了，分别是test4和test5.

**按需更改gitosis.conf文件**

按照团队授权项目的权限，并将用户加入到指定group。在gitosis.conf文件中新增下面这些内容：

```bash
[group testapp2]
writable = testapp2
members = test5		##如果要同时指定多个用户，就用空格分隔

[group testapp2_ro]
readonly = testapp2
members = test4
```

上面这表示，创建了testapp2和testapp2_ro两个组，前者有testapp2的读取和写入权限（有写入就表示有读取了）。后者有testapp2的读取权限。然后将test5和test4分别加入到这两个组中。

**将修改后的仓库，PUSH到服务器**

```bash
[10.208.3.21 root@test-4:/tmp/gitosis-admin]# git commit -am "test2"
[10.208.3.21 root@test-4:/tmp/gitosis-admin]# git push origin master
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 321 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To gituser1@10.208.3.20:gitosis-admin.git
   171ba7a..eaf008a  master -> master
```

**最终测试**

对于testapp2项目，test5用户可以clone和push

```bash
[10.208.3.23 root@test-5:~]# git clone gituser1@10.208.3.20:testapp2.git
Cloning into 'testapp2'...
/etc/profile.d/system_general.sh: line 7: ip: command not found
remote: Counting objects: 57, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 57 (delta 23), reused 38 (delta 12)
Receiving objects: 100% (57/57), 141.96 KiB | 0 bytes/s, done.
Resolving deltas: 100% (23/23), done.
[10.208.3.23 root@test-5:~]# cd testapp2
[10.208.3.23 root@test-5:~/testapp2]# echo 123123123123 >> issue 
[10.208.3.23 root@test-5:~/testapp2]# git commit -am "test1"
[10.208.3.23 root@test-5:~/testapp2]# git config user.name "test5"
[10.208.3.23 root@test-5:~/testapp2]# git config user.email "test5@jzbgltb.com"
[10.208.3.23 root@test-5:~/testapp2]# git push origin master
/etc/profile.d/system_general.sh: line 7: ip: command not found
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 279 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To gituser1@10.208.3.20:testapp2.git
   5e8cf89..e7dd6c3  master -> master
```

对于testapp2项目，test4用户可以clone，但无法push

```bash
[10.208.3.21 root@test-4:/tmp]# git clone gituser1@10.208.3.20:testapp2.git
Cloning into 'testapp2'...
remote: Counting objects: 60, done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 60 (delta 23), reused 37 (delta 12)
Receiving objects: 100% (60/60), 142.23 KiB | 0 bytes/s, done.
Resolving deltas: 100% (23/23), done.
[10.208.3.21 root@test-4:/tmp]# cd testapp2
[10.208.3.21 root@test-4:/tmp/testapp2]# echo 123123123 >> issue
[10.208.3.21 root@test-4:/tmp/testapp2]# git config user.name "test4"
[10.208.3.21 root@test-4:/tmp/testapp2]# git config user.email "test4@jzbgltb.com"
[10.208.3.21 root@test-4:/tmp/testapp2]# git commit -am "test3"
[master 055d59d] test2
 1 file changed, 1 insertion(+)
[10.208.3.21 root@test-4:/tmp/testapp2]# git push origin master
/etc/profile.d/system_general.sh: line 7: ip: command not found
ERROR:gitosis.serve.main:Repository write access denied
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```

#### FQA

**`fatal: 'repositories/testapp2.git' does not appear to be a git repository`报错**

可能是下面这些原因导致的：

- gitosis.conf中写的用户名和keydir里面的key的名字没有完全对应上。
- gitosis.conf有的地方写错了，比如把members写成了member。
- gitosis的仓库目录下，不存在你clone的那个项目（按照上面这个报错，就是说/home/gituser1/repositories目录下，不存在testapp2.git这个仓库）。可以将仓库mv/cp过来，也可以ln创建软链接解决。

```bash
[10.208.3.23 root@test-5:~]# git clone gituser1@10.208.3.20:testapp2.git
Cloning into 'testapp2'...
fatal: 'repositories/testapp2.git' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

**如果因为管理员将错误的gitosis-admin仓库push到ssh服务器，导致管理员也无法管理gitosis-admin仓库了（无法clone和push），怎么办？**

假设是因为gitosis.conf中，关于gitosis-admin项目的member是写错了，那么可以直接在服务器上更改这个文件(在这个实例中是/home/gituser1/repositories/gitosis-admin.git/gitosis.conf文件)。然后在让管理员重新试试clone和push。

我之前将将管理员的公钥文件改名了，但是忘记更改gitosis.conf文件了，push之后导致管理员无法对gitosis-admin仓库进行clone和push。我最终的解决方法是直接更改服务器上gitosis-admin项目的gitosis.conf文件。然后管理员就可以clone和push了。之后在本地修正了问题，并且重新push（必须在修正后重新push，因为直接修改服务器上的文件只是临时有效）。

其实，如果有上次的仓库备份（比如是cp了一份），那么直接替换仓库即可。

### SSH的Git仓库的默认权限问题解决

多人共同开发维护一个项目时，对整个项目文件互有拉取、推送等行为。为防止操作时文件权限出现冲突，可有以下2种方法解决：

1. 本地git的远端设置中，连接远程仓库时多人使用同一个用户名，该用户名为git远程仓库的拥有者
2. 不同开发者需使用不同的用户名连接时，会出现文件权限冲突，这是因为 git 仓库使用的是对象存储，每次改动会新增若干对象文件（具体对象文件在 /.git/objects 下），而新增的对象文件权限属性由系统控制，默认为755，即非文件拥有着无法进行写入（推送）。这里需要将 git 仓库下文件的默认权限设置为同组用户均可读写执行。

注意：不管使用哪种方法，都建议同时设置所有用户的shell为git-shell（`uermod -s /usr/bin/git-shell xxx`），以提升安全性。

在初始化仓库时的语法：

```
git init --bare --shared[=(false|true|umask|group|all|world|everybody|0xxx)]
```

如果仓库已经启用，在远程仓库目录下更改 git 配置的语法：

```
git config core.sharedRepository [(false|true|umask|group|all|world|everybody|0xxx)]
```

**实践：初始化仓库时**

```BASH
[10.208.3.20 root@test-3:/opt/gitrepo]# sudo -g git -u gituser1 mkdir testapp3/
[10.208.3.20 root@test-3:/opt/gitrepo]# ll testapp3/ -d
drwxrwsr-x 7 gituser1 git 119 2020-04-18 10:24:54 testapp3/
[10.208.3.20 root@test-3:/opt/gitrepo/testapp3]# sudo -g git -u gituser1 git init --bare --shared
Initialized empty shared Git repository in /opt/gitrepo/testapp3/
[10.208.3.20 root@test-3:/opt/gitrepo/testapp3]# ll
total 12
drwxrwsr-x 2 gituser1 git   6 2020-04-18 10:24:54 branches
-rw-rw-r-- 1 gituser1 git 126 2020-04-18 10:24:54 config
-rw-rw-r-- 1 gituser1 git  73 2020-04-18 10:24:54 description
-rw-rw-r-- 1 gituser1 git  23 2020-04-18 10:24:54 HEAD
drwxrwsr-x 2 gituser1 git 242 2020-04-18 10:24:54 hooks
drwxrwsr-x 2 gituser1 git  21 2020-04-18 10:24:54 info
drwxrwsr-x 4 gituser1 git  30 2020-04-18 10:24:54 objects
drwxrwsr-x 4 gituser1 git  31 2020-04-18 10:24:54 refs
[10.208.3.20 root@test-3:/opt/gitrepo/testapp3]# ll objects/
total 0
drwxrwsr-x 2 gituser1 git 6 2020-04-18 10:24:54 info
drwxrwsr-x 2 gituser1 git 6 2020-04-18 10:24:54 pack
[10.208.3.20 root@test-3:/opt/gitrepo/testapp3]# git config -l
user.email=tianbao1@example.com
core.repositoryformatversion=0
core.filemode=true
core.bare=true
core.sharedrepository=1
receive.denynonfastforwards=true
```

**实践：已启用后配置**

> 需要确保所有用户的组都一样（附加组一样不行，必须是组一样，因为附加组复发是用SGID）

1. 设置所有已存在文件的权限（包括仓库文件本身）
   - 所有文件与目录权限设置为770
   - 所有目录开启`SGID`
2. 配置该仓库下所有新生成的对象文件的默认权限为770，且目录有SGID权限），下面命令二选一
   - `git config core.sharedRepository 0770`
   - `git config core.sharedRepository 1`

```BASH
[10.208.3.20 root@test-3:/opt/gitrepo]# chmod -R 770 testapp2.git/
[10.208.3.20 root@test-3:/opt/gitrepo]# chmod -R g+s testapp2.git/
[10.208.3.20 root@test-3:/opt/gitrepo]# chown -R :git testapp2.git/
[10.208.3.20 root@test-3:/opt/gitrepo]# cd testapp2.git/
[10.208.3.20 root@test-3:/opt/gitrepo/testapp2.git]# git config core.sharedRepository 0770
[10.208.3.20 root@test-3:/opt/gitrepo/testapp2.git]# git config -l
user.email=tianbao1@example.com
core.repositoryformatversion=0
core.filemode=true
core.bare=true
core.sharedrepository=0770
remote.origin.url=/tmp/testapp/
[10.208.3.20 root@test-3:/opt/gitrepo]# ll -d testapp2.git
drwxrws--- 7 root git 138 2020-04-18 10:52:28 testapp2.git
[10.208.3.20 root@test-3:/opt/gitrepo]# ll testapp2.git/
total 16
drwxrws---  2 root git   6 2020-04-17 16:11:29 branches
-rw-rw-r--  1 root git 130 2020-04-18 10:52:28 config
-rwxrwx---  1 root git  73 2020-04-17 16:11:29 description
-rwxrwx---  1 root git  23 2020-04-17 16:11:29 HEAD
drwxrws---  2 root git 242 2020-04-17 16:11:29 hooks
drwxrws---  2 root git  21 2020-04-17 16:11:29 info
drwxrws--- 32 root git 310 2020-04-18 10:43:21 objects
-rwxrwx---  1 root git 282 2020-04-17 16:11:29 packed-refs
drwxrws---  4 root git  31 2020-04-17 16:11:29 refs
[10.208.3.20 root@test-3:/opt/gitrepo/testapp2.git]# ll objects/
total 0
drwxrws--- 2 root     git  52 2020-04-18 10:13:46 0c
drwxrws--- 2 root     git  52 2020-04-17 16:11:29 16
drwxrws--- 2 root     git  52 2020-04-17 16:11:29 19
drwxrws--- 2 gituser1 git  52 2020-04-18 10:54:10 1b
drwxrws--- 2 root     git  52 2020-04-18 10:07:41 23
drwxrws--- 2 gituser1 git  52 2020-04-18 10:19:42 25
drwxrws--- 2 root     git  52 2020-04-18 10:55:07 27
......
```

### Smart HTTP

设置 Smart HTTP 一般只需要在服务器上启用一个 Git 自带的名为 git-http-backend 的 CGI 脚本。该CGI 脚本将会读取由 git fetch 或 git push 命令向 HTTP URL 发送的请求路径和头部信息，来判断该客户端是否支持 HTTP 通信（不低于 1.6.6 版本的客户端支持此特性）。如果 CGI 发现该客户端支持智能（Smart）模式，它将会以智能模式与它进行通信，否则它将会回落到哑（Dumb）模式下（因此它可以对某些老的客户端实现向下兼容）。

**安装apache，使用apache作为CGI服务器**

```bash
[10.208.3.20 root@test-3:~]# yum install httpd
```

向apache配置文件添加下面这些内容，让git-http-backend作为web服务器对访问/gitrepo路径的请求做处理

```BASH
<VirtualHost *:80>
    ServerName gitrepo2.leju.com
    SetEnv GIT_HTTP_EXPORT_ALL
    SetEnv GIT_PROJECT_ROOT /opt/gitrepo
    ScriptAlias /gitrepo/ /usr/libexec/git-core/git-http-backend/
    <Directory "/usr/libexec/git-core/">
    Options ExecCGI Indexes
    Require all granted
    </Directory>
</VirtualHost>
```

如果留空 GIT_HTTP_EXPORT_ALL 这个环境变量，Git 将只对无授权客户端提供带 git-daemon-export-ok文件的版本库，就像 Git 守护进程一样。

**想实现写操作授权验证，添加下面这些配置即可**

```BASH
<LocationMatch "^/gitrepo/.*/git-receive-pack$">
AuthType Basic
AuthName "Git Access"
AuthUserFile /opt/git/.htpasswd
Require valid-user
</LocationMatch>
```

这需要你创建一个包含所有合法用户密码的 .htaccess 文件。以下是一个添加 “schacon” 用户到此文件的例子：

```BASH
$ htdigest -c /opt/git/.htpasswd "Git Access" schacon
```

但是我克隆的时候，出现了下面这个问题，testapp2目录下之后一个.git目录，没有其他文件

```BASH
[10.208.3.21 root@test-4:/tmp]# git clone http://gitrepo2.leju.com/gitrepo/testapp2.git
Cloning into 'testapp2'...
remote: Counting objects: 57, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 57 (delta 23), reused 38 (delta 12)
Unpacking objects: 100% (57/57), done.
warning: remote HEAD refers to nonexistent ref, unable to checkout.
```

原因是.git目录下.git/refs/heads不存在HEAD指向的文件，这个时候可以用git show-ref命令查看。

后来我重新生成了一下服务器端的仓库，就没这个问题了。

**至于说push，我一直没有实现出来，不管是否启用认证，总是报错403**

```BASH
[10.208.3.21 root@test-4:/tmp/testapp2]# git push http://litianbao:123123@gitrepo2.leju.com/gitrepo/testapp2.git master
fatal: unable to access 'http://litianbao:123123@gitrepo2.leju.com/gitrepo/testapp2.git/': The requested URL returned error: 403
```

# 分布式

## 分布式协作的方式

**集中式工作流**

![image-20200418154245044](.assets/image-20200418154245044.png)

如果有两个开发者从中心仓库克隆代码下来，同时作了一些修改，那么只有第一个开发者可以顺利地把数据推送回共享服务器。第二个开发者在推送修改之前，必须先将第一个人的工作合并进来，这样才不会覆盖第一个人的修改。

**集成管理者工作流**

![image-20200418154312791](.assets/image-20200418154312791.png)

这是 GitHub 和 GitLab 等集线器式（hub-based）工具最常用的工作流程。这么做最主要的优点之一是你可以持续地工作，而主仓库的维护者可以随时拉取你的修改。贡献者不必等待维护者处理完提交的更新（每一方都可以按照自己节奏工作）。

**司令官与副官工作流**

![image-20200418154340108](.assets/image-20200418154340108.png)

这种工作流程并不常用，只有当项目极为庞杂，或者需要多级别管理时，才会体现出优势。利用这种方式，项目总负责人（即司令官）可以把大量分散的集成工作委托给不同的小组负责人分别处理，然后在不同时刻将大块的代码子集统筹起来，用于之后的整合。

## 维护项目

### 应用来自邮件的补丁

如果你通过电子邮件收到了一个需要整合进入项目的补丁，你需要将其应用到特性分支中进行评估。有两种应用该种补丁的方法：使用 git apply，或者使用 git am。

**涉及到的命令**

- 生成可以邮寄的mbox格式的补丁文件。format-patch将每一个提交转换为一封电子邮件，提交信息的第一行作为主题，剩余信息与提交引入的补丁作为正文。它有一个好处是是使用 format-patch 生成的一封电子邮件应用的提交正确地保留了所有的提交信息。
  - `git format-patch -M origin/master`

- git apply 是一个事务性操作的命令，也就是说，要么所有补丁都打上去，要么全部放弃。所以不会出现patch命令那样，一部分文件打上了补丁而另一部分却没有，这样一种不上不下的修订状态。
  - `git apply xxx.patch`

- 在实际打补丁之前，可以先用git apply --check 查看补丁是否能够干净顺利地应用到当前分支中，如果没有任何输出，表示我们可以顺利采纳该补丁。：
  - `git apply --check xxx.patch`

- 使用am应用补丁（效果和apply一样，但是git am会自动commit，而且git am正是为了读取mbox文件而构建的）
  - `git am XXX`

- 如果想通过复制文件到邮件客户端，也可以通过命令行程序（git imap-send，git send-email）发送，此处不介绍这方面内容。

**实践**

生成补丁文件

```BASH
[10.208.3.23 root@test-5:/]# git clone gituser1@10.208.3.20:testapp2.git
Cloning into 'testapp2'...
remote: Counting objects: 75, done.
remote: Compressing objects: 100% (52/52), done.
remote: Total 75 (delta 30), reused 38 (delta 12)
Receiving objects: 100% (75/75), 143.39 KiB | 0 bytes/s, done.
Resolving deltas: 100% (30/30), done.
[10.208.3.23 root@test-5:/]# cd testapp2/
[10.208.3.23 root@test-5:/testapp2]# for i in {1..5} ; do echo '11111111111' >> issue ; done
[10.208.3.23 root@test-5:/testapp2]# git add  .
[10.208.3.23 root@test-5:/testapp2]# git commit -am "test1"
[10.208.3.23 root@test-5:/testapp2]# git config user.name "gituser1"
[10.208.3.23 root@test-5:/testapp2]# git config user.email "gituser1@jzbgltb.com"
[10.208.3.23 root@test-5:/testapp2]# for i in {1..5} ; do echo '22222222222' >> issue ; done
[10.208.3.23 root@test-5:/testapp2]# git commit -am "test2"
[master ac41fb3] test2
 1 file changed, 10 insertions(+)
[10.208.3.23 root@test-5:/testapp2]# git format-patch -M origin/master
0001-test2.patch
###如果在 --- 行与补丁开头（diff --git 行）之间添加文本，那么开发者就可以阅读它；但是应用补丁时会排除它。
[10.208.3.23 root@test-5:/testapp2]# cat 0001-test2.patch 
From ac41fb31b88e47affccd0e552a434ab6b2556125 Mon Sep 17 00:00:00 2001
From: gituser1 <gituser1@jzbgltb.com>
Date: Sat, 18 Apr 2020 17:00:05 +0800
Subject: [PATCH] test2

---
 issue | 10 ++++++++++
 1 file changed, 10 insertions(+)

diff --git a/issue b/issue
index 974542a..9af09ee 100644
--- a/issue
+++ b/issue
@@ -17,3 +17,13 @@ Kernel \r on an \m
 123123123123
 123123123123
 123123123123
+11111111111
+11111111111
+11111111111
+11111111111
+11111111111
+22222222222
+22222222222
+22222222222
+22222222222
+22222222222
-- 
1.8.3.1
```

应用补丁文件

```BASH
[10.208.3.23 root@test-5:/]# git clone gituser1@10.208.3.20:testapp2.git testapp3
Cloning into 'testapp3'...

/etc/profile.d/system_general.sh: line 7: ip: command not found
remote: Counting objects: 75, done.
remote: Compressing objects: 100% (52/52), done.
remote: Total 75 (delta 30), reused 38 (delta 12)
Receiving objects: 100% (75/75), 143.39 KiB | 0 bytes/s, done.
Resolving deltas: 100% (30/30), done.
[10.208.3.23 root@test-5:/]# cd testapp3/
[10.208.3.23 root@test-5:/testapp3]# git status
# On branch master
nothing to commit, working directory clean
[10.208.3.23 root@test-5:/testapp3]# git apply --check ../testapp2/0001-test2.patch
[10.208.3.23 root@test-5:/testapp3]# git apply ../testapp2/0001-test2.patch
[10.208.3.23 root@test-5:/testapp3]# git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   issue
#
no changes added to commit (use "git add" and/or "git commit -a")
```

### 检出远程分支

如果你的贡献者建立了自己的版本库，并且向其中推送了若干修改，之后将版本库的 URL 和包含更改的远程分
支发送给你，那么你可以将其添加为一个远程分支，并且在本地进行合并。

比如 Jessica 向你发送了一封电子邮件，内容是在她的版本库中的 ruby-client 分支中有一个很不错的新功能，为了测试该功能，你可以将其添加为一个远程分支，并在本地检出：

```bash
$ git remote add jessica git://github.com/jessica/myproject.git
$ git fetch jessica
$ git checkout -b rubyclient jessica/ruby-client
```

如果她再次发邮件说另一个分支中包含另一个优秀功能，因为之前已经设置好远程分支了，你就可以直接进行抓取及检出操作。这对于与他人长期合作工作来说很有用。

对于非持续性的合作，如果你依然想要以这种方式拉取数据的话，你可以对远程版本库的 URL 调用 git pull命令。这会执行一个一次性的抓取，而不会将该 URL 存为远程引用：

```BASH
$ git pull https://github.com/onetimeguy/project
From https://github.com/onetimeguy/project
 * branch HEAD -> FETCH_HEAD
Merge made by recursive.
```

### 确定引入了哪些东西

**对contrib分支中所有master 分支尚未包含的提交进行检查**

```BASH
$ git log contrib --not master
commit 5b6235bd297351589efc4d73316f0a68d484f118
Author: Scott Chacon <schacon@gmail.com>
Date: Fri Oct 24 09:53:59 2008 -0700

    seeing if this helps the gem

commit 7482e0d16d04bea79d0dba8988cc78df655f16a0
Author: Scott Chacon <schacon@gmail.com>
Date: Mon Oct 22 19:38:36 2008 -0700

    updated the gemspec to hopefully work better
```

**查看特定commit提交改动了什么**

```BASH
[10.208.3.20 root@test-3:~/testapp]# git show 4a2c1f59c8d4cc7c14c250974c1431015dad1965
commit 4a2c1f59c8d4cc7c14c250974c1431015dad1965
Author: tianbao3 <tianbao1@example.com>
Date:   Thu Apr 16 11:45:45 2020 +0800

    0.3.4

diff --git a/issue2 b/issue2
index 6736cb1..9daeafb 100644
--- a/issue2
+++ b/issue2
@@ -1,3 +1 @@
-\S
-Kernel \r on an \m
-
+test
```

**` git log  -p`查看每次提交所引入的具体修改**

```BASH
commit 039a485c74f9fc795a0778bb96d00d6406d07232
Author: gituser1 <gituser1@jzbgltb.com>
Date:   Sat Apr 18 17:00:05 2020 +0800

    test2

diff --git a/issue b/issue
index 974542a..9af09ee 100644
--- a/issue
+++ b/issue
@@ -17,3 +17,13 @@ Kernel \r on an \m
......
+22222222222

commit 27208d4705a2b33d017dc422eef3b88b11eadd3b
Author: gituser1 <gituser1@jzbgltb.com>
Date:   Sat Apr 18 10:55:00 2020 +0800

    test1

diff --git a/hellow/1/3/le b/hellow/1/3/le
deleted file mode 100644
index 190a180..0000000
--- a/hellow/1/3/le
......
```

**查看与指定分支合并时，当前分支需要导入内容**

方法1：手工的方式找出公共祖先，并对其显式运行 diff 命令：

```bash
$ git merge-base contrib master
36c7dba2c95e6bbb78dfa822519ecfec6e1ca649
$ git diff 36c7db
```

方法2：把 ... 置于另一个分支名后来对该分支的最新提交与两个分支的共同祖先进行比较

```BASH
$ git diff master...contrib
```

### 生成一个构建号

Git 中不存在随每次提交递增的“v123”之类的数字序列，如果你想要为提交附上一个可读的名称，可以对其运
行 git describe 命令。Git 将会给出一个字符串，它由最近的标签名（带注释的标签`git tag -a xxx`）必须是、自该标签之后的提交数目和你所描述
的提交的部分 SHA-1 值构成：

```BASH
#v1.6.2-rc1：master分支的上一个tag
#20：自这个标签之后，master分支又提交了20次
#g：一个普通的字符
#8c5b85c：master分支当前commit的对象
$ git describe master
v1.6.2-rc1-20-g8c5b85c
```

这样你在导出一个快照或构建时，可以给出一个便于人们理解的命名。实际上，如果你的 Git 是从 Git 自己的版本库克隆下来并构建的，那么 git --version 命令给出的结果是与此类似的。如果你所描述的提交自身就有一个标签，那么它将只会输出标签名，没有后面两项信息。

注意 git describe 命令只适用于有注解的标签（即使用 -a 或 -s 选项创建的标签），所以如果你在使用 git describe 命令的话，为了确保能为标签生成合适的名称，打发布标签时都应该采用加注解的方式。你也可以使用这个字符串来调用 checkout 或 show 命令，但是这依赖于其末尾的简短 SHA-1 值，因此不一定一直有效。比如，最近 Linux 内核为了保证 SHA-1 值对象的唯一性，将其位数由 8 位扩展到了 10 位，导致以前的 git describe输出全部失效。

### 准备一次发布

现在你可以发布一个构建了。其中一件事情就是为那些不使用 Git 的可怜包们创建一个最新的快照归档。使用git archive 命令完成此工作：

```BASH
$ git archive master --prefix='project/' | gzip > `git describe master`.tar.gz
$ ls *.tar.gz
v1.6.2-rc1-20-g8c5b85c.tar.gz
```

如果有人将这个压缩包解压，他就可以得到你的项目文件夹的最新快照。你也可以以类似的方式创建一个zip压缩包，但此时你应该向 git archive 命令传递 --format=zip 选项：

```BASH
$ git archive master --prefix='project/' --format=zip > `git describe master`.zip
```

现在你有了本次发布的一个 tar 包和一个 zip 包，可以将其上传到网站或以电子邮件的形式发送给人们。

# Git工具

## 提交的引用方式

### 祖先引用

**^查看父提交**

查看当前提交的父提交

git show HEAD^

**^2查看第二父提交**

这种语法只在合并提交时有用，因为合并提交可能有多个父提交。第一父提交是你合并时所
在分支，而第二父提交是你所合并的分支：

**另外一个指明祖先提交的方法是~**

这也是指向第一父提交，所以HEAD~ 和HEADˆ 是等价的。

当你指定数字的时候就明显不一样了。HEAD~2 是指“第一父提交的第一父提交”，也就是“祖父提交”。

HEAD~3也可以写成HEADˆˆˆ，同样都表示第一父提交的第一父提交的第一父提交：

**混合使用**

HEAD~3^2：第一父提交的第一父提交的第一父提交的第二父提交

![image-20200419011623548](.assets/image-20200419011623548.png)

### 提交范围

**双点**

最常用的指明范围的方法是双点的语法。这种语法主要是让Git 区分出可从一个分支中获得而不能从另一个分支中获得的提交。

![image-20200416121312193](.assets/image-20200416121312193.png)

查看试验分支上哪些没有被提交到主分支（查看experiment上有，但master上没有的提交）

```bash
$ git log HEAD..experiment
```

git log ..experiment和上面的命令是一个效果，git会使用HEAD代替不存在的一边。

**多点**

你也许会想针对两个以上的分支来指明修订版本，比如查看哪些提交被包含在某些分支中的一个，但是不在你当前的分支上。Git允许你在引用前使用ˆ字符或者--not指明你不希望提交被包含其中的分支。因此下面三个命令是等同的：

```bash
$ git log refA..refB
$ git log ^refA refB
$ git log refB --not refA
```

这样很好，因为它允许你在查询中指定多于两个的引用，而这是双点语法所做不到的。如果你想查找所有从refA或refB包含的但是不被refC包含的提交，你可以输入下面中的一个

```BASH
$ git log refA refB ^refC
$ git log refA refB --not refC
```

**三点**

这个可以指定被两个引用中的一个包含但又不被两者同时包含的分支。

如果你想查看master或者experiment中包含的但不是两者共有的引用，你可以运行

```BASH
$ git log master...experiment
F
E
D
C
```

这个再次给出你普通的log输出但是只显示那四次提交的信息，按照传统的提交日期排列。
这种情形下，log命令的一个常用参数是--left-right，它会显示每个提交到底处于哪一侧的分支。这使得数据更加有用。

```bash
$ git log --left-right master...experiment
< F
< E
> D
> C
```

## 储藏（Stashing）与清理

经常有这样的事情发生，当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作，否则以后你无法回到这个工作点。解决这个问题的办法就是git stash命令。“‘储藏”“可以获取你工作目录的中间状态（也就是你修改过的被追踪的文件和暂存的变更），并将它保存到一个未完结变更的堆栈中，随时可以重新应用。

注意：

- 如果在执行恢复的时候，不指定要恢复的储藏的名字，则默认使用最近的一个储藏。
- 每个储藏都有一个名字，名字里有索引（stash@{0}、stash@{1}、stash@{2}）。每次新创建的储藏都为stash@{0}，旧储藏的索引都会+1。
- 恢复储藏的工作目录是干净的（执行git status显示没改动），并且属于同一分支；但是一个干净的工作目录和应用到相同的分支的要求并不是恢复储藏的必要条件。你可以在其中一个分支上保留一份储藏，随后切换到另外一个分支，再重新恢复这些变更。在工作目录里包含已修改和未提交的文件时，你也可以恢复储藏，但如果有任何变更无法被应用（如果现在更改的一个文件，在储藏中也更改了），则Git会该出归并冲突
- 默认情况下，git stash 只会储藏已经在索引中的文件。如果指定 --include-untracked 或 -u 标记，Git 也会储藏任何创建的未跟踪文件。
- 如果指定了 --patch 标记，Git 不会储藏所有修改过的任何东西，但是会交互式地提示哪些改动想要储藏、哪些改动需要保存在工作目录中。
- git stash命令的 --keep-index 选项。会让Git不储藏任何你通过 git add 命令已暂存的东西。
- git stash命令的 --all 选项。比--keep-index更加强大，它会吧.gitignore等其他忽略文件中的模式匹配的文件都储藏。

**储藏示例**

更改完文件之后，储藏工作状态（储藏之后，git status显示没有任何更改）

```bash
[10.208.3.20 root@test-3:~/testapp]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   issue2
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   issue2
#
[10.208.3.20 root@test-3:~/testapp]# git stash
Saved working directory and index state WIP on master: cec3c1d 0.3.7
HEAD is now at cec3c1d 0.3.7
[10.208.3.20 root@test-3:~/testapp]# git status
# On branch master
nothing to commit, working directory clean
```

重新更改这个文件，然后继续储藏工作状态

```bash
[10.208.3.20 root@test-3:~/testapp]# echo hello9999 >> issue2 
[10.208.3.20 root@test-3:~/testapp]# git add .
[10.208.3.20 root@test-3:~/testapp]# git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   issue2
#
[10.208.3.20 root@test-3:~/testapp]# git stash
Saved working directory and index state WIP on master: cec3c1d 0.3.7
HEAD is now at cec3c1d 0.3.7
```

查看储藏列表

```bash
[10.208.3.20 root@test-3:~/testapp]# git stash list
stash@{0}: WIP on master: cec3c1d 0.3.7
stash@{1}: WIP on master: cec3c1d 0.3.7
```

恢复最开始的那个储藏

```bash
##发现暂存区中的内容没有恢复，只是将工作目录恢复了。这是因为只有在执行恢复的时候指定--index，才会恢复暂存区的内容。因此我们执行git reset回到上次commit的状态
[10.208.3.20 root@test-3:~/testapp]# git stash apply stash@{1}
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   issue2
#
no changes added to commit (use "git add" and/or "git commit -a")
[10.208.3.20 root@test-3:~/testapp]# cat .git/refs/heads/master 
cec3c1d079b8a9c0f5f68136df4249d6522f58d1
[10.208.3.20 root@test-3:~/testapp]# git reset --hard cec3c1d079b8a9c0f5f68136df4249d6522f58d1
HEAD is now at cec3c1d 0.3.7
```

使用--index重新恢复储藏，状态成功恢复

```bash
[10.208.3.20 root@test-3:~/testapp]# git stash apply --index  stash@{1}
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   issue2
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   issue2
#
```

删除储藏

```BASH
[10.208.3.20 root@test-3:~/testapp]# git stash drop stash@{0}
Dropped stash@{0} (1c79b8ff4dbdfeddcfe30950113a6856b64dcbf5)
[10.208.3.20 root@test-3:~/testapp]# git stash list
stash@{0}: WIP on master: cec3c1d 0.3.7
```

也可以使用git stash pop在恢复储藏成功之后，自动且将其从堆栈中移除（恢复失败不会移除）

```BASH
[10.208.3.20 root@test-3:~/testapp]# git stash list
stash@{0}: WIP on master: cec3c1d 0.3.7
[10.208.3.20 root@test-3:~/testapp]# git stash pop --index  stash@{0}
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   issue2
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   issue2
#
Dropped stash@{0} (ad34fd58099384eb9ab0997324abb67c687b21fd)
[10.208.3.20 root@test-3:~/testapp]# git stash list
没有显示了，成功移除
```

使用储藏创建分支

这会检出储藏工作时所处的提交，并应用储藏中的工作，如果成功，则会丢弃储藏。

使用储藏stash@{0}创建一个叫做testchanges2的分支

```BASH
[10.208.3.20 root@test-3:~/testapp]# git stash branch testchanges2 stash@{0}
Switched to a new branch 'testchanges2'
# On branch testchanges2
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   issue2
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   issue2
#
Dropped stash@{0} (7e65b261c3205dc3f2d5775c52a176634445ff61)
```

**清理工作目录**

对于工作目录中一些工作或文件，你想做的也许不是储藏而是移除。git clean 命令会帮你做这些事。

有一些通用的原因比如说为了移除由合并或外部工具生成的东西，或是为了运行一个干净的构建而移除之前构建
的残留。

你需要谨慎地使用这个命令，因为它被设计为从工作目录中移除未被追踪的文件。如果你改变主意了，你也不一定能找回来那些文件的内容。一个更安全的选项是运行 git stash --all 来移除每一样东西并存放在栈中。

你可以使用`git clean`命令去除冗余文件或者清理工作目录。使用`git clean -f -d`命令来移除工作目录中所有未追踪的文件以及空的子目录。-f 意味着 强制 或 “确定移除”。

如果只是想要看看它会做什么，可以使用 -n 选项来运行命令，这意味着 “做一次演习然后告诉你 将要 移除什么”。

```BASH
$ git clean -d -n
Would remove test.o
Would remove tmp/
```

默认情况下，git clean 命令只会移除没有忽略的未跟踪文件。任何与 .gitignore 或其他忽略文件中的模式匹配的文件都不会被移除。如果你也想要移除那些文件，例如为了做一次完全干净的构建而移除所有由构建生成的 .o 文件，可以给 clean 命令增加一个 -x 选项。

```BASH
$ git status -s
 M lib/simplegit.rb
?? build.TMP
?? tmp/
$ git clean -n -d
Would remove build.TMP
Would remove tmp/
$ git clean -n -d -x
Would remove build.TMP
Would remove test.o
Would remove tmp/
```

如果不知道 git clean 命令将会做什么，在将 -n 改为 -f 来真正做之前总是先用 -n 来运行它做双重检查。另一个小心处理过程的方式是使用 -i 或 “interactive” 标记来运行它。

这将会以交互模式运行 clean 命令。

```bash
$ git clean -x -i
Would remove the following items:
  build.TMP test.o
*** Commands ***
    1: clean 2: filter by pattern 3: select by numbers 4: ask each 5: quit 6: help
What now>
```

这种方式下可以分别地检查每一个文件或者交互地指定删除的模式。

# Git内部原理

一个新目录执行git init是，git会创建一个.git目录。这个目录包含了几乎所有Git存储和操作的对象。如果想备份或复制一个版本库，只需要报这个目录拷贝到另一个位置即可。

```BASH
[10.208.3.20 root@test-3:~]# mkdir testapp
[10.208.3.20 root@test-3:~]# cd testapp/
[10.208.3.20 root@test-3:~/testapp]# git init
[10.208.3.20 root@test-3:~/testapp]# tree -aF ./        ##执行git init之后，会创建一个.git的隐藏目录，该目录的目录结构如下
./
└── .git/
    ├── branches/
    ├── config        ##当前仓库的配置文件，除了此文件还有全局配置文件~/.gitconfig和系统配置文件/etc/gitconfig
    ├── description   ##提供GitWeb程序使用
    ├── HEAD        ##指示目前被检出的分支（指向当前分支的最新一个commit）
    ├── hooks/      ##客户端或服务端的钩子脚本
    │   ├── applypatch-msg.sample*
    │   ├── commit-msg.sample*
    │   ├── post-update.sample*
    │   ├── pre-applypatch.sample*
    │   ├── pre-commit.sample*
    │   ├── prepare-commit-msg.sample*
    │   ├── pre-push.sample
    │   ├── pre-rebase.sample*
    │   └── update.sample*
    ├── info/    ##包含一个全局性排除文件，用以放置那些不希望被记录在.gitignore文件中的忽略模式
    │   └── exclude    ##git ls-lists默认读取的忽略文件列表
    ├── objects/        ##存储所有数据内容
    │   ├── info/
    │   └── pack/
    └── refs/            ##存储指向数据（分支）的提交对象的指针
        ├── heads/
        └── tags/
     此时未创建的index文件	  ##保存暂存区信息
```

## Git对象

每次我们运行 git add 和 git commit 命令时， Git 所做的实质工作：将被改写的文件保存为数据对象，更新暂存区，记录树对象，最后创建一个指明了顶层树对象和父提交的提交对象。这三种主要的 Git 对象（数据对象、树对象、提交对象）最初均以单独文件的形式保存在 .git/objects 目录下。

### 数据对象（blob object）

Git 是一个内容寻址文件系统，它的核心部分是一个简单的键值对数据库（key-value data store）。你可以向该数据库插入任意类型的内容，它会返回一个键值，通过该键值可以在任意时刻再次检索（retrieve）该内容。可以通过底层命令 hash-object 来演示上述效果——该命令可将任意数据保存于 .git 目录，并返回相应的键值。

向Git数据库中存入些文件：

```BASH
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

-w 选项指示 hash-object 命令存储数据对象；若不指定此选项，则该命令仅返回对应的键值。--stdin 选项则指示该命令从标准输入读取内容；若不指定此选项，则须在命令尾部给出待存储文件的路径。该命令输出一个长度为 40 个字符的校验和。这是一个 SHA-1 哈希值——一个将待存储的数据外加一个头部信息（header）一起做 SHA-1 校验运算而得的校验和。

```BASH
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

可以在 objects 目录下看到一个文件。这就是开始时 Git 存储内容的方式（一个文件对应一条内容，以该内容加上特定头部信息一起的 SHA-1 校验和为文件命名）。校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。

可以通过 cat-file 命令从 Git 那里取回数据。为cat-file指定 -p 选项可指示该命令自动判断内容的类型，并为我们显示格式友好的内容：

```BASH
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

我们同样可以将这些操作应用于文件中的内容。例如，可以对一个文件进行简单的版本控制。首先，创建一个新文件并将其内容存入数据库：

```BASH
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
```

接着，向文件里写入新内容，并再次将其存入数据库：

```bash
$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
```

数据库记录下了该文件的两个不同版本(之前我们存入的第一条内容也还在）：

```BASH
$ find .git/objects -type f
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

我们可以通过`git cat-file -p XXXX`将文件恢复到任意一个版本。但是记住文件的每一个版本所对应的 SHA-1 值并不现实；另一个问题是，在这个（简单的版本控制）系统中，文件名并没有被保存——我们仅保存了文件的内容。上述类型的对象我们称之为数据对象（blob object）。利用 cat-file -t 命令，可以让 Git 告诉我们其内部存储的任何对象类型，只要给定该对象的 SHA-1 值：

```BASH
$ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
blob
```

### 树对象（tree object）

树对象能解决文件名保存的问题，也允许我们将多个文件组织到一起。Git 以一种类似于 UNIX 文件系统的方式存储内容，但作了些许简化。所有内容均以树对象和数据对象的形式存储，其中树对象对应了 UNIX 中的目录项，数据对象则大致上对应了 inodes 或文件内容。一个树对象包含了一条或多条树对象记录（tree entry），每条记录含有一个指向数据对象或者子树对象的 SHA-1 指针，以及相应的模式、类型、文件名信息。例如：

```BASH
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859 README
100644 blob 8f94139338f9404f26296befa88755fc2598c289 Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0 lib
```

master^{tree} 语法表示 master 分支上最新的提交所指向的树对象。请注意，lib 子目录（所对应的那条树对象记录）并不是一个数据对象，而是一个指针，其指向的是另一个树对象：

```BASH
$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b simplegit.rb
```

从概念上讲，Git 内部存储的数据有点像这样：

![image-20200418204301941](.assets/image-20200418204301941.png)

### 提交对象（commit object）

暂存操作会为每一个文件计算校验和（ SHA-1哈希算法），然后会把当前版本的文件快照保存到Git仓库中（Git 使用 blob 对象来保存它们），最终将校验和加入到暂存区域等待提交

```BASH
$ git add README test.rb LICENSE
$ git commit -m 'The initial commit of my project'
```

假如现在有三个树对象，分别代表了我们想要跟踪的不同项目快照。若想重用这些快照，你必须记住所有三个 SHA-1 哈希值。并且，你也完全不知道是谁保存了这些快照，在什么时刻保存的，以及为什么保存这些快照。而以上这些，正是提交对象（commit object）能为你保存的基本信息。

在进行提交操作时，Git 会先计算每一个子目录（本例中只有项目根目录）的校验和，然后在Git 仓库中这些校验和保存为树对象。随后，Git 会保存一个提交对象（commit object），该提交对象会包含一个指向暂存内容快照的指针（这个项目根目录的树对象的指针）。但不仅仅是这样，该提交对象还包含了指向它的父对象的指针、作者的姓名和邮箱、时间戳、留空一行最后是提交注释。如此一来，Git 就可以在需要的时候重现此次保存的快照。

首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象。

```BASH
$ git cat-file -p fdf4fc3
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Scott Chacon <schacon@gmail.com> 1243040974 -0700
committer Scott Chacon <schacon@gmail.com> 1243040974 -0700

first commit
```

### 标签对象（tag object）

标签对象（tag object）非常类似于一个提交对象——它包含一个标签创建者信息、一个日期、一段注释信息，以及一个指针。主要的区别在于，标签对象通常指向一个提交对象，而不是一个树对象。它像是一个永不移动的分支引用（永远指向同一个提交对象），只不过给这个提交对象加上一个更友好的名字罢了。

标签有两种类型：附注标签和轻量标签。

## Git引用

我们可以借助类似于 git log 1a410e 这样的命令来浏览完整的提交历史，但为了能遍历那段历史从而找到所
有相关对象，你仍须记住 1a410e 是最后一个提交。我们需要一个文件来保存 SHA-1 值，并给文件起一个简单
的名字，然后用这个名字指针来替代原始的 SHA-1 值。
在 Git 里，这样的文件被称为“引用（references，或缩写为 refs）”；你可以在 .git/refs 目录下找到这类
含有 SHA-1 值的文件。在目前的项目中，这个目录没有包含任何文件，但它包含了一个简单的目录结构：

```BASH
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/tags
```

如果想更新某个引用，Git 提供了一个更加安全的命令 update-ref 来完成此事：

```BASH
$ git update-ref refs/heads/master
1a410efbd13591db07496601ebc7a059dd55cfe9
```

这基本就是 Git 分支的本质：一个指向某一系列提交之首的指针或引用。若想在其他提交上创建一个分支，可以这么做：

```BASH
$ git update-ref refs/heads/test cac0ca
```

当运行类似于 git branch (branchname) 这样的命令时，Git 实际上会运行 update-ref 命令，取得当前所在分支最新提交对应的 SHA-1 值，并将其加入你想要创建的任何新引用中。

**Git会自动维护几个特定目的的特殊符号引用**

- HEAD：始终指向当前分支的最近提交；或检出到其它分支时，目标分支的最近提交；
- ORIG_HEAD：合并操作时，新生成的提交前面的那一个提交保存于此引用中；存储在.git/ORIG_HEAD文件中。比如是在master分支中将develop分支合并，则此处合并前保存master分支的上一个提交，即便checkout到develop分支中查看这个引用也是一样。
- FETCHED_HEAD：获取时（比如clone）对应的远程分支的最近一次提交，可以想象为.git/refs/remotes/origin/master
- MERGE_HEAD：合并操作时，其它分支的上一次提交

### HEAD 引用

HEAD 文件是一个符号引用（symbolic reference），指向目前所在的分支。所谓符号引用，意味着它并不像普通引用那样包含一个 SHA-1 值——它是一个指向其他引用的指针。如果查看 HEAD 文件的内容，一般而言我们看到的类似这样：

```BASH
$ cat .git/HEAD
ref: refs/heads/master
```

如果执行 git checkout test，Git 会像这样更新 HEAD 文件：

```BASH
$ cat .git/HEAD
ref: refs/heads/test
```

当我们执行 git commit 时，该命令会创建一个提交对象，并用 HEAD 文件中那个引用所指向的 SHA-1 值设置其父提交字段。

你也可以手动编辑该文件，然而同样存在一个更安全的命令来完成此事：symbolic-ref。可以借助此命令来查看 HEAD 引用对应的值和设置HEAD引用的值：

```BASH
$ git symbolic-ref HEAD
refs/heads/master
$ git symbolic-ref HEAD refs/heads/test
$ cat .git/HEAD
ref: refs/heads/test
```

### 标签引用

标签有两种类型：附注标签和轻量标签。

这就是轻量标签的全部内容（一个固定的引用）

```BASH
$ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d
```

与附注标签，Git会创建一个标签对象，并记录一个引用来指向该标签对象，而不是直接指向提交对象

```BASH
$ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'
```

下面是上述过程所建标签对象的 SHA-1 值：

```BASH
$ cat .git/refs/tags/v1.1
9585191f37f7b0fb9444f35a9bf50de191beadc2
```

对该 SHA-1 值运行 cat-file 命令：

```BASH
$ git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2
object 1a410efbd13591db07496601ebc7a059dd55cfe9
type commit
tag v1.1
tagger Scott Chacon <schacon@gmail.com> Sat May 23 16:48:58 2009 -0700
test tag
```

object 条目指向我们打了标签的那个提交对象的 SHA-1 值。另外要注意的是，标签对象并非必须指向某个提交对象；你可以对任意类型的 Git 对象打标签。例如，在 Git 源码中，项目维护者将他们的 GPG 公钥添加为一个数据对象，然后对这个对象打了一个标签。

### 远程引用

如果你添加了一个远程版本库并对其执行过推
送操作，Git 会记录下最近一次推送操作时每一个分支所对应的值，并保存在 refs/remotes 目录下。例如，
你可以添加一个叫做 origin 的远程版本库，然后把 master 分支推送上去：

```BASH
$ git remote add origin git@github.com:schacon/simplegit-progit.git
$ git push origin master
Counting objects: 11, done.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (7/7), 716 bytes, done.
Total 7 (delta 2), reused 4 (delta 1)
To git@github.com:schacon/simplegit-progit.git
a11bef0..ca82a6d master -> master
```

此时，如果查看 refs/remotes/origin/master 文件，可以发现 origin 远程版本库的 master 分支所对应的 SHA-1 值，就是最近一次与服务器通信时本地 master 分支所对应的 SHA-1 值.

远程引用和分支（位于 refs/heads 目录下的引用）之间最主要的区别在于，远程引用是只读的。虽然可以git checkout 到某个远程引用，但是 Git 并不会将 HEAD 引用指向该远程引用。因此，你永远不能通过commit 命令来更新远程引用。Git 将这些远程引用作为记录远程服务器上各分支最后已知位置状态的书签来管理。

想要在当前仓库中，fetch后的远程仓库创建新分支的方法是`git checkout -b newbranch origin/xxx`，这就会依据那个远程仓库，创建一个新的分支。

## 包文件

**主要是说git如何通过打包对象减少了磁盘空间占用的。参考[ProGit](https://www.progit.cn/)一书即可。**

这会涉及到object目录下的pack目录及其下的文件。

## 引用规格

**简单的远程分支到本地引用的简单映射方式如下：**

```BASH
$ git remote add origin https://github.com/schacon/simplegit-progit
```

执行这条命令会在.git/config文件中添加下面这些内容

```BASH
[remote "origin"]
url = https://github.com/schacon/simplegit-progit
```

**复杂的映射方式此处不做介绍，参考[ProGit](https://www.progit.cn/)一书即可。**

**引用规格推送**

如果 QA 团队想把他们的 master 分支推送到远程服务器的 qa/master 分支上，可以运行：

```BASH
$ git push origin master:refs/heads/qa/master
```

如果他们希望 Git 每次运行 git push origin 时都像上面这样推送，可以在他们的配置文件中添加一条 push值：

```BASH
[remote "origin"]
url = https://github.com/schacon/simplegit-progit
fetch = +refs/heads/*:refs/remotes/origin/*
push = refs/heads/master:refs/heads/qa/master
```
