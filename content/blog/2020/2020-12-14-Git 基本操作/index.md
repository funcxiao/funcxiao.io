+++
title = "Git 基本操作"
description = "git常用的基础操作记录"
draft = false
[taxonomies]
tags = ["git"]
[extra]
feature_image = "overview.png"
feature = true
link = ""
+++

![Git](git.jpg)

Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

## What is Git?

Git是一个版本管理工具，开源、免费、敏捷高效，源于`Linus Torvalds`为了管理Linux内核代码而开发的版本管理软件。

## How to install and config?

```shell
$yay -S git # 安装git，不同的系统大致相同，或者直接下载安装
$git config -l # 查看git config
$git config --global init.defaultbranch=main #出于世界潮流，现在github默认分支已经是main,本地设置一下方便统一
$ls ~/.ssh/    # 确认是否有SSH密钥
$ssh-keygen -o # 没有就使用ssh-keygen生成
$cat ~/.ssh/id_rsa.pub # 把密钥复制到对应的网站或者其他配置的地方就不用再登录了
```

现在Windows系统下的`core.autocrlf`默认状态是true，Linux系统下的默认状态是false,所以正常情况下是不需要设置的，如果跨平台使用出现问题，可以调整设置，选项有`true、false、input`

## How to use？

### basic usage

```shell
$git init               # git初始化项目
$git add -A             # git add 来添加文件到git中，-A添加所有
$git commit -m "xxxx"   # git commit -m 提交
$git remote add origin git@github.com:xxx/xxx.git # 添加远端仓库
$git push -u origin main # 推送到远端仓库
```

关于`git commit`规范，广为人知的就是[`Angular Commit Message Guidelines`](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines),进而衍生出了[`conventional-commits`](https://github.com/conventional-commits)。
如果只是追求最简单的用法，直接使用[`commitizen/cz-cli`](https://github.com/commitizen/cz-cli)即可:

```shell
npm install -g commitizen cz-conventional-changelog
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
```

### advanced usage

[查看提交历史](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)

```shell
$git log -n  #查看n条
$git log -n <filename>  #查看指定文件最近n条版本信息
$git log --pretty=format:"%h - %an, %ar : %s" #指定格式:简写哈希值，作者，修订时间，提交说明
$git log --pretty=oneline #仅显示 SHA-1 校验和所有 40 个字符中的前几个字符
$git log --graph #在日志旁以 ASCII 图形显示分支与合并历史
```

[回退版本](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%92%A4%E6%B6%88%E6%93%8D%E4%BD%9C)

```shell
$git reset --hard HEAD^~2 #回滚到前两个版本
$git reset --hard <commit-id>  #回滚到指定版本号，可以只写前几位，git会自动寻找匹配的版本号
$git reset HEAD filename #取消指定暂存文件(git add的文件)
$git checkout --hard <commit-id> <filename> #撤销文件修改，指定文件回退到指定版本
$git reflog #查看所有HEAD操作记录，可以查看恢复reset操作
```

[创建、合并分支](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)

```shell
$git branch dev #创建dev分支
$git checkout dev #切换到dev分支
$git checkout -b hotfix #创建并切换到hotfix分支
$git checkout dev && git merge hotfix #切换并合并hotfix分支
$git branch -d hotfix #删除已经合并的hotfix分支
$git checkout -b testing <commit-id> #从指定版本创建testing分支 
```

远程

```shell
$git remote add origin xxx #添加远程仓库
$git remote -v #查看远程仓库信息
$git remote rm origin #删除添加的远程仓库
$git push -u origin mian #推送到远程分支
```

[变基](https://git-scm.com/book/zh/v2/Git-分支-变基)
变基(rebase)和合并(merge)功能类似，都是用来整合分支，变基适合用来向远程分支推送时让提交历史更加整洁

```shell
$git checkout experiment
$git rebase master #通过变基实现不同分支之间的合并
$git pull --rebase #解决本地与远端同一分支提交历史不一致
```

[子模块](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97#_git_submodules)

```shell
$git submodule add https://github.com/chaconinc/DbConnector #添加子模块,会创建 .gitmoudle 文件
$git submodule init #用来初始化本地配置文件
$git submodule update #从该项目中抓取所有数据并检出父项目中列出的合适的提交
$git clone --recurse-submodules xxx #克隆时初始化并更新子模块
$git submodule update --init #初始化并更新
$git submodule update --init --recursive #初始化、抓取并检出任何嵌套的子模块
```

## GUI

Git有很多GUI工具可以使用，这里有一份[GUI Clients](https://git-scm.com/downloads/guis)列表,当然常用就那几个：

- [SourceTree](https://www.sourcetreeapp.com/)
- [GitHub Desktop](https://desktop.github.com/)

其实个人感觉`vscode`加上那几个git扩展已经够用了

## git server

目前常用的选择有[`GitLab`](https://docs.gitlab.com/omnibus/README.html)和[`Gogs`](https://gogs.io)两种：

- `GitLab`是使用ruby开发，功能大而全，生态也比较好，不过对机器配置要求不低(emmm,反正aliyun之类的最低配的主机是不太能跑得动的)
- `Gogs`是采用golang开发，功能没有前者完善，不过主打的就是轻量级，低配机器甚至是树霉派跑起来都是无压力的

## CI/CD

参考`gitops`

## reference

- [Pro Git](https://git-scm.com/book)
- [净化Git之rebase变基的使用](https://www.cnblogs.com/sunsky303/p/12851267.html)
