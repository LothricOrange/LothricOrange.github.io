### 文件拷贝

```powershell
[root@master Downloads]# scp erlang-23.2.7-1.el7.x86_64.rpm root@slave01:~/Downloads/
[root@master Downloads]# scp erlang-23.2.7-1.el7.x86_64.rpm root@slave02:~/Downloads/
[root@master Downloads]# scp erlang-23.2.7-1.el7.x86_64.rpm root@slave03:~/Downloads/

[root@master Downloads]# scp rabbitmq-server-3.9.15-1.el7.noarch.rpm root@slave01:~/Downloads/
[root@master Downloads]# scp rabbitmq-server-3.9.15-1.el7.noarch.rpm root@slave02:~/Downloads/
[root@master Downloads]# scp rabbitmq-server-3.9.15-1.el7.noarch.rpm root@slave03:~/Downloads/
```

###  安装 erlang

```powershell
[root@master Downloads]# rpm -ivh erlang-23.2.7-1.el7.x86_64.rpm
[root@master Downloads]# erl -version
```

### 安装socat

```powershell
[root@master Downloads]# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
[root@master Downloads]# curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@master Downloads]# yum makecache
```

```powershell
[root@master Downloads]# yum install socat
```

### socat安装完成后，就可以安装RabbitMQ了

```powershell
[root@master Downloads]# rpm -Uvh rabbitmq-server-3.9.15-1.el7.noarch.rpm
```

### 安装完成后，可以查看下他的安装情况

```powershell
[root@master Downloads]# whereis rabbitmqctl
```

### 启动RabbitMQ服务

```powershell
[root@master Downloads]# service rabbitmq-server start
```

### 查看服务状态
```powershell
[root@master Downloads]# service rabbitmq-server status
```

### 其他常用的启停操作：

```powershell
rabbitmq-server -deched --后台启动服务
rabbitmqctl start_app --启动服务
rabbitmqctl stop_app --关闭服务
```

### 配置下打开他的Web管理页面

```powershell
[root@master Downloads]# rabbitmq-plugins enable rabbitmq_management
```

<font color="white">
    这时，可以使用默认的guest/guest用户登录。 但是注意下，默认情况下，只允许在localhost本地登录，远程访问是无法登录的。这时，可以创建一个管理员账户来登录。
</font>

```powershell
[root@master Downloads]# rabbitmqctl add_user admin admin
[root@master Downloads]# rabbitmqctl set_permissions -p / admin "." "." ".*"
[root@master Downloads]# rabbitmqctl set_user_tags admin administrator
```

### RabbitMQ集群搭建

```powershell
[root@slave01 Downloads]# scp /var/lib/rabbitmq/.erlang.cookie root@master:/var/lib/rabbitmq
[root@slave01 Downloads]# scp /var/lib/rabbitmq/.erlang.cookie root@slave02:/var/lib/rabbitmq
[root@slave01 Downloads]# scp /var/lib/rabbitmq/.erlang.cookie root@slave03:/var/lib/rabbitmq
```

#### master, slave02, slave03

```powershell
[root@master ~]# cp /var/lib/rabbitmq/.erlang.cookie ~/.erlang.cookie
[root@master ~]# service rabbitmq-server stop
[root@master ~]# service rabbitmq-server start
## [root@master ~]# rabbitmqctl stop_app
[root@master ~]# rabbitmqctl join_cluster --ram rabbit@slave01
[root@master Downloads]# rabbitmq-plugins enable rabbitmq_management
```



### error

```powershell

[root@master ~]# rabbitmqctl join_cluster --ram rabbit@slave01
Error: unable to perform an operation on node 'rabbit@master'. Please see diagnostics information and suggestions below.

Most common reasons for this are:

 * Target node is unreachable (e.g. due to hostname resolution, TCP connection or firewall issues)
 * CLI tool fails to authenticate with the server (e.g. due to CLI tool's Erlang cookie not matching that of the server)
 * Target node is not running

In addition to the diagnostics info below:

 * See the CLI, clustering and networking guides on https://rabbitmq.com/documentation.html to learn more
 * Consult server logs on node rabbit@master
 * If target node is configured to use long node names, don't forget to use --longnames with CLI tools

DIAGNOSTICS
===========

attempted to contact: [rabbit@master]

rabbit@master:
  * connected to epmd (port 4369) on master
  * epmd reports node 'rabbit' uses port 25672 for inter-node and CLI tool traffic
  * TCP connection succeeded but Erlang distribution failed
  * suggestion: check if the Erlang cookie is identical for all server nodes and CLI tools
  * suggestion: check if all server nodes and CLI tools use consistent hostnames when addressing each other
  * suggestion: check if inter-node connections may be configured to use TLS. If so, all nodes and CLI tools must do that
   * suggestion: see the CLI, clustering and networking guides on https://rabbitmq.com/documentation.html to learn more


Current node details:
 * node name: 'rabbitmqcli-453-rabbit@master'
 * effective user's home directory: /var/lib/rabbitmq
 * Erlang cookie hash: XnbAFRwaUl5DCV7ty5nhzA==

```

### 解决

```powershell
[root@master ~]# cp /var/lib/rabbitmq/.erlang.cookie ~/.erlang.cookie
[root@master ~]# service rabbitmq-server stop
[root@master ~]# service rabbitmq-server start
```

