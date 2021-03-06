# 一、本地Git
***
1. 设置你的名字和Email地址 
   
	```
	$ git config --global user.name "Your Name"
	$ git config --global user.email "email@example.com"
	```

	> 注意gitconfig命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置， 
	> 当然也可以对某个仓库指定不同的用户名和Email地址。

2. 通过git init命令把这个目录变成Git可以管理的仓库：

	``` 
	$ git init 
	Initialized empty Git repository in /Users/michael/learngit/.git/
	```

3. 查看仓库当前的状态 
	`$ git status`     

4. 将文件提交到git仓库 
（1）用命令git add告诉Git，把文件添加到仓库：git add <文件名> 
	`$ git add readme.txt`  
（2）第二步，用命令git commit告诉Git，把文件提交到仓库：

	`git commit -m <描述> `
	```
		$ git commit -m "wrote a readme file"  
		[master (root-commit) eaadf4e] wrote a readme file  
		1 file changed, 2 insertions(+)  
		create mode 100644 readme.txt
	```
	可以一次提交很多文件：
	```
	$ git add file1.txt
	$ git add file2.txt file3.txt
	$ git commit -m "add 3 files."
	```

5. 当对文件有了修改后：
（1）`$ git status` 查看仓库当前的状态
（2）`$ git diff readme.txt` 查看具体修改了什么内容，`$ git diff HEAD -- readme.txt` 查看工作区和版本库里面最新版本的区别
（3）`$ git add readme.txt` 将文件添加到git仓库
（4）`$ git commit -m "add distributed"` 将文件提交到git仓库
（5）`$ git status` 查看仓库当前的状态

6. 版本切换
（1）`$ git log` 命令显示从最近到最远的提交日志,如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数。你看到的一大串类似`1094adb...`的是`commit id（版本号）`，很重要。
（2）回退到上一版本:			   
	首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，也就是最新的提交`1094adb...`（注意我的提交ID和你的肯定不一样），
	上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个^比较容易数不过来，所以写成`HEAD~100`。
	```
	$ git reset --hard HEAD^
	HEAD is now at e475afc add distributed
	```
	（3）回到指定版本（找到指定版本的commit id就可以）
	```
	$ git reset --hard 1094a（id号不用写全，可以分辨就好）
	HEAD is now at 83b0afe append GPL
	```
	如果向上找不到`commit id`，不要着急：
	```
	$ git reflog
	e475afc HEAD@{1}: reset: moving to HEAD^
	1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
	e475afc HEAD@{3}: commit: add distributed
	eaadf4e HEAD@{4}: commit (initial): wrote a readme file
	要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。
	```
7. 撤销修改：
	场景1：
		当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，
	用命令`git checkout -- file`。

	场景2：
		当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想
	丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

	场景3：
		已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版
	本回退一节，不过前提是没有推送到远程库。
	
8. 删除文件：  
（1）在Git中，删除也是一个修改操作，我们实战一下，先添加一个新文件test.txt到Git并且提交： 
	``` 
	$ git add test.txt 
	$ git commit -m "add test.txt"  
	```
	（2）一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用`rm`命令删了	`$ rm test.txt` 

	（3）这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，`git status`命令会立刻告诉你哪些文件被删除了

	```
	$ git status
	On branch master
	Changes not staged for commit:
	(use "git add/rm <file>..." to update what will be committed)    
	(use "git restore <file>..." to discard changes in working directory)
		deleted:    test.txt  
	no changes added to commit (use "git add" and/or "git commit -a")
	```

	（4）或者使用`git restore <file>`可以很轻松地把误删的文件恢复到最新版本。  
	（5）使用`git add` 添加修改到暂存区，或者用`git rm `从版本库中删除该文件  
	并且`git commit` 提交

# 二、远程仓库
***
1、配置

**第1步：创建SSHKey。**

在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
	`$ ssh-keygen -t rsa -C "youremail@example.com"`

你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。

如果一切顺利的话，可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，`id_rsa`是私钥，不能泄露出去，	`id_rsa.pub`是公钥，可以放心地告诉任何人。

**第2步：登陆GitHub**

打开`“Account settings”`，`“SSH Keys”`页面：
然后，点`“Add SSH Key”`，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容。

**为什么GitHub需要SSH Key呢？**

因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充
的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。

当然，GitHub允许你添加多个Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。

>最后友情提示，在GitHub上免费托管的Git仓库，任何人都可以看到喔（但>只有你自己才能改）。所以，不要把敏感信息放进去。
如果你不想让别人看到Git库，有两个办法，一个是交点保护费，让GitHub把公开的仓库变成私有的，这样别人就看不见了（不可读更不可写）。
另一个办法是自己动手，搭一个Git服务器，因为是你自己的Git服务器，所以别人也是看不见的。这个方法我们后面会讲到的，相当简单，公司内部开发必备。

**第3步：在GitHub上创建一个git仓库**

**第4步：将本地与远程库关联**

`$ git remote add origin https://github.com/wzw9803/firstGit.git`

**第5步：将本地库的所有内容推送到远程库上：**

`$ git push -u origin master`
把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。
由于远程库是空的，我们第一次推送`master`分支时，加上了-u参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

从现在起，只要本地作了提交，就可以通过命令：
`git push origin master`
把本地`master`分支的最新修改推送至GitHub

2、从远程库克隆

首先，登陆GitHub，创建一个新的仓库，名字叫gitskills：
我们勾选Initialize this repository with a README，	这样GitHub会自动为我们创建一个README.md文件。创建完毕后，可以看到README.md文件

现在，远程库已经准备好了，下一步是用命令git clone克隆一个本地库
`$ git clone git@github.com:michaelliao/gitskills.git`

如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了。
你也许还注意到，GitHub给出的地址不止一个，
还可以用`https://github.com/michaelliao/gitskills.git`这样的地址。
实际上，Git支持多种协议，默认的`git://使用ssh`，但也可以使用`https`等其他协议。
使用`https`除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放`http`端口的公司内部就无法使用`ssh`协议而只能用`https`。
	
# 三、创建于合并分支		
***				
因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。

Git鼓励大量使用分支：
查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`


	
	
	
	
	
	
	
	
	