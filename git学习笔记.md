# git学习笔记

## 配置

### 配置基本信息

/etc/gitconfig文件：

​	系统中对所有用户都普遍适用的配置  使用git config --system 读写的就是这个文件

~/.gitconfig文件：

​	用户目录下的配置文件只适用于该用户 使用 git config --global 读写的就是这个文件

.git/config文件：

​	当前项目的Git目录中的配置文件（也就是工作目录中的.git/config文件） 这里的配置仅仅针对当前项目有效

例子：

配置个人的用户名和电子邮件地址

git config --global user.name "linzi"

git config --global user.email "linzi_jxw@163.com"

查看已有配置

git config --list



基础命令：

​	git init 初始化仓库

​	初始化后的目录结构

![初始化后的目录结构](git学习笔记.assets/image-20201102140113252.png)

### 配置命令别名

```shell
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
```





## git底层操作

### 区域

#### 工作区

#### 暂存区

#### 版本库



### 对象

#### Git对象

​		Git的核心部分是一个简单的键值对数据库。你可以向该数据库插入任意类型的内容，它会返回一个键值，通过该键值可以在任意时刻再次检索该内容。git对象是key-value组成的键值对（key是val对应的hash）键值对在git内部是一个blob类型

​	向数据库写入内容 并返回对应键值对

```shell
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
	-w 选项指示 hash-object命令存储数据对象；若不指定此选项，则该命令仅返回对应的键值
	--stdin 选项指示该命令从标准输入读取内容；若不指定此选项，则须在命令尾部给出待存储文件的路径
	返回：该命令输出一个长度为40个字符的校验和 这是一个SHA-A哈希值
	
$ find .git/objects/ -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
	查看如果存储该内容的
	
JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (master)
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
	根据键值拉取数据
	-p 选项可指示该命令自动判断内容的类型，并为我们现实格式友好的内容
	-t 输出该内容的类型
	利用cat-file -t命令可以让Git告诉我们其内部存储的任何对象类型
	返回：blob
	
```

​	

#### 树对象

​	git对象存在一个问题 记住文件的每一个版本所对应的SHA-1值并不现实 在Git中，文件名并没有被保存仅保存了文件的内容 解决方案：树对象

​	树对象（tree object)，它能解决文件名保存的问题，也允许我们将多个文件组织到一起 Git以一种类似于UNIX文件系统的方式存储内容。所有内容以树对象和数据对象（git对象）的形式存储，其中树对象对应了UNIX中的目录项，数据对象(git对象)则大致上对应文件内容。一个树对象包含了一条或多条记录（每条记录含有一个指向git对象或者子树对象的SHA-1指针，以及相应的模式、类型、文件名信息）一个树对象也可以包含另一个树对象。

查看树对象

命令：

​	git cat-file -p master^{tree}

​	master^{tree}语法表示master分支上最新的提交所指向的树对象

#### 构建树对象

我们可以通过update-index;write-tree;read-tree等命令来构建树对象并塞入暂存区

```shell
JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (master)
$ echo "Hello Git" > test.txt

JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (master)
$ git hash-object -w test.txt
warning: LF will be replaced by CRLF in test.txt.
The file will have its original line endings in your working directory
9f4d96d5b00d98959ea9960f069585ce42b1349a

JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (master)
$ git update-index --add --cacheinfo 100644 9f4d96d5b00d98959ea9960f069585ce42b1349a test.txt

JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (master)
$ git write-tree 给暂存区做一个快照 生成一个树对象 放到数据库里面进去
43bd1cff5fe2dcc90c3c0b4c66c9a5c19175e617

文件模式为 	100644 普通文件
		   100755 可执行文件
           120000 符号链接 
--add选项
	因为此前文件并不在暂存区中首次需要 --add
--cacheinfo选项
	因为将要添加的文件位于git数据库中，而不是位于当前目录下 所以需要--cacheinfo 
	

查看暂存区当前的样子
	git ls-files -s

git update-index --add new.txt
	将合并 git hash-object -w new.txt 和
	git update-index --cacheinfo xxxhashcode new.txt 
	
将第一个树对象加入第二个树对象，使其成为新的树对象
命令：
	git read-tree --prefix=bak 43bd1cff5fe2dcc90c3c0b4c66c9a5c19175e617
	git write-tree
	read-tree命令 可以把树对象读入暂存区

```

#### 提交对象

我们可以通过调用commit-tree命令创建一个提交对象，为此需要制定一个树对象的SHA-1值，以及该提交的父提交对象（如果有的话 第一次将暂存区做快照就没有父对象）

创建提交对象

```shell
echo 'first commit' | git commit-tree 43bd1cff5fe2dcc90c3c0b4c66c9a5c19175e617
```

## Git高层操作

git init



`git add filename`  将新文件添加到暂存区

`git add ./`				将当前修改添加到暂存区

`git commit -m` "版本的注释信息"



工作目录下面的所有文件两种状态：**已跟踪** 或 **未跟踪**

已跟踪的文件是指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，它们的状态可能是 **已提交** **已修改** **已暂存**

![文件状态生命周期](git学习笔记.assets/image-20201102181942156.png)

检查当前文件状态

`git status `



## 命令总结

### 通用命令

git init				初始化版本库

git status			查看工作状态

git diff				查看当前做的哪些更新还没有暂存 工作区中的与暂存区的做对比

git diff --cached

git diff -staged	有哪些更新已经暂存起来准备好了下次提交 暂存区中的与版本库的做对比



#### 删除

1. 工作区先 rm 文件
2. git add ./ 将修改提交暂存区
3. git commit -m 完成提交
4. 或者直接 git rm(省略 1 和2)

#### 改名

1. git mv old.txt new.txt

       git mv 相当于运行了下面三条命令

    1.  mv old.txt new.txt
    2. git rm old.txt
    3. git add new.txt

2. git commit -m

    

#### 查看历史记录

```shell
JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (master)
$ git log
commit b6c1cdf865b454a889472547e88d68e8fc1bbd3d (HEAD -> master)
Author: linzi <linzi_jxw@163.com>
Date:   Mon Nov 2 21:20:02 2020 +0800

    add new.txt

commit e23400a229cd75d3cf2ebd440ddd55f8b6e472e1
Author: linzi <linzi_jxw@163.com>
Date:   Mon Nov 2 18:40:03 2020 +0800

    first commit

commit f64326ecf7316e515cef1549d523d07e20a035bf
Author: linzi <linzi_jxw@163.com>
Date:   Mon Nov 2 18:39:14 2020 +0800

    first commit

```

git log 会按提交时间列出所有的更新，最近的排在最上面

git log --pretty=oneline

git log --oneline		与上一样大致一样 除了 SHA-1显示变短

#### 查看更多历史记录

```shell
JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (testing)
$ git reflog
95ba5bc (HEAD -> testing) HEAD@{0}: checkout: moving from master to testing
b6c1cdf (master) HEAD@{1}: checkout: moving from testing to master
95ba5bc (HEAD -> testing) HEAD@{2}: commit: add c.txt
7782914 HEAD@{3}: checkout: moving from master to testing
b6c1cdf (master) HEAD@{4}: checkout: moving from testing to master
7782914 HEAD@{5}: checkout: moving from master to testing
b6c1cdf (master) HEAD@{6}: checkout: moving from testing to master
7782914 HEAD@{7}: commit: add test.txt
b6c1cdf (master) HEAD@{8}: checkout: moving from master to testing
b6c1cdf (master) HEAD@{9}: commit: add new.txt
e23400a HEAD@{10}: commit: first commit
f64326e HEAD@{11}: commit (initial): first commit

```





### 工作区 《--》暂存区

git rm --cached <file>		 将文件从暂存区中移除 （新文件）

### untracked 《--》unmodified

git add <file>							添加文件到暂存区

git restore --staged new.txt		unstaged文件

### unmodified 《--》modified

git add <file>					

git checkout -- <file> 		将工作区中的修改丢弃

git restore <file>					to discard changes in working directory

git diff										查看当前做的哪些更新还没有暂存

### modified 《--》staged

git restore --staged new.txt		将staged的内容取消缓存 但内容还在文件中

### staged 《--》commited

git commit -a -m "注释信息" <file> 	将工作区中修改的文件直接提交 省略加入到暂存区的步骤

### modifi 《--》commited

git commit -a -m "注释信息"		跳过了暂存阶段

## 分支

使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。在很多版本控制系统中，这是一个略微低效的过程——常常需要完全创建一个源代码目录的副本。对于大项目来说，这样的过程会耗费很多时间。而Git的分支模型极其的高效轻量的，是Git的必杀特性。

比如上份工作（同时有多个功能需要开发，往往需要开发完一个功能才能进行下一个功能的开发 不能穿插着来）git开发过程中，通常一个功能开一个分支

### 创建分支

git branch 分支名

​	为你创建了一个可以移动的新的指针，这会在当前所在的提交对象上创建一个指针

​	`git branch testing`

### 创建并切换分支

`git checkout -b testing`创建并切换到testing分支

### 查询所有分支

​	`git branch`

```shell
JXW@DESKTOP-7ENRA1S MINGW64 ~/Desktop/LearnGit (master)
$ git branch
* master
  testing
* 表示当前在哪个分支
```



### 切换分支

​	`git checkout testing`

​	最佳实践：每次切换分支之前 当前分支一定得是干净的（已提交状态）

​	坑：	

		1. 比如 testing 分支下创建c.txt 但是未追踪 这个时候切换到master 此时c.txt文件还是会存在master分支的目录中，此时c.txt不应该出现在主分支目录下
  		2. testing分支已暂存c.txt（从未提交过）但是未提交 此时切换master分支，那么master分支的暂存区也会有c.txt文件

​	注意：切换分支会修改 HEAD 暂存区 工作目录 版本库不会动 只会无限增多

3. testing分支已经提交过c.txt 此时c.txt被修改过 如果这时切换master分支，不被允许切换

	### 删除分支

​	先切换到其他分支再删除 `git branch -d 分支名` 如果分支没有被merge 那么会提示使用 `git branch -D 分支`强制删除

### 查看每一个分支的最后一次提交

​	`git branch -v`

### 新建一个分支并且使分支指向对应的提交对象

​	`git branch name commitHash`

### 查看项目分叉历史 (分支图)

​	`git log --oneline --decorate --graph --all`

​	命令太长 可以配置别名：`git config --global alias.laofu "log --oneline --decorate --graph --all"`

### 查看哪些分支已经合并到当前分支

​	`git branch --merged`

在这个列表中分支名字前没有*号的分支通常可以使用 git branch -d 删除掉

### 查看所有包含未合并的分支

`git branch --no-merged`尝试使用 git branch -d命令删除在这个列表中的分支时会失败。如果真的想要删除分支并丢弃那些工作，可以使用-D选项强制删除它

### 查看当前分支所指向对象

`git log --oneline --decorate`

`git branch`

### 合并分支

​	`git merge hotfix` 在合并的时候需要先切换到 `git checkout master` 然后再 `git merge hotfix`

​	再合并的时候，有时候会出现“快进（fast-forward）”这个词。由于当前master分支所指向的提交是你当前提交的直接上游，所以Git只是简单的将指针向前移动。换句话，当你试图合并两个分支时，如果顺着一个分支走下去能够达到另一个分支，那么Git在合并两者的时候，只会简单的将指针向前推进，因为这种情况下的合并操作没有需要解决的分歧。

当出现冲突时 解决掉冲突，然后git add ./ 最后再commit就可以



### Git存储

有时，当你的项目的在一个分支上工作了一段时间，这时有需要切换到另外一个分支做点事情。往往需要先将当前分支的修改进行一次提交。针对这个问题我们可以使用Git stash命令

```shell
git stash			命令会将未完成的修改保存在一个栈上，而你可以在任何时候重新应用这些改动（git stash apply）
git stash list		查看存储
git stash apply stash@{2}	如果不指定一个储藏，Git认为指定的是最近的储藏
git stash pop		来应用储藏然后立即从栈上扔掉它
git stash drop		加上将要移除的储藏的名字来移除它
```



## 撤销与重置

### 工作区

​	撤回工作目录中的修改

​	`git checkout -- <file>...`

### 暂存区

​	将暂存区中的修改unstaged

​	`git reset HEAD <file>`

​	`git restore --staged new.txt`

### 版本库

​	提交撤回版本库不会删提交对象 只是在日志中看不到log

 1. 注释内容写错了 `git commit --amend` 可以修改上次提交的注释

 2. 撤销到当前提交，上次提交的内容还保留在暂存区中 `git reset --soft HEAD^`

 3. 工作区、暂存区和版本库都撤销到上次提交 意味着上次提交将丢失 `git reset --hard HEAD^`

    注意：--hard是reset命令唯一的危险用法，它也是Git会真正销毁数据的仅有的几个操作之一。其他形式的reset调用都可以轻松销毁，但是--hard选项不能，因为它强制覆盖了工作目录中的文件



## 团队协作

github上创建远程仓库 不要勾选init

1. create a new repository on the command line

    1. git init

    2. git add README.md

    3. git commit -m "first commit"

    4. git remote add origin https://......url

    5. git push -u origin master
2. 
    push an existing repository from the command line     

    1. git remote add origin https://....url

    2. git push -u origin master

        

### 为远程仓库配置别名

`git remote add <shortname> <url>`

​	添加一个新的远程Git仓库，同时指定一个可以轻松引用的简写

`git remote -v `

​	显示远程仓库使用的Git别名与其对应的URL

`git remote show <remote-name>`

​	查看远程仓库更多信息

`git remote rename <oldname> <newname>`

​	重命名

`git remote rm <remote-name>`

​	如果因为一些原因想要移除一个远程仓库

	1. 从服务器上搬走了或者不再想使用某一个特定的镜像了
 	2. 某一个贡献者不再贡献了

### 推送本地项目到远程仓库

`git push <remote-name> <branch-name>`

### 拉取远程仓库数据

`git fetch <remote-name>` 这个命令会访问远程仓库，从中拉取所有还没有的数据。执行完成后，将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看

​	注意：git fetch 命令会将数据拉取到你的本地仓库 它并不会自动合并或修改你当前的分支 当准备好时必须手动将其合并

​	比如远程仓库别名是taobao   git fetch 后 会有一个远程跟踪分支 taobao/master fetch下载下来的东西在这个分支 随后需要使用git merge进行合并到master分支上

### 克隆远程仓库

`git clone url`

默认克隆时为远程仓库起的别名为origin  如果运行`git clone -o taobao url` 那么该远程仓库别名为taobao