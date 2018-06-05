---
title: "程序员优雅使用macOS指南"
date: "2018-06-05 16:11:00+0800"
---

最近结合iTerm2+oh-my-zsh+powerline配了一个我自认为堪称艺术品的mac环境出来~

## 软件包管理

### brew管理命令行软件

程序员使用mac要优雅，首先要管理好软件，所有能通过AppStore安装的软件一律通过AppStore管理，AppStore里没有的软件全部尽可能通过brew（一个命令行的包管理工具，相当于第三方AppStore）管理，所以先安装brew(项目官网：[Homebrew](https://brew.sh/index_zh-cn.html))。 打开终端（Launchpad—>其他—>终端，或者直接command+空格，输入terminal然后回车） 在终端键入如下命令：

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

brew可以拿来管理没有图形界面的程序，比如node，git，wget等等，这里介绍一些常用的简单命令：

查看你安装好的所有程序：

```shell
brew list
```

搜索你想要装的app，<your app name>代表你想要安装的app名字：

```shell
brew search <your app name> 
```

安装你想要的app：

```shell
brew install <your app name>
```

更新brew：

```shell
brew update
```

一键更新所有app：

```shell
brew upgrade
```

卸载app：

```shell
brew uninstall <your app name>
```

### brew cask管理图形界面软件

装好brew后只能拿来装命令行下使用的app，还需要brew cask（项目官网：[Homebrew-Cask](https://caskroom.github.io/)）来管理图形界面的程序，键入：

```shell
brew tap homebrew/cask
```

然后通过类似的命令就可以管理图形界面程序。

比如要安装Google Chrome浏览器，就可以先运行：

```shell
brew cask search chrome
```

得到想要的app的全名为google-chrome，然后运行：

```shell
brew cask install google-chrome
```

谷歌浏览器就已经装好了。

### 为brew cask添加一键升级所有软件命令

brew cask虽然好用，但是比起原生的brew，它缺少了很关键的一个功能，就是类似于brew upgrade的一键更新所有已安装的图形界面软件的功能，这个时候就需要[brew-cask-upgrade](https://github.com/buo/homebrew-cask-upgrade)了。

```shell
brew tap buo/cask-upgrade
```

然后当你需要更新所有图形界面软件的时候使用如下命令

```shell
brew cu -a
```

## mac OS终端命令行配置

不得不说mac自带的终端太难用，没有自动补全，也没有着色，所以接下来是配置出一个更强大、更优雅的终端的教程，主要使用iTerm2 + Oh My Zsh + Powerline。这是我最后配置出的效果图：

![img](https://pic3.zhimg.com/50/v2-2f8e6a297ae5b3b3f7daca80389752a3_hd.jpg)

按F12自动出现一个下拉式的终端，有自动补全、着色还有各种酷炫的提示功能。

#### 安装iTerm2代替自带终端

安装iTerm2（项目地址：[iTerm2 - macOS Terminal Replacement](https://iterm2.com/)）代替自带的终端：

```
brew cask install iterm2
```

以后就不用打开自带的终端了，全部使用这个iTerm2。

#### 安装oh-my-zsh（项目地址：[robbyrussell/oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)）代替默认的bash

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

安装了oh-my-zsh后，所有程序员需要的基本功能都已经有了，嫌麻烦的同学做到这一步也就可以了，后面的事情主要是美化字体和着色。

#### 配置zsh的自动补全和高亮的插件

1. 安装[zsh-users/zsh-completions](https://github.com/zsh-users/zsh-completions)：

```shell
git clone https://github.com/zsh-users/zsh-completions ~/.oh-my-zsh/custom/plugins/zsh-completions
```

2. 安装[zsh-users/zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)：

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

然后把这两个插件添加到zsh里面，oh-my-zsh自带了一些插件也可以用上，用vim或者别的编辑器编辑你当前用户目录下的.zshrc文件：

```shell
vim ~/.zshrc
```

修改plugins这一行使之如图所示：

![img](https://pic4.zhimg.com/50/v2-6f18abf3b9bd27333cdd2d59cc0538a8_hd.jpg)

然后运行以下命令以使其生效：

```shell
source ~/.zshrc
```

没有生效的话就删掉.zcompdump文件再重启终端（碰到一些奇怪的问题也可以这样解决）：

```shell
rm ~/.zcompdump
```

#### 使用agnoster主题

1. 下载[agnoster](https://github.com/fcamblor/oh-my-zsh-agnoster-fcamblor)主题，到下载的工程里面运行install文件`./install`，主题将安装到~/.oh-my-zsh/themes目录下。

2. 设置使用该主题，进入~/.zshrc打开.zshrc文件，然后将ZSH_THEME后面的字段改为agnoster，即ZSH_THEME="agnoster"（agnoster即为要设置的主题）。

#### 安装powerline字体

第4步安装的主题需要配合powerline字体才能使用，接下来安装powerline字体，地址在这里：[powerline/fonts](https://github.com/powerline/fonts)：

1. 将工程下载下来后cd到install.sh文件所在目录。
2. 执行`./install.sh`指令安装所有Powerline字体。
3. 安装完字体库之后，把iTerm 2的设置里的Profile中的Text 选项卡中里的Font设置成 Powerline的字体，也就是名称里带有for Powerline的字体：

![img](https://raw.githubusercontent.com/fcamblor/oh-my-zsh-agnoster-fcamblor/master/iterm_set_font.png)

#### 配色方案

1. 安装[配色方案](https://github.com/altercation/solarized)。

   ```shell
   git clone git://github.com/altercation/solarized.git
   ```

进入刚刚下载的工程的solarized/iterm2-colors-solarized 下，直接双击 Solarized Dark.itermcolors 和 Solarized Light.itermcolors 两个文件就可以把配置文件导入到 iTerm2 里。

2. 配置配色方案。

在 iTerm2 里通过load presets选择刚刚安装的配色主题即可 。

#### F12自动呼出终端

这里还要配置一下 F12自动呼出终端：

1. 勾选键盘，将F1、F2等键用作标准功能键：

![img](https://pic4.zhimg.com/50/v2-f45577629fc1e8ba4e62ecb7b8858fe0_hd.jpg)

2. 把键盘-->快捷键-->显示Dashboard F12前面的勾去掉：

![img](https://pic2.zhimg.com/50/v2-972892cb042a2408a67f887a35c09dc9_hd.jpg)

3. 在iTerm2设置里Keys-->HotKey里的东西都勾上：

![img](https://pic3.zhimg.com/50/v2-fe6b266fddfd7f7d3d5f3a105ac182d6_hd.jpg)

#### 安装powerline[可选]

这一步可选，其实我觉得不用powerline还好看一点。安装powerline（项目地址：[Installation - Powerline beta documentation](http://powerline.readthedocs.io/en/latest/installation.html)）：

```shell
pip install powerline-status
```

添加powerline到环境变量：

```shell
# config for powerline
 if [ -d "$HOME/Library/Python/2.7/bin" ]; then
     PATH="$HOME/Library/Python/2.7/bin:$PATH"
 fi
 . /Users/william/Library/Python/2.7/lib/python/site-packages/powerline/bindings/zsh/powerline.zsh
```

![img](https://pic4.zhimg.com/50/v2-71917ac7e264dc568ae08e1294137abd_hd.jpg)

注意.后面有空格，并且要把/powerline/bindings/zsh/powerline.zsh前面的地址替换成你自己的安装地址，通过`pip show powerline-status`可以得到你的powerline安装信息，其中Location后的地址就是你的安装地址，比如我的：

![img](https://pic2.zhimg.com/50/v2-deac13cee5b7ec6a764f4286d57e9afc_hd.jpg)

然后删掉.zcompdump文件再重启终端：

```shell
rm ~/.zcompdump
```

