[TOC]

### Git

## 版本控制工具

我们学习的东西，一定是当下最流行的！

主流的版本控制器有如下这些：

- **Git**
- **SVN**（Subversion）

### 版本控制分类

**1、本地版本控制**

记录文件每次的更新，可以对每个版本做一个快照，或是记录补丁文件，适合个人用，如RCS。

![image-20210706100126484](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706100126484.png)

**2、集中版本控制  SVN**

所有的版本数据都保存在服务器上，协同开发者从服务器上同步更新或上传自己的修改

![image-20210706100148440](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706100148440.png)

**3、分布式版本控制 	Git**

每个人都拥有全部的代码！安全隐患！

所有版本信息仓库全部同步到本地的每个用户，这样就可以在本地查看所有版本历史，可以离线在本地提交，只需在连网时push到相应的服务器或其他用户那里。由于每个用户那里保存的都是所有的版本数据，只要有一个用户的设备没有问题就可以恢复所有的数据，但这增加了本地存储空间的占用。

不会因为服务器损坏或者网络问题，造成不能工作的情况！

![image-20210706100208508](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706100208508.png)

### Git与SVN的主要区别

SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而工作的时候，用的都是自己的电脑，所以首先要从中央服务器得到最新的版本，然后工作，完成工作后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，对网络带宽要求较高。

Git是分布式版本控制系统，没有中央服务器，每个人的电脑就是一个完整的版本库，工作的时候不需要联网了，因为版本都在自己电脑上。协同的方法是这样的：比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。Git可以直接看到更新了哪些代码和文件！

**Git是目前世界上最先进的分布式版本控制系统。**

## Git环境配置

### 软件下载

打开 [git官网](https://git-scm.com/)，下载git对应操作系统的版本。

### 启动Git

安装成功后在开始菜单中会有Git项，菜单下有3个程序：任意文件夹下右键也可以看到对应的程序！

![image-20210706100341823](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706100341823.png)

**Git Bash： **Unix与Linux风格的命令行，使用最多，推荐最多

Git CMD： Windows风格的命令行

Git GUI：图形界面的Git，不建议初学者使用，尽量先熟悉常用命令

### 常用的Linux命令

![image-20210706100448675](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706100448675.png)

### Git配置

所有的配置文件，其实都保存在本地！

```shell
#查看所有配置 
git config -l
```

![image-20210706102216325](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706102216325.png)

查看不同级别的配置文件：

```shell
#查看系统config
git config --system --list　　
#查看当前用户（global）配置
git config --global  --list
```

**Git相关的配置文件所在dir：**

1）、Git\etc\gitconfig  ：Git 安装目录下的 gitconfig   --system 系统级

![image-20210706101300683](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706101300683.png)

2）、C:\Users\Administrator\ .gitconfig   只适用于当前登录用户的配置  --global 全局![image-20210706102051713](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706102051713.png)

这里可以直接编辑配置文件，通过命令设置后会响应到这里。

### 设置用户名与邮箱（用户标识，必要）

当你安装Git后首先要做的事情是设置你的用户名称和e-mail地址。这是非常重要的，因为每次Git提交都会使用该信息。它被永远的嵌入到了你的提交中：

```shell
git config --global user.name "xxx"  #名称
git config --global user.email 123@qq.com   #邮箱
```

只需要做一次这个设置，如果你传递了--global 选项，因为Git将总是会使用该信息来处理你在系统中所做的一切操作。如果你希望在一个特定的项目中使用不同的名称或e-mail地址，你可以在该项目中运行该命令而不要--global选项。总之--global为全局配置，不加为某个项目的特定配置。

## Git基本理论⭐

### 工作区域

Git本地有三个工作区域：工作目录（Working Directory）、暂存区(Stage/Index)、资源库(Repository或Git Directory)。如果在加上远程的git仓库(Remote Directory)就可以分为四个工作区域。文件在这四个区域之间的转换关系如下：

![image-20210706102524475](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706102524475.png)

- Workspace：工作区，就是你平时存放项目代码的地方
- Index / Stage：暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
- Repository：仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
- Remote：远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换

本地的三个区域确切的说应该是git仓库中HEAD指向的版本：

![image-20210706103153658](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706103153658.png)

- Directory：使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。
- WorkSpace：需要通过Git进行版本控制的目录和文件，这些目录和文件组成了工作空间。
- .git：隐藏文件夹 。存放Git管理信息的目录，初始化仓库的时候自动创建。
- Index/Stage：暂存区，或者叫待提交更新区，在提交进入repo之前，我们可以把所有的更新放在暂存区。
- Local Repo：本地仓库，一个存放在本地的版本库；HEAD会只是当前的开发分支（branch）。
- Stash：隐藏，是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

### 工作流程

git的工作流程一般是这样的：

１、在工作目录中添加、修改文件；

２、将需要进行版本管理的文件放入暂存区域；

３、将暂存区域的文件提交到git仓库。

因此，git管理的文件有三种状态：已修改（modified）,已暂存（staged）,已提交(committed)

![image-20210706103703458](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706103703458.png)

## Git项目搭建

### 创建工作目录与常用指令

工作目录（WorkSpace)一般就是你希望Git帮助你管理的文件夹，可以是你项目的目录，也可以是一个空目录，建议不要有中文。

日常使用只要记住下图6个命令：![image-20210706103858505](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706103858505.png)

### 本地仓库搭建

创建本地仓库的方法有两种：一种是创建全新的仓库，另一种是克隆远程仓库。

1、创建全新的仓库，需要用GIT管理的项目的根目录执行：

```shell
# 在当前目录新建一个Git代码库
$ git init
```

![image-20210706104513364](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706104513364.png)

2、执行后可以看到，仅仅在项目目录多出了一个.git目录，关于版本等的所有信息都在这个目录里面。

![image-20210706104526990](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706104526990.png)

### 克隆远程仓库

1、另一种方式是克隆远程目录，由于是将远程服务器上的仓库完全镜像一份至本地！

``` shell
# 克隆一个项目和它的整个代码历史(版本信息)
$ git clone [url]  
# https://gitee.com/kuangstudy/openclass.git
```

2、去 gitee 或者 github 上克隆一个测试！

## Git文件操作

### 文件的四种状态

版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在还不想提交的文件，或者要提交的文件没提交上。

- Untracked: 未跟踪, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过git add 状态变为Staged.
- Unmodify: 文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为Modified. 如果使用git rm移出版本库, 则成为Untracked文件
- Modified: 文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过git add可进入暂存staged状态, 使用git checkout 则丢弃修改过, 返回到unmodify状态, 这个git checkout即从库中取出文件, 覆盖当前修改 !
- Staged: 暂存状态. 执行git commit则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为Unmodify状态. 执行git reset HEAD filename取消暂存, 文件状态为Modified

### 查看文件状态

上面说文件有4种状态，通过如下命令可以查看到文件的状态：

```shell
#查看指定文件状态
git status [filename]
#查看所有文件状态
git status

# git add .                  添加所有文件到暂存区
# git commit -m "注释：消息内容"    提交暂存区中的内容到本地仓库 -m 提交信息 m表示message
```

![image-20210706152928261](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706152928261.png)

![image-20210706152947189](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706152947189.png)

![image-20210706153007821](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706153007821.png)

![image-20210706153548174](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706153548174.png)

![image-20210706153612677](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706153612677.png)

更改sb.txt内容，在之前的基础之上增加一行`水水水水水水水水水水水水水水水水水水水`内容，继续使用`git status`查看。从图中可以看到新修改了但是没有staged就是没有存入暂存区。

![image-20210706230251655](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706230251655.png)

想看一下sb.txt到底改了什么内容，可以使用`git diff sb.txt`

![image-20210706230355099](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706230355099.png)

从这里我们可以详细看到有哪些内容变动了。

清楚了被改动的地方，我们可以放心的提交到版本库了，使用`git add readme.txt`和`git commit -m " +水"`两条命令。

![image-20210706230508188](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706230508188.png)

### 忽略文件

有些时候我们不想把某些文件纳入版本控制中，比如数据库文件，临时文件，设计文件等

在主目录下建立".gitignore"文件，此文件有如下规则：

1. 忽略文件中的空行或以井号（#）开始的行将会被忽略。
2. 可以使用Linux通配符。例如：星号（*）代表任意多个字符，问号（？）代表一个字符，方括号（[abc]）代表可选字符范围，大括号（{string1,string2,...}）代表可选的字符串等。
3. 如果名称的最前面有一个感叹号（!），表示例外规则，将不被忽略。
4. 如果名称的最前面是一个路径分隔符（/），表示要忽略的文件在此目录下，而子目录中的文件不忽略。
5. 如果名称的最后面是一个路径分隔符（/），表示要忽略的是此目录下该名称的子目录，而非文件（默认文件或目录都忽略）。

```shell
#为注释
*.txt        #忽略所有 .txt结尾的文件,这样的话上传就不会被选中！
!lib.txt     #但lib.txt除外
/temp        #仅忽略项目根目录下的temp文件,不包括其它目录temp
build/       #忽略build/目录下的所有文件
doc/*.txt    #会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

### 版本回退

通过上面，我们清楚了怎么修改文件并且再保存了，我们对readme.txt总共已经进行了几次次修改，想要查看一下历史记录该怎么查？

使用命令`git log`查看历史版本记录

![image-20210706231005700](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706231005700.png)

还可以按照每一次版本变更为一行内容进行显示，使用命令`git log --pretty=oneline`

![image-20210706231142839](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706231142839.png)

现在我需要版本回退，想把当前的版本回退到上一个版本，可以使用如下两种命令：

① `git reset --hard HEAD^`回到上一个版本，`git reset --hard HEAD^^`回退到上上一个版本，以此类推；
② 如果回退到前50个版本的话，使用方法①就显得不太明智了，我们可以使用简便命令操作：`git reset --hard HEAD~50`就可以了。

当前readme.txt文件内容如下：（使用Linux中的cat方法可以查看）

![image-20210706231614799](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706231614799.png)

执行`git reset --hard HEAD^`命令回到上一版本，然后再通过cat查看：

![image-20210706231633789](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706231633789.png)

![image-20210706231653222](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706231653222.png)

已经成功回到了上一版本了。

现在又想回到刚才回退之前的最新版本了，但是使用`git log --pretty=oneline`查看的是比当前老的版本，我们只能通过`git reflog`来查看比当前新的版本

![image-20210706231846972](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706231846972.png)

可以通过`git reset --hard 版本号`到达指定版本号

![image-20210706232056857](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706232056857.png)

**小结：** `git add`把文件添加到暂存区 ； `git commit`提交更改到当前分支上；master是一个分支，前面的**HEAD 指向当前所在的分支**，**HEAD 分支随着提交操作自动向前移动**。

### 撤销修改和删除文件

先把sb.txt文件中删的只剩下a，通过命令查看：

![image-20210707205306420](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707205306420.png)

这时候想恢复以前的版本，现在可以采取的操作有：
① 如果知道要删掉的内容，直接可以手动去改掉，然后重新add并commit即可。
② 也可以按照前面版本回退的方法直接恢复到上一个版本。`git reset --hard HEAD^`
这时候我们还有第三种方法，先看一下当前状态：

![image-20210707205447118](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707205447118.png)

红线内告诉我们了说使用`git checkout -- file`可以丢弃工作区的修改：

![image-20210707205543725](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707205543725.png)

解释一下`git checkout -- sb.txt`这个命令：
① sb.txt自从修改之后还没放到暂存区，使用撤销修改就回到了和版本库一模一样的状态了。
② 另一种就是sb.txt已经放入暂存区，接着又作了修改，撤销修改之后就回到了添加暂存区后的状态。
上面撤销就是情况①
对于第②种情况，我们对sb.txt添加一行内容为666,接着git add添加到暂存区后，再添加内容777，想通过撤销命令让其回到暂存区后的状态。如下：

![image-20210707210129571](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707210129571.png)

执行`rm b.txt`删除工作区的sb.txt如下：

![image-20210707210343439](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707210343439.png)

可以打开文件夹看到，sb.txt已经不在工作区了。现在我们只是把sb.txt从工作区中删掉了，但是前面我们已经把当前状态同步到了版本库了。

选择① 执行commit命令，把当前状态提交到版本库，这样当前状态就会同步至版本库，版本库内的sb.txt就会被删掉。

选择② 在我们没有commit之前，从版本库中恢复此文件下来：
可以使用`git checkout -- b.txt`

![image-20210707210727147](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707210727147.png)

## GIT连接GITHUB repo

### 创建SSH Key

1.注册GitHub账号。
2.创建SSH Key。`windows + R`键同时按，打开运行命令窗口，输入`.`进入家目录。

看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果有（那就奇了怪了！），一般第一次使用是没有的，真有的话可以直接跳过下面的命令。

打开命令行，执行命令：`ssh-keygen -t rsa -C " xxxx.com"`邮箱是自己GitHub账号;它会让你选择路径，还会让你设置密码，这里最好全部都按照默认，一路回车下去，如下：

![image-20210706213515401](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706213515401.png)

再次打开家目录，会发现有一个`.ssh`文件夹，如下：

![image-20210706213601112](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210706213601112.png)

`id_rsa`是私钥，不能泄露， `id_rsa.pub`是公钥。

3.登录GitHub，打开`settings`找到`SSH Key`页面，点击`New SSH Key`， 填上标题（任意），同时在Key文本框中粘贴`id_rsa.pub`文件的公钥内容。

点击`Add SSH key`之后会跳转到输入密码的界面，我们需要输入GitHub密码来继续，接下来我们就可以看到我们新加的SSH key了。

### 添加remote repo

我们有本地的Git仓库，又想在GitHub中创建一个Git仓库，并且希望这两个仓库进行远程同步

1.在GitHub上创建一个仓库，在页面右上角`➕`号选择新的仓库（New repository）；

2.填入仓库名称就可以了，直接点创建。这时候一个新的仓库就建立完成了，目前这个仓库还是空的（勾选Initialize ...会创建一个README.md）。

页面提示说，我们可以有多种方法初始化这个仓库，我们按照给出的提示在本地仓库中运行下面的命令：

```shell
git remote add origin https://github.com/username/reponame.git
#这里的https 也可以改成后面的shh链接

git push -u origin master
#第一次-u 后面就不用-u了
```

推送成功。

这里使用的几个命令,其中`git push`命令，实际上是把当前分支master推送到远程。

因为远程库是空的，第一次推送master增加一个`-u`参数，这样Git不仅仅会把本地的master分支的内容推送到远程GitHub仓库中新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送和拉取就可以简化命令了。

刷新GitHub仓库的页面，这样就可以看到GitHub上的仓库和本地一模一样了。还能看到commit版本号。

## 分支管理

### 创建与合并分支

每次提交，Git都把它们串成一条时间线，这条时间线是一个分支。

到现在为止，我们还只有一个master主分支，HEAD严格来说是指向master，所以HEAD指向的就是当前分支
现在我们来创建新的分支：
1.查看所有分支命令`git branch`
2.`git checkout -b 分支名` 创建并切换到这个分支

![image-20210707211213058](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707211213058.png)

`git checkout`命令加上参数`-b`就代表创建之后切换到，相当于两条命令：
`git branch second` 和 `git checkout second` 相当于这两条。
`git branch`查看分支，会列出所有分支，当前分支前面有一个星号，上面用到了。

![image-20210707211335998](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707211335998.png)

### **分支之间的工作**

我们在third分支上，我们查看一下sb.txt的内容，接着添加一行123，再把它添加到版本库。

![image-20210707211709335](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707211709335.png)

在在third分支上完成了提交之后，切换到master分支上查看sb.txt的内容：

![image-20210707211830292](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707211830292.png)

现在我们想要把third分支上增加的内容合并到分支master分支上，可以在master分支上使用如下命令：`git merge third`

![image-20210707212008422](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707212008422.png)

在执行合并那里看到一句`Fast-forword`这是因为执行的模式是`快进模式`，就是直接把master执行了third的当前提交，合并速度快。

现在把third分支删除：
`git branch -d third`

![image-20210707212119956](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707212119956.png)

总结一下**创建与合并命令**：
查看分支`git branch`
创建分支`git branch name`
切换分支`git checkout name`
创建+切换分支`git checkout -b name`
合并某分支到当前分支`git merge name`
删除分支`git branch -d name`

------

下面我们考虑：
如果在second分支上我们修改提交了新内容，在master上也修改提交了新内容，那么我们在合并的时候取谁的呢？
还是一步一步来，
1查看sb.txt的内容
2添加一些新内容
4提交到版本库
如下：

![image-20210707212509340](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707212509340.png)

同样的操作，切换到second分支上修改内容提交：

![image-20210707212829757](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707212829757.png)

在master分支上进行合并second，如下操作：

![image-20210707213241186](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707213241186.png)

可以从上面这一张图可以仔细的分析出，两个分支上都做了新的提交，在合并的时候会有冲突，并且分支名也从master变成了`master | MERGING`， 同时我们`cat sb.txt`发现文件内容也变了，在`git status`上给我们提供了解决方法：`git commit`；
Git使用了`<<<<<<<` ; `========`;`>>>>>>>` 分别标记出不同分支修改的内容，我们可以打开文件修改成和主分支一致，然后在`master | MERGING`这个临时分支上进行`git commit`。

![image-20210707213846196](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707213846196.png)

经过这一系列的操作之后，我们可以通过`git log`查看分支合并情况：

![image-20210707213946303](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707213946303.png)

----------

### 分支管理策略

通常我们合并分支的时候，Git一般是用Fast forward模式，这种模式下，删除分支之后，log中会丢掉分支信息，现在我们来使用`-no-ff`来禁用Fast forward模式。
1创建一个dev分支。
2修改sb.txt内容。
3添加到暂存区。
4切换回second分支。

![image-20210707215339346](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707215339346.png)

 5合并dev分支，删除，使用命令`git merge -no-ff -m "注释内容" dev`
6查看历史记录

![image-20210707215556881](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707215556881.png)

删除dev之后我们发现在最后的log中还有dev的操作。

**分支策略**：master主分支应该是非常稳定的，也是用来发布的新版本，一般情况下干活都不在master分支上干，都是在新建的分支上，干完之后需要发布，或者说新建分支代码稳定之后可以合并到主分支master上。

### Bug 分支

在开发过程中，bug问题是不能避免的，那么有了bug就需要修复，在Git中，每个bug我们都可以通过一个临时分支来修复，修复完成之后合并分支，然后将临时的分支删除掉。

例如我在开发中接到一个404bug的时候，可以临时创建一个404分支来修复。但是我目前在的分支dev的开发工作还没有完成，但是bug需要五个小时内完成，怎么办？这时候Git有一个stash功能，可以将当前工作现场`隐藏到后台`，等以后再恢复现场继续工作。如下：

修改sb.txt，隐藏后status很干净

![image-20210707220610078](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707220610078.png)

接下来我们要知道bug是哪个分支上的，比如bug是master分支上的，我就需要再master分支上重新建一个分支：

![image-20210707221053369](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707221053369.png)

修复好bug之后切换到master分支，并合并，最后删除这个临时分支：

![image-20210707221328104](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707221328104.png)

现在又可以回到second分支上继续干活了，要把我们的现场恢复回来：

通过`git stash list`查看隐藏的工作现场
恢复的方式有两种：
1 `git stash apply`，这种恢复方式恢复后stash内容并不删除，需要使用`git stash drop`来删除。
2 另一种方式是使用`git stash pop`，恢复的同时把stash内容也删除了。

![image-20210707221653422](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707221653422.png)

## 多人协作

当我们从远程库克隆的时候，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且远程库的默认名称是origin。

![image-20210707222943177](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707222943177.png)

### **推送分支**

推送一个分支就是把该分支上所有本地提交到远程库上，推送的时候要制定本地的分支，Git会把该分支推送到远程库对应的远程分支上：
使用命令`git push origin master`

![image-20210707223428007](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707223428007.png)

### **抓取分支**

在进行多人协作时，大家都会往master分支上推送各自的修改，现在我们假设有另外一个同事，通过添加SSH Key添加到GitHub上，或者是同一台电脑上另外一个目录克隆。
先把second分支也push到GitHub上。

在这个目录下模仿另外一个同事也进行克隆这个项目，双方在second分支上做开发。同事已经推送到远程了，这里我也改了一下这个文件，也想要推到远程：

![image-20210707223809140](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707223809140.png)

推送失败的原因是同事最近也提交了，提交到有冲突，上面提示我们，先用git pull把最新的提交抓取下来，然后在本地合并，解决冲突之后再推送。

先要指定本地dy分支与远程origin/dy分支的连接，根据提示设置dy分支和origin/dy的链接，如下：

![image-20210707223941644](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707223941644.png)

我们又看到了熟悉的MERGING临时分支了；
解决方式和之前是一样的，我先看一下内容，然后修改，然后再继续推送到远程中：

![image-20210707224018867](https://gitee.com/ahrunio/pic-go-image-hosting-service/raw/master/img/image-20210707224018867.png)

所以：多人协作的工作模式是：
1可以试图用`git push origin branchname`推送自己的修改
2如果推送失败，是因为远程分支比本地更新早，需要先用`git pull`试图合并。
3如果合并有冲突，需要解决冲突，并在本地提交，使用`git push origin branchname`

## 总结

Git基本常用命令如下：
创建目录`mkdir`
显示当前目录的路径：`pwd`
把当前的目录变成可以管理的Git仓库：`git init`
把xx文件添加到暂存区：`git add xx`
提交文件：`git commit -m "注释"`
查看仓库状态`git status`
查看xx文件修改了哪些内容`git diff xx`
查看历史记录`git log`
回退版本`git reset --hard HEAD^`或者`git reset --hard HEAD~50`
查看文件`cat xx`
查看历史记录的版本号id`git reflog`
把xx文件在工作区的修改全部撤销`git checkout -- xx`
删除xx文件`rm xx`
关联一个远程库`git remote add origin 地址`
把当前master分支推送到远程库`git push -u(第一次要用-u 以后不需要) origin master`
从远程库中克隆`https://github.com/duanmingpy/remote_repo.git`
创建dy分支并切换到`git checkout –b dy`
查看当前所有分支`git branch`
切换回master分支`git checkout master`
在当前的分支上合并dy分支`git merge dy`
删除dy分支`git branch –d dy`
创建分支`git branch name`
把当前的工作隐藏起来，等以后恢复现场后继续工作`git stash`
查看所有被隐藏的文件列表`git stash list`
恢复被隐藏的文件，但是内容不删除`git stash apply`
删除文件`git stash drop`
恢复文件的同时删除文件`git stash drop`
查看远程库的信息`git remote`
查看远程库的详细信息`git remote -v`
把master分支推送到远程库对应的远程分支上`git push origin master`
指定local branch 和 remote branch的分支连接`git branch --set-upstream localbranchname remotebacnchname`
