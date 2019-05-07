### 一、Gitlab环境搭建
已有该环境直接看第二部分
#### 安装依赖

虚拟机配置：2vCPU+4Gb(MEM)

操作系统版本：CentOS7.6
```
yum install curl policycoreutils openssh-server openssh-clients policycoreutils-python

# 开启sshd
systemctl enable sshd
systemctl start sshd

# 使用postfix发送邮件通知
yum install postfix 
systemctl enable postfix 
systemctl start postfix 

# 打开防火墙端口，如果需要
firewall-cmd --permanent --add-service=http 
systemctl reload firewalld
```

#### 下载GitLab包

centos 6系统的下载地址:https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6

centos 7系统的下载地址:https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7

```
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-11.8.7-ce.0.el7.x86_64.rpm

rpm -ivh gitlab-ce-10.8.1-ce.0.el7.x86_64.rpm
```



#### 配置GitLab

GitLab的相关参数配置都存在/etc/gitlab/gitlab.rb文件中，每次配置完成之后需要执行“gitlab-ctl reconfigure”，进行重新配置才能生效。

```
# vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.1.100' 

# gitlab-ctl reconfigure
```

#### 通过HTTP访问GitLab

浏览器中打开URL: http://192.168.1.100，第一次登陆会跳转到修改root用户密码页面，然后修改完跳转到登录，账号为root，密码为之前修改的密码。



#### 禁止GitLab服务开机自启动

```
# systemctl list-unit-files|grep -i gitlab

# systemctl disable gitlab-runsvdir

# systemctl list-unit-files|grep -i gitlab
```

#### GitLab的常用命令

语法:
```
gitlab-ctl command (subcommand)

start	启动所有服务

stop	关闭所有服务

restart	重启所有服务

status	查看所有服务状态

tail	查看日志信息

service-list	列举所有启动服务

graceful-kill	平稳停止一个服务

help	    帮助

reconfigure	修改配置文件之后，需要重新加载下

show-config	查看所有服务配置文件信息

uninstall	卸载这个软件

cleanse	    删除gitlab数据

示例：

gitlab-ctl start	    #启动所有服务

gitlab-ctl start nginx    #启动单独一个服务

gitlab-ctl tail	  #查看日志，类似tail -f

gitlab-ctl tail nginx	    #查看一个应用的日志

gitlab-ctl show-config	    #显示所有服务配置文件

gitlab-ctl uninstall	    #卸载gitlab
```

### 二、CI／CD与K8S集成