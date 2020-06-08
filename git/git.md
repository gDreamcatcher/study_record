# Git Command

[TOC]

## Git创建新仓库

```shell
### Git global setup
git config --global user.name  "***"
git config --global user.email  "***"

### ...或在命令行上创建一个新的存储库
git clone http://xxx.git
cd vim-plugin
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

### ...或从命令行推送现有空的存储库
cd existing_folder
git init
git remote add origin http://xxx.git
git add .
git commit -m "Initial commit"
git push -u origin master

### ...或从命令行推送已有存储库
cd existing_repo
git remote add origin http://xxx.git
git push -u origin --all
git push -u origin --tags
```



## 修改最新的提交的用户名和邮箱

```shell
# 先设置好用户名和邮箱
git config --local user.name "Author Name"
git config --local user.email email@address.com
# 修改最新提交的用户名和邮箱
git commit --amend --reset-author --no-edit
```

## Git reset

```shell
# 撤回commit操作，代码仍然保留
git reset --soft HEAD^
# 撤回commit操作，代码不保留
git reset --hard HEAD^
# 撤回commit操作，代码保留,撤销add
git reset --mixed HEAD^
# 不撤回commit，修改提交信息
git commit --amend
```


```shell
# 下载指定浅分支
git clone --depth=1 -b tag1.1.1 http://xxx.git
git fetch --unshallow
```



## Git Submodule

```shell
# 添加子模块
git submodule add https://github.com/chaconinc/DbConnector

# 查看子模块的更新
git log --cache --submodule

# 克隆带有子模块的仓库
git clone --recurse-submodules https://github.com/chaconinc/MainProject

# 或者
git clone https://github.com/chaconinc/MainProject
git submodule update --init --recursive

# 带子模块切换仓库
git checkout --recurse-submodules master


# 如果submodule的远程仓库找不到对应的分支
# copy the new URL to your local config
$ git submodule sync --recursive
# update the submodule from the new URL
$ git submodule update --init --recursive

# 更新子模块
git submodule update --remote --merge

# 提交前检查子模块有没有更新没有提交， 如果有的话报错
git push --recurse-submodules=check
```

