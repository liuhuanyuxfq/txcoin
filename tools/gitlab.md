# GitLab安装和使用
## 安装
### 1.安装和配置必要的依赖
在Ubuntu 16.04上（推荐），下面的命令还会打开系统防火墙中的HTTP和SSH访问。

    sudo apt-get update
    sudo apt-get install -y curl openssh-server ca-certificates


### 2.添加GitLab软件包储存库并安装软件包
添加GitLab软件包库。

    curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
接下来，安装GitLab软件包。 EXTERNAL_URL为您要访问您的GitLab实例的URL。 安装将自动配置并启动该URL的GitLab。 HTTPS安装后需要额外的配置。

    EXTERNAL_URL="http://gitlab.lhy.com" apt-get install gitlab-ee
### 3.浏览器打开URL并登录
在第一次访问时，您将被重定向到密码重置屏幕。 提供初始管理员帐户的密码，您将被重定向回登录屏幕。 使用默认帐户的用户名root登录。
详见[See our documentation for detailed instructions on installing and configuration]().

## 参考资源
- [官网安装指南](https://about.gitlab.com/installation/#ubuntu)
- [BitNami安装指南](https://bitnami.com/stack/gitlab)
- [CSDN上的GitLab使用总结](http://blog.csdn.net/huaishu/article/details/50475175)