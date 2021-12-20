### push Authentication failed for

git config --list 查看用户信息.
如果push遇到在输入密码是熟错后，就会报这个错误 fatal: Authentication failed for.
git config --system --unset credential.helper 之后你在 push 就会提示输入名称和密码.

### push push failed to push some refs to

![gitpush失败2](D:\Typora\image\20170416195411727.png)
git pull –rebase origin master 意为先取消 commit 记录，并且把它们临时 保存为补丁(patch)(这些补丁放到”.git/rebase”目录中)，之后同步远程库到本地，最后合并补丁到本地库之中。
![gitpush失败3](D:\Typora\image\20170416195452761.png) 
git pull --rebase origin master 接下来就可以把本地库push到远程库当中了.
![gitpush失败4](D:\Typora\image\20170416200330505.png)