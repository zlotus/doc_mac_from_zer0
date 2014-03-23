# 使用 zsh

## 安装X11

下载 **X11** <http://xquartz.macosforge.org/landing/>

**X11** 安装指南 <http://guide.macports.org/#installing.xcode>

## 安装 Xcode

从Mac Apple Store下载

## 安装 MacPorts

下载 **Mavericks** 的 **MacPorts**

安装...

等待时间挺长的...

## 安装 git

额，怎么 **Maverick** 自带 **git**…

    man git

## 顺便安装 wget 测试 MacPorts

**wget** 为非必须，只是试一下 **MacPorts** 的威力，第一次运行先更新下，`-v` 有详细信息：

    $ sudo port -v selfupdate

搜索 **wget**

    $ port search wget
    wget @1.15 (net, www)
    internet file retriever

再用 **MacPorts** 下载：

    $ sudo port install wget
    --->  Computing dependencies for wget
    ... doing something ...
    --->  Cleaning wget
    --->  Updating database of binaries: 100.0%
    --->  Scanning binaries for linking errors:100.0%
    --->  No broken files found.

**MacPorts** 其他几个常用的命令：

    $ sudo port uninstall wget
    
    # List the installed ports that need upgrading.
    $ port outdated
    
    $ sudo port upgrade outdated
    
    $ port list
    
    # Displays meta-information available for portname.
    $ port info wget
    
    # Lists the other ports that are required to build and run wget 
    $ port deps wget
    
    # Lists the build variants available for wget.
    $ port variants wget


全自动安装、激活、清理。据说是出于编译原因，用MacPorts安装的包都放在了 `/opt/local/` 下。

## 安装oh-my-zsh

按照 <https://github.com/robbyrussell/oh-my-zsh> 手册安装

    # The manual way
    # 1. Clone the repository
    $ git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
    
    # 2. OPTIONAL Backup your existing ~/.zshrc file
    $ cp ~/.zshrc ~/.zshrc.orig
    
    # 3. Create a new zsh config by copying the zsh template we’ve provided.
    $ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
    
    # 4. Set zsh as your default shell:
    $ chsh -s /bin/zsh
    
    # 5. Start / restart zsh (open a new terminal is easy enough…)

##配置oh-my-zsh

试了几个外部命令，发现 **`port`** 不见了，打开 `~/.zshrc` 配置环境变量，将 bash 中 MacPorts 自动添加的 `PATH` 拷到 `.zshrc` 中：

    # check current PATH, HOME
    $ echo $PATH
    $ echo $HOME
    
    # edit in vi
    $ vi ~/.zshrc ~/.bash_profile
    
    # MacPort added in .bash_profile, copy this: 
    → export PATH=/opt/local/bin:/opt/local/sbin:$PATH
    
    # to .zshrc
    → export PATH=$HOME/bin:/opt/local/bin:/opt/local/sbin:/usr/local/bin:$PATH

附上从酷壳中找到的 [vim速查手册](http://jrmiii.com/attachments/Vim.pdf)