# Go语言环境安装
## 下载
	cd /usr/local
	wget https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
## 在/usr/local下安装程序
	tar -zxvf go1.8.3.linux-amd64.tar.gz
## 配置全局变量
	vi /etc/profile
在这个文件末尾加入下面内容：

	export GOPATH=/root/go
	export GOROOT=/usr/local/go
	export PATH=$PATH:$GOROOT/bin
## 加载使生效
	source /etc/profile
## 验证是否生效
	go version
## GOPATH目录结构：
	/root/go/
		--/src/  存放源代码(.go .c .h .s等)
	    --/pkg/　　编译后生成的文件(.a)
	    --/bin/　　编译后生成的可执行文件