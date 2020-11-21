---
layout:     post
author:     zjh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - 学习记录
---
# 安装篇
## 1. 安装依赖软件
```
yum  install -y curl policycoreutils-python openssh-server 
yum -y install policycoreutils openssh-server openssh-clients postfix
```
## 2.设置postfix开机自启，并启动，postfix支持gitlab发信功能
```
systemctl enable postfix && systemctl start postfix
```
## 3.下载gitlab安装包，然后安装
```
#参考清华大学开源镜像站
#centos 6系统的下载地址:https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6
#centos 7系统的下载地址:https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
#我的是centos7,所以我在https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7中找了个gitlab10.1.0版本
#下载rpm包
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-8.0.0-ce.0.el7.x86_64.rpm
#安装
rpm -i gitlab-ce-8.0.0-ce.0.el7.x86_64.rpm
```
## 4.修改gitlab配置文件指定服务器ip和自定义端口
vim  /etc/gitlab/gitlab.rb
![](https://images2015.cnblogs.com/blog/1008644/201609/1008644-20160911113500228-232680032.png)
退出并保存
>ps:注意这里设置的端口不能被占用，默认是8080端口，如果8080已经使用，请自定义其它端口，并在防火墙设置开放相对应得端口。防火墙设置请参考<a href="/markdownDetail.html?id=99089">《centos打开系统防火墙端口》</a>

## 5.重置并启动GitLab
```
#执行以下命令
gitlab-ctl reconfigure
gitlab-ctl restart
```
## 6.访问 GitLab页面
如果没有域名，直接输入服务器ip和指定端口进行访问，初始账户: root 密码: 5iveL!fe  
第一次登录修改密码
## 邮箱配置
官网参考地址：<a href="https://docs.gitlab.com/omnibus/settings/smtp.html#qq-exmail">https://docs.gitlab.com/omnibus/settings/smtp.html#qq-exmail</a>    
我这使用的是腾讯企业邮箱
```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "xxxx@xx.com"
gitlab_rails['smtp_password'] = "password"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = 'xxxx@xx.com'
gitlab_rails['smtp_domain'] = "exmail.qq.com"
```
# 汉化篇
## 0、环境
建议在本地吧diff打好传到服务器去，如果提示 -bash: patch: command not found 执行：
```
yum -y install patch
```
## 1、下载汉化补丁
```
[root@gitlab ~]# git clone https://gitlab.com/xhang/gitlab.git
[root@gitlab ~]# cd gitlab    
```
## 2、查看全部分支版本
```
[root@gitlab ~]# git branch -a
```
## 3、对比版本、生成补丁包
```
[root@gitlab ~]# git diff remotes/origin/10-1-stable remotes/origin/10-1-stable-zh > /tmp/10.1.0-zh.diff
```
## 4、停止服务器
```
[root@gitlab ~]# gitlab-ctl stop
```
## 5、打补丁
```
[root@gitlab ~]# patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < /tmp/10.1.0-zh.diff
```
## 6、启动和重新配置
```
[root@gitlab ~]# gitlab-ctl start
[root@gitlab ~]# gitlab-ctl reconfigure
```
# 卸载篇
## 1、停止 GitLab
```
sudo gitlab-ctl stop
```
## 2、卸载 GitLab
```
sudo rpm -e gitlab-ce
```
## 3、查看 GitLab 进程
```
ps -ef|grep gitlab
```
## 4、杀掉第一个守护进程( 进程序号每人都不一样要注意)
```
kill -9 3370
```
## 5、再次查看 GitLab 进程是否存在
```
ps -ef|grep gitlab
```
## 6、删除 GitLab 文件
```
// 删除所有包含gitlab的文件及目录
find / -name *gitlab*|xargs rm -rf

// 删除 gitlab-ctl uninstall 时自动在 root 下备份的配置文件
find / -name gitlab |xargs rm -rf
```
通过几步就可以彻底卸载 GitLab
# 优化篇
## 降低gitlab的资源消耗
因为gitlab的配置要求至少是4g以上内存，如果服务器使用人数少，还是可以搭建，只是请求成功比例不一定会100%  
```
#通过修改/etc/gitlab/gitlab.rb里面建议配置：
  
unicorn['worker_processes'] = 1
postgresql['max_worker_processes'] = 2
nginx['worker_processes'] = 1

#官方推荐值不少于2：cpu核数+1，但是看使用人数而定，毕竟稳定时最重要的
```
修改完记得 gitlab-ctl reconfigure
