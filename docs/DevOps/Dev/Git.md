# Git

：一个开源的、分布式的版本控制工具，常用于软件配置管理（Software Configuration Management，SCM）。

主要原理:
- 用户可以将当前状态的一些文件保存为一个版本，永久保存它们此时的状态。
- 用户可以创建分支，像指针一样指向某个版本，从而对该版本进行操作。
- 单个用户提交的多个版本会按先后顺序排列成一条线。如果对一个历史版本做出几种不同的改动，版本图就会分叉成几条不同的线，需要手动处理它们的差异，合并成一条线。
- 用git对一个目录进行版本管理时，会创建一个 .git 子目录，用于保存该git仓库的所有数据。
  - 比如所有进行版本管理的文件都会保存一个副本，这会导致项目目录的总体积变大。

## 配置

git的配置文件有三种：
- 系统的配置文件：保存在 /etc/gitconfig 。
- 当前用户的配置文件：保存在 ~/.gitconfig 。
- 当前git仓库的配置文件：保存在仓库下的 .git/config 。只作用于当前git仓库，会覆盖上层的配置。

可以在文本编辑器中修改配置文件，也可以使用以下命令进行修改：
```shell
git config 
        --system      # 使用系统的配置文件
        --global      # 使用当前用户的配置文件
        --local       # 使用当前git仓库的配置文件

        -l            # 显示配置文件的全部内容
        -e            # 在文本编辑器中打开配置文件
        
        <key>         # 显示配置文件中某项参数的值
        <key> <value> # 设置配置文件中某项参数的值
```

## 安装

1. 安装git：
    ```shell
    yum install git
    ```
    也可以安装git的GUI工具，比如Tortoisegit。

2. 初始化配置：
    ```shell
    git config --global user.name "name"
    git config --global user.email "you@example.com"
    ```
    每次commit时git都会自动备注提交者的用户名和邮箱。

3. 在某个目录下创建git仓库。
    ```shell
    git init
    ```
    这会创建一个 .git 子目录，还会创建一个默认的master分支。

## 版本

对文件进行版本控制：
```shell
git add <文件名>      # 将文件相比上一版本的改动加入暂存区
git add .             # 将当前目录的所有文件加入暂存区
git rm --cached 1.py  # 从暂存区删除某个文件
```
- 被修改的文件应该先加入暂存区，以便之后提交成一个版本。
- 用 git rm/mv 做出的改动会自动加入暂存区。
- 可以创建一个 .gitignore 文件，记录不想进行版本控制的文件。如下：
```shell
echo 1.py >> .gitignore
```

提交版本：
```shell
git commit 
        -m "initial version" # 将当前的暂存区提交为一个版本，需要加上备注信息
        -a                   # 提交从上一个版本以来被改动的所有文件
        --amend              # 重新提交，用当前的暂存区覆盖上一个版本
```
- 版本是在某个时刻对所有文件的一个快照。
- git会自动取每个版本的SHA-1值作为版本号。

撤销：
```shell
git reset            # 清空暂存区
        --hard       # 重置到当上一个版本（相当于git reset 加git checkout .）
        <版本号>     # 将当前分支回滚到指定版本
            --hard   # 回滚的同时会删除这之间的版本

git revert <版本号>  # 通过新建一个版本来抵消某个历史版本的变化（这样不会删除历史版本）
```

## 分支

- 分支是一个指针，指向某个版本。用户可以任意改变分支指向的版本。
- 创建git仓库后默认会创建一个master分支。
- 当前所处的版本会用“HEAD”标明。
- 创建分支的例子：
  - 创建一个 dev 分支来开发、调试，等确定稳定了才合并到 release 分支。
  - 发现bug时，创建一个临时的 hotfix 分支，修复了bug之后再合并到正式的分支。
- 合并两个分支时，如果两个分支在同一个文件处的内容不同，就会产生冲突，要先解决冲突才能合并。

命令：
```shell
git branch          # 显示所有分支
        -v          # 显示每个分支所在的版本
        <分支名>    # 新建一个分支
        -d <分支名> # 删除一个分支

git checkout
        <分支名>    # 切换到指定分支所在版本
        -b <分支名> # 切换到指定分支，如果该分支不存在则创建它
        -- <文件名> # 将某个文件恢复到上一次add或commit的状态
        .           # 将当前目录的所有文件恢复到上一次add或commit的状态

git merge <分支名>  # 将指定分支合并到当前分支（这会产生一个新版本）
```

## 标签

标签相当于不能移动的分支名，永远指向某个版本，常用于版本发布。

命令：
```shell
git tag                 # 显示已有的所有标签
        -a v1.0 9fceb02 # 给版本9fceb02加上标签v1.0
        -d <tagName>    # 删除一个标签

git checkout <tagName>  # 切换到指定标签所在版本
```

## 变基

如下图，通过rebase方式将C3合并到master时，会先找到C3与C4的共同祖先C2；然后删除C3，将从C2到C3之间的所有变动应用到C4上，生成一个新版本C3'；最后将master分支快进到C3'处。

![](rebase.png)

merge方式与rebase方式最终生成的版本都一样，但是rebase方式会删除次分支，将版本图简化成一条线。

命令：
```shell
git rebae
        <分支名>           # 将当前分支以变基方式合并到指定分支（这会产生一个新版本）
        branch1 branch2   # 将branch2以变基方式合并到branch1
        branch1 branch2 --onto branch3  # 将branch2相对于branch1的变基应用到branch3上
```

## 管理仓库

```shell
git status    # 查看当前git仓库的状态（包括当前的分支名、暂存区内容）

git log       # 在文本阅读器中查看git仓库的日志（按时间倒序显示每个事件）

git diff                # 比较当前仓库与上一次add或commit的差异
        --cached        # 比较上一次add与上一次commit的差异
        branch1         # 比较当前仓库与某个分支的差异
        branch1 branch2 # 比较两个分支的差异
        --stat          # 只显示统计信息
        <文件名>        # 只比较某个文件的差异
```

从git仓库的所有版本中永久性地删除某个文件：
```shell
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch <文件的相对路径>' --prune-empty --tag-name-filter cat -- --all
git push origin --force --all --tags    # 强制推送，覆盖远端仓库
```

## 远端仓库

搭建一个git服务器之后，就可以将本地的git仓库推送到服务器存储，或从服务器拉取git仓库。
- 一个本地仓库可以设置 0 个或任意个远端仓库。
- 通过URL或name可以指定一个远端仓库。
- 将本地仓库推送到远端时，会自动推送所有分支，并让本地分支与远端分支建立一对一的关系（称为跟踪）。
  - 如果已有被跟踪的远端分支，则让本地分支与它合并。（如果发生冲突就不能直接推送）
  - 如果不存在被跟踪的远端分支，则自动创建它。
  - 如果选择强制推送，则相当于清空远端仓库后再上传本地仓库。
  - 默认不会推送标签，要手动推送。

有两种传输方式：
- 基于HTTPS协议：
  - 先在git服务器上创建账号
  - 然后在本机连接到git服务器，输入账号、密码进行认证。
- 基于SSH协议：
  - 先生成一对SSH密钥，将密钥保存在本机的 ~/.ssh/id_rsa 文件中，将公钥保存到git服务器上。
  - 然后在本机连接到git服务器，使用私钥进行认证。

命令：
```shell
git clone <URL>              # 将一个远端仓库克隆到本地（这会在当前目录下创建该仓库目录）
# 此时git会自动将这个远端仓库命名为origin，并让本地的master分支跟踪origin/master分支

git remote                   # 显示已配置的所有远端仓库的名字
        -v                   # 显示各个远端仓库的URL
        show <name>          # 显示某个远端仓库的具体信息
        add <name> <URL>     # 添加一个远端仓库，并设置其名字
        rm <name>            # 删除一个远端仓库
        rename <name> <name> # 重命名一个远端仓库

git pull [name或URL]         # 拉取远端仓库的内容到本地仓库（会自动将被跟踪的远程分支合并到本地分支）

git fetch [name或URL]        # 从远端仓库拉取所有本地仓库没有的内容（不会自动合并，比较安全）

git push [name或URL]         # 推送本地仓库到远端仓库
        --force                   # 强制推送
        --all                     # 推送本地仓库的所有分支
        <标签名>                      # 推送一个标签
        --tags                        # 推送所有标签
```
- 执行git pull、fetch、push时，如果不指定远端仓库，则使用默认的origin仓库。
- 例：推送单个分支
    ```shell
    git push origin master : origin/master # 推送分支master到远端仓库origin，并与远端分支master合并
    git push origin : origin/master        # 推送一个空分支（这会删除指定的远端分支）
    ```