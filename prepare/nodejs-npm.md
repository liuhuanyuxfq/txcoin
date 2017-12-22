# Node.js Runtime和NPM的安装
## 一、Node.js安装
### 1.安装gcc
	yum install gcc-c++
### 2.安装nvm
nvm（Node version manager），就是Node.js的版本管理软件，可以轻松的在Node.js各个版本间切换。

	git clone https://github.com/cnpm/nvm.git #克隆项目
	cd nvm #进入项目目录
	./nvm.sh #执行安装脚本
	node -v	#查看node版本
	npm -v	#查看npm版本
### 3.通过nvm安装管理nodejs

1. 列出所有可安装的版本:`nvm list-remote`；
2. 安装相应的版本使用:`nvm install v0.12.4`；还可以直接安装 iojs 各个版本；
3. 查看一下你当前已经安装的版本:`nvm ls`；
4. 切换版: `nvm use v0.12.4`；
5. 设置默认版本:`nvm alias default v0.12.4`;
6. 使用帮助:`nvm help`；