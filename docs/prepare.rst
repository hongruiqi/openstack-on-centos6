准备工作
===========

设置软件源
-----------

更改 centos 源
~~~~~~~~~~~

备份系统自带配置

    cd /etc/yum.repos.d
    mv CentOS-Base.repo.backup
    
从 中科大镜像_ 下载相应版本 ``CentOS-Base.repo`` 文件，放入 ``/etc/yum.repos.d``。

.. _中科大镜像: http://lug.ustc.edu.cn/wiki/mirrors/help/centos
