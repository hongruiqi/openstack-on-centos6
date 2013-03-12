安装和配置 Keystone
==========

安装
----------

安装 openstack-keystone
~~~~~~~~~~

::

    yum install openstack-utils openstack-keystone python-keystoneclient
    
初始化数据库
~~~~~~~~~~

修改`/etc/keystone/keystone.conf`中`sql->connection`项为

::    

    mysql://keystone:keystone@127.0.0.1:3306/keystone
    

*因为mysql版本的问题，`127.0.0.1` 不可写为 `localhost`，且端口号不可忽略，否则连接不上数据库。所有mysql连接的配置都需注意。*
    
在mysql中建立keystone数据库和用户，并赋予权限。

:: 
   
    openstack-db --init --service keystone
    
如上命令会根据配置文件中的 `connection` 项，创建 `keystone` 数据库和 `keystone` 用户，密码为 `keystone`。

设置 admin_token
~~~~~~~~~~

::    

    openstack-config --set /etc/keystone/keystone.conf DEFAULT admin_token <admin_token>
    
admin_token 为一密钥，用于操作keystone时的认证。

同步数据库
~~~~~~~~~~

::

    keystone-manage db_sync

如上命令在keystone数据库中创建相应数据表。

启动 keystone 服务
~~~~~~~~~~

::

    service openstack-keystone start

配置服务
----------

创建用户、租户和角色
~~~~~~~~~~

创建租户

::

    keystone --token <admin_token> --endpoint http://127.0.0.1:35357/v2.0 tenant-create --name demo
    
创建用户并与租户绑定

::

    keystone --token <admin_token> --endpoint http://127.0.0.1:35357/v2.0 user-create --tenant-id <上一步中返回的tenant-id> --name admin --pass admin
    
创建管理员角色(根据keystone默认的`policy.json`)

::

    keystone --token <admin_token> --endpoint http://127.0.0.1:35357/v2.0 role-create --name admin
 
赋予demo中的admin用户管理员权限

::

    keystone --token <admin_token> --endpoint http://127.0.0.1:35357/v2.0 user-role-add --tenant-id <tenant-id> --user-id <user-id> --role-id <role-id>

创建服务
~~~~~~~~~~
    
修改 `/etc/keystone/keystone.conf` 中，`catalog->driver` 项为`keystone.catalog.backends.sql.Catalog`，即设置服务目录采用数据库存储。

**定义 `Identity` 服务**

::

    keystone --token <admin-token> --endpoint http://127.0.0.1:35357/v2.0 service-create --name=keystone --type=identity

    keystone --token <admin-token> \
    --endpoint http://127.0.0.1:35357/v2.0 \
    endpoint-create \
    --region scut \
    --service=id=<上一步返回的service-id> \
    --publicurl=http://192.168.1.1:5000/v2.0 \
    --internalurl=http://192.168.1.1:5000/v2.0 \
    --adminurl=http://192.168.1.1:35357/v2.0

**定义 `Compute` 服务**

::

    keystone --token <admin-token> --endpoint http://127.0.0.1:35357/v2.0 service-create --name=nova --type=compute

    keystone --token <admin-token> \
    --endpoint http://127.0.0.1:35357/v2.0 \
    endpoint-create \
    --region scut \
    --service=id=<上一步返回的service-id> \
    --publicurl='http://192.168.1.1:8774/v2/%(tenant_id)s' \
    --internalurl='http://192.168.1.1:8774/v2/%(tenant_id)s' \
    --adminurl='http://192.168.1.1:8774/v2/%(tenant_id)s'
    
**定义`Volume`服务**

::

    keystone --token <admin-token> --endpoint http://127.0.0.1:35357/v2.0 service-create --name=volume --type=volume

    keystone --token <admin-token> \
    --endpoint http://127.0.0.1:35357/v2.0 \
    endpoint-create \
    --region scut \
    --service=id=<上一步返回的service-id> \
    --publicurl='http://192.168.1.1:8776/v1/%(tenant_id)s' \
    --internalurl='http://192.168.1.1:8776/v1/%(tenant_id)s' \
    --adminurl='http://192.168.1.1:8776/v1/%(tenant_id)s'

**定义`Image`服务**

::

    keystone --token <admin-token> --endpoint http://127.0.0.1:35357/v2.0 service-create --name=glance --type=image

    keystone --token <admin-token> \
    --endpoint http://127.0.0.1:35357/v2.0 \
    endpoint-create \
    --region scut \
    --service=id=<上一步返回的service-id> \
    --publicurl='http://192.168.1.1:9292' \
    --internalurl='http://192.168.1.1:9292' \
    --adminurl='http://192.168.1.1:9292'
   
**定义`EC2`兼容服务**

::

    keystone --token <admin-token> --endpoint http://127.0.0.1:35357/v2.0 service-create --name=ec2 --type=ec2

    keystone --token <admin-token> \
    --endpoint http://127.0.0.1:35357/v2.0 \
    endpoint-create \
    --region scut \
    --service=id=<上一步返回的service-id> \
    --publicurl='http://192.168.1.1:8773/services/Cloud' \
    --internalurl='http://192.168.1.1:8773/services/Cloud' \
    --adminurl='http://192.168.1.1:8773/services/Admin'

**定义`Object Storage`服务**

::

    keystone --token <admin-token> --endpoint http://127.0.0.1:35357/v2.0 service-create --name=swift --type=object-store

    keystone --token <admin-token> \
    --endpoint http://127.0.0.1:35357/v2.0 \
    endpoint-create \
    --region scut \
    --service=id=<上一步返回的service-id> \
    --publicurl='http://192.168.1.1:8888/v1/AUTH_%(tenant_id)s' \
    --internalurl='http://192.168.1.1:8888/v1/AUTH_%(tenant_id)s' \
    --adminurl='http://192.168.1.1:8888/v1'

验证 Identify 服务安装
~~~~~~~~~~

验证 keystone 是否正确运行以及用户是否正确建立。

::

    keystone --os-username=admin --os-password=admin --os-auth-url=http://127.0.0.1:35357/v2.0 token-get
    
验证用户在指定的 tenant 上是否有明确定义的角色。

::

    keystone --os-username=admin --os-password=admin --os-tenant-name=demo --os-auth-url=http://127.0.0.1:35357/v2.0 token-get
    
可以将以上参数设置为环境变量，不用每次输入

::

    export OS_USERNAME=admin
    export OS_PASSWORD=admin
    export OS_TENANT_NAME=demo
    export OS_AUTH_URL=http://127.0.0.1:35357/v2.0 # 管理员命令必须通过 35357 端口执行

此时可直接运行

::

    keystone token-get
    
最后，验证admin账户有权限执行管理命令

::    

    keystone user-list

