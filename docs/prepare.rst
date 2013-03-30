准备工作
==========

设置软件源
----------

更改 centos 源
~~~~~~~~~~

备份系统配置 ::

    cd /etc/yum.repos.d
    mv CentOS-Base.repo.backup
    
从 中科大镜像_ 下载相应版本 `CentOS-Base.repo` 文件，放入 `/etc/yum.repos.d`。

.. _中科大镜像: http://lug.ustc.edu.cn/wiki/mirrors/help/centos

运行 ``yum makecache`` 生成缓存。

添加 epel 仓库
~~~~~~~~~~

下载安装 epel 配置文件 ::
    
    wget -c http://mirror.neu.edu.cn/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
    rpm -i epel-release-6-8.noarch.rpm
    cd /etc/yum.repos.d
    
打开 `epel.repo`, 修改配置 ::

    [epel]
    enabled = 1
    baseurl = http://mirrors.ustc.edu.cn/fedora/epel/6/$basearch
    # mirrorlist = ...  # 注释此项
    
运行 `yum makecache` 生成缓存。

安装 MySQL 和 rabbitmq
----------

安装 MySQL
~~~~~~~~~~

::

    yum install mysql mysql-server MySQL-python # 安装 MySQL 数据库及其 Python 绑定
    service mysqld start      # 启动 MySQL 服务
    chkconfig mysqld on       # 设置开机启动服务
    mysql_secure_installation # 设置root密码，删除空用户和测试表
    
安装 rabbitmq
~~~~~~~~~~

::
    
    yum install rabbitmq-server # 安装 rabbitmq-server 服务器
    service rabbitmq-server start  # 启动 rabbitmq-server
    chkconfig rabbitmq-server on   # 设置开机启动服务
