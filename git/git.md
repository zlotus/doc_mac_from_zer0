# git

使用说明参考：<http://rogerdudler.github.io/git-guide/index.zh.html> 和 <http://www.cnblogs.com/findingsea/archive/2012/08/27/2654549.html>

先在 [Github](https://github.com/) 上建立 repository *doc_mac_from_zer0*

## 初始化前确认认证的公钥：

    $ ssh -T git@github.com
    
应返回：

    Hi zlotus! You've successfully authenticated, but GitHub does not provide shell access.

## 克隆库：

    $ git clone https://github.com/zlotus/doc_mac_from_zer0.git
    
## 上传 README.md：

    $ git init
    $ touch README.md
    $ git add README.md
    $ git commit -m 'first_commit'
    $ git remote add origin https://github.com/zlotus/doc_mac_from_zer0.git
    $ git push origin master
    
## push 文件

    $ git add .
    $ git commit -m 'first_commit'
    $ git remote add origin https://github.com/zlotus/doc_mac_from_zer0.git
    $ git push origin master


### 在检查连接是否成功时报错：

    $ ssh -T git@github.com
    Permission denied (publickey).

    
解决参考：<http://blog.csdn.net/keyboardota/article/details/7603630>

> 使用git clone命令从github上同步github上的代码库时，如果使用SSH链接（如我自己的beagleOS项目：git@github.com:DamonDeng/beagleOS.git），而你的SSH key没有添加到github帐号设置中，
> 系统会报下面的错误：
> 
    Permission denied (publickey).
    fatal: The remote end hung up unexpectedly

> 这时需要在本地创建SSH key，然后将生成的SSH key文件内容添加到github帐号上去。
> 创建SSH key的方法很简单，执行如下命令就可以：
>
    $ ssh-keygen

> 然后系统提示输入文件保存位置等信息，连续敲三次回车即可，生成的SSH key文件保存在中～/.ssh/id_rsa.pub

> 然后用文本编辑工具打开该文件，我用的是 `vim`，所以命令是：
>
    $ vim ~/.ssh/id_rsa.pub

> 接着拷贝.ssh/id_rsa.pub文件内的所以内容，将它粘帖到 **github** 帐号管理中的添加 SSH key 界面中。

> 打开 **github** 帐号管理中的添加 SSH key 界面的步骤如下：
> 
1. 登录 `github`
2. 点击右上方的 `Accounting settings` 图标
3. 选择 `SSH key`
4. 点击 `Add SSH key`
5. 在出现的界面中填写 SSH key `Title`，填一个你自己喜欢的名称即可，然后将上面拷贝的~/.ssh/id_rsa.pub文件内容粘帖到 `key` 一栏，在点击 `add key` 按钮就可以了。
6. 添加过程github会提示你输入一次你的github密码

> 添加完成后再次执行git clone就可以成功克隆github上的代码库了。

完成了 SSH Key 的添加后：

    $ .ssh  ssh -T git@github.com           
    Hi zlotus! You've successfully authenticated, but GitHub does not provide shell access.

再提交：

    $ git push origin master

附 **git** 常用命令图解 <http://marklodato.github.io/visual-git-guide/index-zh-cn.html>

## 提交到 HEAD

    $ git add *
    $ git commit -m "add git commit & iTerm2"

## 推送改动到远端仓库

    $ git push origin master

## git 日常命令速查

来自：<http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000>

作者 @ [廖雪峰](http://weibo.com/liaoxuefeng)

我把教程里的命令列出来方便查找，详情请访问原始地址，**在网上见过的最好的中文教程**。

###### 把一个本地目录变成 git 仓库：

    $ git init

###### 将改动添加到给 stage(index)：

    $ git add <filename>

###### 将 stage 中的改动提交给仓库：

    $ git commit -m "<change commit>"

###### 查看工作区状态：

    $ git status

###### 查看工作区文件与版本库文件的变动，`HEAD`表示版本库最新版本：

    $ git diff <filename>
    $ git diff HEAD -- <filename>

###### 查看版本历史记录，参数`--pretty=oneline`为概况：

    $ git log
    $ git log --pretty=oneline

###### 退回上一个历史版本，n个`^`代表退回上n个历史版本，或简写成`~n`：

    $ git reset --hard HEAD^

###### 查看历史操作：

    $ git reflog

###### 丢弃工作区的修改：

    $ git checkout -- <filename>

###### 撤销(unstage)暂存区的修改，重新放回工作区：

    $ git reset HEAD <filename>

###### 从版本库中移除文件：

    $ git rm <filename>

###### 从版本库中恢复删除的文件：

    $ git checkout -- <filename>

###### 创建 SSH KEY，用于在 GitHub 上添加 SSH KEY ，连接远程仓库：

    $ ssh-keygen -t rsa -C "youremail@example.com"

###### 关联远程仓库：

    $ git remote add origin git@github.com:yourdir/yourrepo.git

###### 将本地库推送到远程库：

    $ git push -u origin master

###### 克隆远程库到本地，通过 SSH 或 HTTPS ：

    $ git clone git@github.com:yourdir/yourrepo.git
    $ git clone https://github.com/yourdir/yourrepo.git

###### 创建，切换到分支：

    # create branch: 
    $ git branch <branchname>
    
    # checkout branch
    $ git checkout <branchname>

    # create & checkout branch: 
    $ git checkout -b <branchname>
    
###### 查看当前分支：

    $ git branch

###### 合并分支：

    $ git merge <branchname>

###### 删除分支：

    $ git branch -d <branchname>

###### 有图有真相的查看分支：

    $ git log --graph --pretty=oneline --abbrev-commit

###### 合并时禁用 fast forward 策略：

    $ git merge --no-ff -m "merge with no-ff" dev

###### 暂存工作区现场：

    $ git stash

###### 查看暂存区：

    $ git stash list

###### 从暂存区恢复工作现场，并清理暂存区：

    # recover: 
    $ git stash apply stash@{stashid}
    
    # clean:
    $ git stash drop
    
    # recover & clean
    $ git stash pop

###### 强行销毁还没有被合并的分支，此操作会丢失修改数据：

    $ git branch -D feature-vulcan

###### 查看远程库的信息，使用`-v`查看详细信息：

    $ git remote
    $ git remote -v

###### 推送分支<branch-name>：

    $ git push origin <branch-name>

###### 创建远程origin的<branch-name>分支到本地，建立本地<branch-name>：

    $ git checkout -b dev origin/<branch-name>

###### 建立本地<branch-name>与origin/<branch-name>之间的连接：

    $ git branch --set-upstream <branch-name> origin/<branch-name>

###### 获得最新的当前分支：

    $ git pull

###### 为当前分支版本创建标签，或根据<commit-id>创建标签：

    $ git tag <tag-name>
    $ git tag <tag-name> <commit-id>
    $ git tag -a <tag-name> -m "<commit-string>" <commit-id>

###### 查看现有标签：

    $ git tag

###### 查看标签信息：

    $ git show <tag-name>

###### 用PGP签名标签：

    $ git tag -s <tag-name> -m "<commit-string>" <commit-id>

###### 删除标签：

    $ git tag -d <tag-name>

###### 推送<tag-name>标签到远程库，或推送所有未推送的tag到远程库：

    $ git push origin <tag-name>
    $ git push origin --tags

###### 删除远程库标签，要先删除本地tag，再从远程库中删除：

    $ git tag -d <tag-name>
    $ git push origin :refs/tags/<tag-name>

###### 在git中配置alias：

    $ git config --global alias.<alias-string> <origin-string>
    
    # looks awsome! 
    $ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

###### 在git中配置proxy，有时候会被各种各样的墙挡住去路：

    # --local modify .git/config
    # --global modify ~/.gitconfig
    git config --local http.proxy http://127.0.0.1:8888
