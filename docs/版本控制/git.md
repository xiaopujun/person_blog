## 基本概念

- **git工作区**：工作区就是写代码的地方

- **暂存区**：存放将要提交的文件（也叫索引区）

- **本地库**：存放本地版本的地方（文件做好准备，向远程的仓库提交）

## git常用命令

git的常用命令可以支持建库，提交，克隆等操作。

### `git init`

初始化一个git本地仓库，很多git命令都需要在本地库中进行，因此这个命令是git的第一个命令。使用此命令后，git会在此目录上生成一个隐藏文件  "
.git"，此隐藏文件就代表了这已经是一个本地库了，并且默认有一个主分支（master）

### `git config`

设置全局用户名邮箱

```git
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱地址"
```

设置本地用户名邮箱（仅对当前项目有效）

```git
git config user.name "你的用户名"
git config user.email "你的邮箱地址"
```

### `git status`

此命令用户显示工作目录和暂存区的状态，可以看到哪些修改操作被暂存了，哪些还没有，哪些文件没有被追踪。

### `git add`

此命令用于将工作区中的文件添加到暂存区中,可以同时添加多个文件

```shell
git add <file1> <file2> ... # 添加单个/多个文件

git add <dir> # 添加指定目录下的所有文件

git add . # 添加当前目录下的所有文件

```

### `git rm`

注意：不添加--cache 参数，则此命令会将文件本身也从文件系统中删除

```shell
git rm <file> # 将文件从暂存区和工作区中删除(文件本身也会删除)

git rm -f <file> # 强制删除已修改且放入暂存区的文件

git rm --cached <file> # 将暂存区中的文件从暂存区中删除（文件本身不删除）

git rm -r <directory> # 删除指定目录下的所有文件


```

- `git commit <file>`：此命令可以将暂存区中的文件，以及日志和版本信息提交到本地仓库中。
- `git commit <file> -m '提交信息'`：此命令可以将暂存区中的文件，以及日志和版本信息提交到本地仓库中。

### `git commit`

```shell
git commit -m "提交信息" #提交暂存区到本地仓库中

git commit [file1] [file2] ... -m "提交信息" #提交指定文件到本地仓库中

git commit -am "提交信息" # 跳过暂存区，直接提交工作区中的文件到本地仓库中

git commit --amend -m "新的提交信息" # 修改之前的提交信息

```

### `git restore`

```shell
git restore <file> # 将指定文件恢复到最新的提交状态，丢弃所有未提交的更改。

git restore --staged <file> 或 git restore -S <file> # 将暂存区中的指定文件恢复到工作目录中的状态，同时从暂存区移除。

```

## GitHub合作开发

### 拉取PR请求

```shell
git fetch <远程仓库别名,通常是opigin> pull/<PR编号>/head:<拉取到本地的分支名>

# 例如
git fetch origin pull/1/head:PR1
```