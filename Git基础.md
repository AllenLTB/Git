









---------------------------









---------------------







窍门

这个文件可以实现<TAB><TAB>自动不全的功能

如果是源码安装的话就进入都contrib/completion目录，找到git-completion.bash文件，将其复制到/etc/bash_completion.d目录中。

[10.208.3.20 root@test-3:~/testapp]# rpm -qf /etc/bash_completion.d/git
git-1.8.3.1-21.el7_7.x86_64



因为我追踪了分支，所以切换到master分支时，有下面这个提示

```bash
[10.208.3.20 root@test-3:/tmp/testapp]# git checkout master
Switched to branch 'master'
Your branch is behind 'testapp/master' by 4 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)
[10.208.3.20 root@test-3:/tmp/testapp]# git checkout master2
Switched to branch 'master2'
[10.208.3.20 root@test-3:/tmp/testapp]# echo 456234afdkjlasjflajsdlfjalsdjflasdf >> passwd 
[10.208.3.20 root@test-3:/tmp/testapp]# git add .
[10.208.3.20 root@test-3:/tmp/testapp]# git commit -m "0.4.3"
[master2 3b1e39c] 0.4.3
 1 file changed, 1 insertion(+)
[10.208.3.20 root@test-3:/tmp/testapp]# git push testapp master2:master
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 276 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To git@github.com:AllenLTB/testapp.git
   fdcacde..3b1e39c  master2 -> master

[10.208.3.20 root@test-3:/tmp/testapp]# git checkout master 
Switched to branch 'master'
Your branch is behind 'testapp/master' by 5 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)
```



删除远端分支

````bash
[10.208.3.20 root@test-3:/tmp/testapp]# git push testapp :master2
To git@github.com:AllenLTB/testapp.git
 - [deleted]         master2
````

