---
layout: post
title:  git在各操作系统平台下的安装和配置
date:   2016-08-07 19:40:07 +08:00
category: 其他
tags: git 安装部署
comments: true
---

* content
{:toc}


本文是git系列博客的第一篇，主要介绍git在windows,linux,Mac OX等不同操作系统下的安装和配置，主要以后两者为主。






## 工具下载

- **ubuntu**:`sudo apt-get intall git` 安装
- **windows**:下载[git for windows](https://git-for-windows.github.io/)安装即可
- **mac**:个人对[homebrew](http://brew.sh/)不是很安心,建议安装[macports](https://www.macports.org/),再用macports安装git(`sudo port install git +bash_completion`)


## 环境相关配置

- `git config --global user.email "you@example.com"` 配置邮件
- `git config --global user.name "Your Name"` 配置用户名
- `git config --global color.ui true` 开启颜色显示
- 创建SSH Key。在用户主目录下(`~`)，看看有没有`.ssh`目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：`ssh-keygen -t rsa -C "youremail@example.com"`
- 登陆GitHub，打开“Account settings”，“SSH Keys”页面。然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`.ssh`目录下`id_rsa.pub`文件的内容，点“Add Key”

## 命令行显示配置

- 提示语换英语,mac下在.bash_profile里添加下面内容,ubuntu在~/.profile下添加

```
# git language
export LANGUAGE='en_US.UTF-8 git'
```

- 终端显示分支,mac下在.bash_profile里添加下面内容

```
# Git branch in prompt.

parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
export PS1="\u@\h:\w\[\033[32m\]\$(parse_git_branch)\[\033[00m\]$ "
```

参考[Add Git Branch Name to Terminal Prompt (Mac)](http://mfitzp.io/article/add-git-branch-name-to-terminal-prompt-mac/)


## git补全

如果是linux或者windows用户，一般不会出现这个问题，mac下我当时是bash环境没设置好，按照[这篇文章](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137396287703354d8c6c01c904c7d9ff056ae23da865a000)安的，使用的是Command Line Tools安装的git,不能识别macports里的git的补全。有几种办法可以完全补全

- 方法一：直接下载补全文件并使其生效

参考[git auto-complete for *branches* at the command line?](http://apple.stackexchange.com/questions/55875/git-auto-complete-for-branches-at-the-command-line)

1.通过curl下载：`curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash`

2.在`~/.bash_profile`里添加

```shell
if [ -f ~/.git-completion.bash ]; then
  . ~/.git-completion.bash
fi
```

- 方法二：使用macports的bash环境

参考：[How to use bash-completion](https://trac.macports.org/wiki/howto/bash-completion)

检查的下自己的bash:`which bash`,`which -a bash`


- 方法三：使用[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)并启用git plugins

检查下支持的shell:`cat /etc/shells`
查看当前shell:`echo $SHELL`


## 参考资料

>* [Creating a Happy Git Environment on OS X](https://gist.github.com/trey/2722934)
>* [提示语中文改英文](http://askubuntu.com/questions/320661/change-gits-language-to-english-without-changing-the-locale)
>* [Add Git Branch Name to Terminal Prompt (Mac)](http://mfitzp.io/article/add-git-branch-name-to-terminal-prompt-mac/)
>* [git auto-complete for *branches* at the command line?](http://apple.stackexchange.com/questions/55875/git-auto-complete-for-branches-at-the-command-line)
>* [How to configure git bash command line completion?](http://stackoverflow.com/questions/12399002/how-to-configure-git-bash-command-line-completion)
>* [Install Bash git completion](https://github.com/bobthecow/git-flow-completion/wiki/Install-Bash-git-completion)
>* [How to use bash-completion](https://trac.macports.org/wiki/howto/bash-completion)





----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)
