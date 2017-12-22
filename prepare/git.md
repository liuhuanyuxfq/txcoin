## Git安装
### centos：
	yum install git
### ubuntu：
	apt-get install git
### 验证安装：
	git --version
## Git常用命令
- 创建版本库：`git init`
- 把文件添加到仓库的暂存区：`git add <filename>`
- 把暂存区的内容添加到仓库：`git commit -m "<message>"`
- 查看仓库当前状态：`git status`
- 查看差异：`git diff <filename>`
- 查看工作区和当前版本库文件的区别：`git diff HEAD -- <filename>`
- 查看版本历史：
 
   		git log
    	git log --pretty=oneline

- 切换版本：

		git reset --hard HEAD^      #切换到上一版本
		git reset --hard HEAD~10	#切换到前10个版本
		git reset --hard <版本号>    #切换到指定版本
- 撤销修改：
 
		git checkout -- <filename>      #工作区未add到暂存区
		git reset HEAD <filename>		#已经add到暂存区，先执行这个，然后再执行上面一行
- 删除文件：`git rm <filename>`
- 查看分支：`git branch`
- 创建分支：`git branch <name>`
- 切换分支：`git checkout <name>`
- 创建+切换分支：`git checkout -b <name>`
- 合并某分支到当前分支：`git merge <name>`
- 删除分支：`git branch -d <name>`
- 查看本地仓库和远程仓库差异：

		git fetch origin    		 	# 获取远端库最新信息
		git diff master origin/master   # 做diff
- 添加远程仓库：`git remote add <name> <url>`
- 将本地仓库推送到远程仓库：`git push -u origin master`
 

## 使用项目仓库更新自己的仓库：
    git remote add upstream https://github.com/yeasy/hyperledger_code_fabric
    git fetch upstream
    git checkout master
    git rebase upstream/master
    git push -f origin master

## 本地项目添加到git并推送到远端仓库
### 1.设置git的用户名和邮箱
	git config --global --list								#列出所有全局设置
    git config --global user.name"liuhy"					#设置全局的用户名
    git config --global user.email "liuhuanyu@neusoft.com"  #设置全局的邮箱
### 2.导新项目到远端仓库（gitlab为例）上
    cd "本地存在项目的路径"  
    git init		#创建git版本库
    git remote add <NAME> git@gitlab.lhy.com:<USERNAME>/<PROJECTNAME>.git	#添加远端仓库链接，<NAME>是远端仓库的别名
    git add .		#把当前目录添加到本地缓存区
    git commit -m 'first git demo'	
    git push -u <NAME> master
## 比较好的博客：
- [http://www.cnblogs.com/schaepher/p/5561193.html]()
- [http://blog.csdn.net/niuszeng/article/details/51305061]()