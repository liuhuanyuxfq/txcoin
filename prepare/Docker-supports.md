# 一、安装Docker
## CentOS:
### 添加yum源
	tee /etc/yum.repos.d/docker.repo <<-'EOF'
	[dockerrepo]
	name=Docker Repository
	baseurl=https://yum.dockerproject.org/repo/main/centos/7/
	enabled=1
	gpgcheck=1
	gpgkey=https://yum.dockerproject.org/gpg
	EOF
### 安装docker-engine
	yum install docker-engine
### 启动docker
	systemctl start docker.service
### 验证docker是否启动
	docker info
### 查看docker和docker-compose版本
	docker --version
	docker-compose --version
##  ubuntu:
	apt-get update
	apt-get install docker
**注：此系统的安装没有亲测！**
# 二、安装Docker Compose
## CentOS：
### 安装python-pip
	pip -V	#查看是否已经安装了python-pip
	yum -y install epel-release	#安装epel-release
	yum -y install python-pip	#安装python-pip
	pip install --upgrade pip	#对安装好的pip进行升级 
### 安装Docker-Compose
	pip install docker-compose
如果报`ReadTimeoutError: HTTPSConnectionPool(host='pypi.python.org', port=443): Read timed out`的错误，可以使用下面的命令安装：

	pip --default-timeout=200 install -U docker-compose
如果报`pkg_resources.DistributionNotFound: backports.ssl-match-hostname>=3.5`的错误，用下面的命令：
	
	pip install --upgrade backports.ssl_match_hostname 
### 查看docker-compose版本：
	docker-compose --version
##  ubuntu:
安装Docker会自带docker-compose（没有亲测）