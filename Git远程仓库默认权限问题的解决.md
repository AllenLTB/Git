# Git远程仓库默认权限问题的解决

多人共同开发维护一个项目时，对整个项目文件互有拉取、推送等行为。为防止操作时文件权限出现冲突，可有以下2种方法解决：

1. 本地git的远端设置中，连接远程仓库时多人使用同一个用户名，该用户名为git远程仓库的拥有者
2. 不同开发者需使用不同的用户名连接时，会出现文件权限冲突，这是因为 git 仓库使用的是对象存储，每次改动会新增若干对象文件（具体对象文件在 /.git/objects 下），而新增的对象文件权限属性由系统控制，默认为755，即非文件拥有着无法进行写入（推送）。这里需要将 git 仓库下文件的默认权限设置为同组用户均可读写执行。

不管使用哪种方法，都建议同时设置所有用户的shell为git-shell（`uermod -s /usr/bin/git-shell xxx`），以提升安全性。

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



