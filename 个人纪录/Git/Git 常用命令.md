https://www.progit.cn/
https://gitee.com/progit/

### 分支命名规范

主分支: master 主分支，所有提供给用户使用的正式版本，都在这个主分支上发布
开发分支: dev-* 开发分支，永远是功能最新最全的分支
功能分支: feature-* 新功能分支，某个功能点正在开发阶段
发布版本: release-* 发布定期要上线的功能
修复分支: bug-* 修复线上代码的 bug

### 更新远程分支

git remote update origin --prune

### 创建远程分支

git checkout -b my-test  //在当前分支下创建my-test的本地分支分支
git push origin my-test  //将my-test分支推送到远程
git branch --set-upstream-to=origin/my-test //将本地分支my-test关联到远程分支my-test上

### 删除远程分支

git push origin --delete <分支名>
git branch -d <分支名> 如果失败则用大写的 D 强制删除

### 储藏

git stash save "message" 储藏代码
git stash pop stash@{1} 恢复指定的进度到工作区

### 设置与配置

git config 设置
git help 用来显示任何命令的 Git 自带文档

### 获取与创建项目

git init 将一个目录转变成一个 Git 仓库
git clone 克隆一个现有的远程仓库

### 快照基础

git add 从工作目录添加到暂存区
git status 展示工作区及暂存区域中不同状态的文件
git diff 查看工作环境与的暂存区的差异
git difftool 可以用来简单地启动一个外部工具来为你展示两棵树之间的差异
git commit 将所有通过 add 暂存的文件内容在数据库中创建一个持久的快照，然后将当前分支上的分支指针移到其之上
git reset 用来根据你传递给动作的参数来执行撤销操作

- --soft 参数 Git 重置 HEAD 到另外一个 commit,但也到此为止工作区、暂存区都不会有任何改变
- --mixed 默认参数 Git 重置 HEAD 到另外一个 commit 并且重置暂存区以便和版本库相匹配
- --hard 参数 Git 重置 HEAD 返回到另外一个 commit 重置暂存区以便反映 HEAD 的变化，并且重置工作区也使得其完全匹配起来。

git rm 用来从工作区或者暂存区移除文件的命令
git mv 用于移到一个文件并且在新文件上执行 add 命令及在老文件上执行 git rm 命令
git clean 用来从工作区中移除不想要的文件的命令

### 分支

git branch 列出所有的本地分支
git checkout 用来切换分支或者检出内容到工作目录
git merge 用来合并一个或者多个分支到你已经检出的分支中
git mergetool 当你在 Git 的合并中遇到问题时，可以使用 git mergetool 来启动一个外部的合并帮助工具
git log 用来展示一个项目的可达历史记录，从最近的提交快照起
git stash 用来临时地保存一些还没有提交的工作，以便在分支上不需要提交未完成工作就可以清理工作目录
git tag 用来为代码历史记录中的某一个点指定一个永久的书签一般来说它用于发布相关事项

### 项目

git fetch 命令与一个远程的仓库交互，并且将远程仓库中有但是在当前仓库的没有的所有信息拉取下来然后存储在你本地数据库中
git pull 从指定的远程仓库中抓取内容，然后马上尝试将其合并进所在的分支中
git push 用来与另一个仓库通信，计算本地数据库与远程仓库的差异，然后将差异推送到另一个仓库中。 它需要有另一个仓库的写权限，因此这通常是需要验证的
git remote 是一个远程仓库记录的管理工具，它允许你将一个长的 URL 保存成一个简写的句柄，例如 origin ，这样你就可以不用每次都输入他们了
git archive 用来创建项目一个指定快照的归档文件
git submodule 用来管理一个仓库的其他外部仓库。 它可以被用在库或者其他类型的共享资源上

### 查找与比较

git show 可以以一种简单的人类可读的方式来显示一个 Git 对象
git shortlog 是一个用来归纳 git log 的输出的命令
git describe 用来接受任何可以解析成一个提交的东西，然后生成一个人类可读的字符串且不可变

### 调试

git bisect 通过自动进行一个二分查找来找到哪一个特定的提交是导致 bug 或者问题的第一个提交
git blame 标注任何文件的行，指出文件的每一行的最后的变更的提交及谁是那一个提交的作者
git grep 可以帮助在源代码中，甚至是你项目的老版本中的任意文件中查找任何字符串或者正则表达式

### 补丁

git cherry-pick 用来获得在单个提交中引入的变更，然后尝试将作为一个新的提交引入到你当前分支上
git rebase 基本是是一个自动化的 cherry-pick 命令.它计算出一系列的提交，然后再以它们在其他地方以同样的顺序一个一个的 cherry-picks 出它们
git revert 撤销众多 commit 中的某一个 commit
rebase 笔记 https://zhangbuhuai.com/post/git-rebase.html

### 设置命令单词颜色

git config --global color.ui true
git config --global color.ui false