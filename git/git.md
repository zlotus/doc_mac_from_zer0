# git

使用说明参考：<http://rogerdudler.github.io/git-guide/index.zh.html> 和 <http://www.cnblogs.com/findingsea/archive/2012/08/27/2654549.html>

先在 [Github](https://github.com/) 上建立 repository *doc_mac_from_zer0*

## 初始化前确认认证的公钥：

     $ ssh -T git@github.com
应返回：

    → Hi zlotus! You've successfully authenticated, but GitHub does not provide shell access.

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
    → Permission denied (publickey).

    
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
    → Hi zlotus! You've successfully authenticated, but GitHub does not provide shell access.

再提交：

    $ git push origin master

附 **git** 常用命令图解 <http://marklodato.github.io/visual-git-guide/index-zh-cn.html>

## 提交到HEAD

    $ git add *
    $ git commit -m "add git commit & iTerm2"

## 推送改动到远端仓库

    $ git push origin master
