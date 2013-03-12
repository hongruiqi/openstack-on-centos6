安装和配置 Glance
==========

安装
----------

安装 openstack-glance
~~~~~~~~~~

::

    yum install openstack-nova openstack-glance
    
配置 glance 数据库
~~~~~~~~~~

::

    mysql -u root -p
    create database glance;
    grant all on glance.* to 'glance'@'%' identified by 'glance';
    grant all on glance.* to 'glance'@'localhost' identified by 'glance';
    
修改配置文件
----------

**/etc/glance/glance-api.conf** ::

    enable_v1_api=True
    enable_v2_api=True
    
    [keystone_authtoken]
    auth_host = 127.0.0.1
    auth_port = 35357
    auth_protocol = http
    admin_tenant_name = service
    admin_user = glance
    admin_password = glance
    
    flavor=keystone
    
    sql_connection = mysql://glance:glance@127.0.0.1:3306/glance
    
**/etc/glance/glance-api-paste.ini** ::

    [filter:authtoken]
    admin_tenant_name = service
    admin_user = glance
    admin_password = glance
    
**glance-registry.conf** ::

    [keystone_authtoken]
    auth_host = 127.0.0.1
    auth_port = 35357
    auth_protocol = http
    admin_tenant_name = service
    admin_user = glance
    admin_password = glance
    
    flavor = keystone
    
    sql_connection = mysql://glance:glance@127.0.0.1:3306/glance
    
**glance-registry-paste.ini** ::
    
    [pipeline:glance-registry-keystone]
    pipeline = authtoken context registryapp

同步数据库，启动服务
----------

::

    glance-manage db_sync
    service glance-registry restart
    service glance-api restart

验证 Glance 安装
----------

获取测试镜像 ::

    mkdir /tmp/images
    cd /tmp/images
    wget http://smoser.brickies.net/ubuntu/ttylinux-uec/ttylinux-uec-amd64-12.1_2.6.35-22_1.tar.gz
    tar -zxvf ttylinux-uec-amd64-12.1_2.6.35-22_1.tar.gz
    
设置环境变量 ::

    export OS_USERNAME=admin
    export OS_TENANT_NAME=demo
    export PASSWORD=admin
    export OS_AUTH_URL=http://127.0.0.1:5000/v2.0/
    export OS_REGION_NAME=scut
    
上传内核 ::

    glance image-create --name="tty-linux-kernel" \
    --disk-format=aki \
    --container-format=aki < ttylinux-uec-amd64-12.1_2.6.35-22_1-vmlinuz
    
上传 initrd ::

    glance image-create --name="tty-linux-ramdisk" \
    --disk-format=ari \
    --container-format=ari < ttylinux-uec-amd64-12.1_2.6.35-22_1-loader
    
上传镜像 ::

    glance image-create --name="tty-linux" \
    --disk-format=ami \
    --container-format=ami \
    --property kernel_id=<上面返回的kernel_id> \
    ramdisk_id=<上面返回的ramdisk_id> < ttylinux-uec-amd64-12.1_2.6.35-22_1.img
    
使用 image-list 命令应该显示三个镜像 ::

    glance image-list
