1.对于已经创建好的gitee仓库项目，向仓库中上传文件

\#进入到需要上传的项目目录，右键选择Git Bash Here

 

`git init `# 第一步：初始化项目，然后会生成.git文件夹

`git add .  `# 第二步：将当前目录加入到git（.代表全部）

`git commit -m "first “`（提交的描述信息也就是提交的说明）# 第三步：git提交到本地版仓库，并注明提交的缘由

`git remote add origin https://gitee.com/hui829/web-vue-project-exercise.git`

\# 第四步：将文件上传到gitee中的master分支,-u代表第一次上传

`git push -u origin master`

注意：一般一个完整项目中只有第一级目录下会生成.git文件夹，如果上传的第二级目录的文件夹中有.git文件夹，则只会上传二级目录文件夹，而不会上传二级目录文件夹下的内容，如果上传的二级目录的文件夹中没有.git文件，则会上传二级目录文件夹下的内容（多级目录同理）

 

2.对于已有的仓库，且已经进行过一次或多次上传文件后，还需要更新代码至gitee

\#进入到需要上传的项目目录，右键选择Git Bash Here



`git status ` # 第一步：列出自己做出修改的文件，会列出你修改的文件以及新增的文件

`git add . `# 第二步：将当前目录加入到git, .代表全部更新, 指定文件更新则点替换为指定文件名

`git add test.txt `# 第二步-2：上传单个文件

`git commit -m `"更新说明" # 第三步： 添加更新说明

`git push origin master`  # 第四步： 执行更新操作



3.从gitee上面克隆项目到本地

进入到需要将克隆项目存放的目录，右键选择Git Bash Here



`git clone https://gitee.com/gavinzhulei/vue-form-making.git `# xxx 代表g项目的克隆地址,仅需一步

![img](E:\Project\Textbook\assets\wps1.jpg) 

问题原因：在码云创建的仓库有ReadMe文件，而本地没有，造成本地和远程的不同步

报错解决：**one ：**

本地没有ReadMe文件，那么就在本地生成一个：

`**git pull --rebase origin master** `  本地生成ReadMe文件

`**git push origin master**`

 

**two：**

**`git push -f origin master ` #**  那我就强制上传覆盖远程文件，

(这个命令在团队开发的时候最好不要用,否则可能会有生命危险)

 

 

[1] Git技术:公司必备，-定要会

[2] Git概念:Git是一个免费的、开源的分布式版本控制系统，可以快速高效地处理从小型到大型的项目。

[3]什么是版本控制?

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。

[4]为什么要使用版本控制?

软件开发中采用版本控制系统是个明智的选择。

有了它你就可以将某个文件回溯到之前的状态,甚至将整个项目都回退到过去某个时间点的状态。

就算你乱来一气把整个项目中的文件改的改删的删，你也照样可以轻松恢复到原先的样子。

但额外增加的工作量却微乎其微。你可以比较文件的变化细节,查出最后是谁修改了哪个地方，从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等。

[5]版本控制系统的分类:

集中化的版本控制系统:

![img](E:\Project\Textbook\assets\wps2.jpg) 

集中化的版本控制系统诸如CVS, SVN以及Perforce等，都有一个单-的集中管理的服务器,保存所有文件的修订版本, 而协同工作的人们都通过客户端连到这台服务器,取出最新的文件或者提交更新。多年以来,这已成为版本控制系统的标准做法,这种做法带来了许多好处,现在,每个人都可以在定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统;要远比在各 个客户端上维护本地数据库来得轻松容易。

事分两面，有好有坏。这么做最显而易见的缺点是中央服务器的单点故障。如果服务器宕机- -小时， 那么在这一-小时内， 谁都无法提交更新，也就

无法协同工作。

 

 

 

密分布式的版本控制系统

由于上面集中化版本控制系统的那些缺点，于是分布式版本控制系统面世了。

在这类系统中，像Git, BitKeeper等客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。

![img](E:\Project\Textbook\assets\wps3.jpg) 

 

分布式的版本控制系统在管理项目时存放的不是项目版本与版本之间的差异.它存的是索引(所需磁盘空间很少所以每个客户端都可以放下整个项目的历史记录)



<img src="E:\Project\Textbook\assets\wps4.jpg" alt="img" style="zoom:50%;" /> 

[1]代码托管中心是干嘛的呢?

我们已经有了本地库，本地库可以帮我们进行版本控制，为什么还需要代码托管中心呢?

它的任务是帮我们维护远程库,

下面说一 下本地库和远程库的交互方式，也分为两种: .

(1) 团队内部协作

![img](E:\Project\Textbook\assets\wps5.jpg) 

(2) 跨团队协作

![img](E:\Project\Textbook\assets\wps6.jpg) 

[2]托管中心种类:

局域网环境下:可以搭建 GitLab服务器作为代码托管中心，GitLab可以自己去搭建

外网环境下:可以由GitHub或者Gitee作为代码托管中心，GitHub或者Gitee是现成的托管中心，不用自己去搭建

 

## 常用命令：

`  git init  `// 初始化版本库，cd /d/git先到指定目录，生成.git目录是隐藏的

`  git add .  `// 添加文件到版本库（只是添加到缓存区），.代表添加文件夹下所有文件

\###修改后的文件需重新git add

`  git commit -m "v-1" 文件	`// 把添加的文件提交到版本库，-m填写提交备注

\###不管文件在哪，必须通过add,commit命令操作才可以将内容提交到本地库。

\###修改后的文件需重新git commit



`git status `// 查看工作区状态  ，

\###报红色，工作区的文件，未被git add的文件

\###报绿色，暂存区的文件，可被git commit提交到本地库的文件



`git config `//配置

`--list `看所有用户

`--global user.name "liu" user.email "2581210093@qq.com" ` \\\ 设置全局用户名liu和e-mail邮箱



`git log` //查看commit的日志，下一页：空格、上一页：b、退出：q

` --oneline `//一行简短进行展示

`-pretty=oneline `//美丽的方式：一行进行展示  

![img](E:\Project\Textbook\assets\wps7.jpg) 

`git reflog` //分支等引用变更记录管理

{2}：指回到这个历史版本需要走多少步

![img](E:\Project\Textbook\assets\wps8.jpg) 

`git reset --hard 53b88b0` //跳到指定索引位置，

\###--hard本地库的指针移动的同时，重置暂存区，重置工作区

\###--mix本地库的指针移动的同时，重置暂存区，但工作区不动

\###--soft本地库的指针移动，但暂存区、工作区都不动

 

`git diff`  //比较工作区和暂存区，所有文件的差异

+文件名：单个文件的比对

+版本索引 文件名：比较暂存区和文件库，该文件的差异

![img](E:\Project\Textbook\assets\wps9.jpg) 

 

【1】什么是分支:

在版本控制过程中，使用多条线同时推进多个任务。这里面说的多条线，就是多个分支。

【2】通过一张图展示分支:

![img](E:\Project\Textbook\assets\wps10.jpg) 

【3】分支的好处:

同时多个分支可以并行开发，互相不耽误，互相不影响，提高开发效率如果有一个分支

`git branch ` //创建主分支

+分支名：创建副分支

`-a ` //查看本地和远程所有分支

-d 分支名  // 删除本地分支  *删除前记得切换到别的分支。*-D 强制删除

`git branch  -v ` //查看分支

`git checkout 分支名`  //切换分支

`-b 分支名 `// 创建并切换到分支 

 

`  git merge ` //合并，合并某个分支到当前分支下，并自动进行新的提交

  `git merge --abort ` //当我们使用git merge操作合并代码但还没add时，若想取消这次合并就用这个

 

## 远程操作：

`git remote add 别名 git@github.com:username/Hello-World.git ` //为远程库起别名

`git push 远程库 本地分支 ` //推送本地指定分支到指定远程库的指定分支上

`--delete 分支名 ` // 上传，删除远程分支

![img](E:\Project\Textbook\assets\wps11.jpg) 

`git pull   `// 拉取，下拉指定主机的指定分支，并与本地的指定分支合并

\##pull = fetch + merge 操作的合并

`git pull 远程库 远程分支 --allow-unrelated-histories ` //允许不相关历史合并

 

`git clone 地址 ` // 克隆，其作用是将存储库克隆到新目录中

`--branch 分支名 ` //克隆指定分支代码到本地

 

`git fetch 远程仓库 分支 ` //获取远程版本库的提交

 

 

## 协同开发：

邀请其他成员，并能push操作

![img](E:\Project\Textbook\assets\wps12.jpg) 

 

 

 

 

 

 

 

## 跨团队合作

![img](E:\Project\Textbook\assets\wps13.jpg) 

 

pull request请求按钮：

![img](E:\Project\Textbook\assets\wps14.jpg) 

 

 

## 免密操作

**第一种操作：**

`git config --global credential.helper store`

执行之后会在linux用户主目录下的.gitconfig文件中多加 helper = store

[user]

​    name = 用户名

​    email = 邮箱

[credential]

​    helper = store

 

之后cd到项目目录，执行git pull命令，会提示输入账号密码。输完这一次以后就不再需要，

并且会在根目录生成一个.git-credentials文件

 

**第二种操作：**

`ssh-keygen -t rsa -C 2581210093@qq.com  `//默认回车

`./ssh/id_rsa.pub`  //复制该文件内容

![img](E:\Project\Textbook\assets\wps15.jpg) 

 

## JetBrains集成系列工具

![img](E:\Project\Textbook\assets\wps16.jpg) 

 

本地库初始化操作：

![img](E:\Project\Textbook\assets\wps17.jpg) 

 

本地库commit提交操作：

![img](E:\Project\Textbook\assets\wps18.jpg) 

 

远程库提交操作：

![img](E:\Project\Textbook\assets\wps19.jpg) 

 

远程clone克隆操作：

![img](E:\Project\Textbook\assets\wps20.jpg) 

 