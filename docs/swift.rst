安装和配置 Swift
==========

主节点
----------

安装
~~~~~~~~~~

::

    yum install openstack-swift openstack-swift-proxy \
        openstack-swift-account openstack-swift-container \
        openstack-swift-object memcached
    
修改配置文件
~~~~~~~~~~

**/etc/swift/swift.conf** ::

    [swift-hash]
    swift_hash_path_suffix = <random string>
    
创建 SSL 自签名证书
~~~~~~~~~~

::

    cd /etc/swift
    openssl req -new -x509 -nodes -out cert.crt -keyout cert.key
    
配置 proxy-server
~~~~~~~~~~

**/etc/swift/proxy-server.conf** ::
    
    [DEFAULT]
    bind_port = 8888
    user = demo
    [pipeline:main]
    pipeline = healthcheck cache swift3 authtoken keystone proxy-server
    [app:proxy-server]
    use = egg:swift#proxy
    allow_account_management = true
    account_autocreate = true
    [filter:keystone]
    paste.filter_factory = keystone.middleware.swift_auth:filter_factory
    operator_roles = Member,admin, swiftoperator
    [filter:authtoken]
    paste.filter_factory = keystone.middleware.auth_token:filter_factory
    # Delaying the auth decision is required to support token-less
    # usage for anonymous referrers ('.r:*').
    delay_auth_decision = 10
    service_port = 5000
    service_host = 127.0.0.1
    auth_port = 35357
    auth_host = 127.0.0.1
    auth_protocol = http
    auth_uri = http://127.0.0.1:5000/
    auth_token = <keystone 中的 admin_token>
    admin_token = <keystone 中的 admin_token>
    admin_tenant_name = service
    admin_user = swift
    admin_password = swift
    signing_dir = /etc/swift # notice
    [filter:cache]
    use = egg:swift#memcache
    set log_name = cache
    [filter:catch_errors]
    use = egg:swift#catch_errors
    [filter:healthcheck]
    use = egg:swift#healthcheck
    
创建 account, container, object rings
~~~~~~~~~~

::

    swift-ring-builder account.builder create 18 1 1
    swift-ring-builder container.builder create 18 1 1
    swift-ring-builder object.builder create 18 1 1
    
(第二个参数为每个对象复制的个数）

添加每个节点上的存储设备
~~~~~~~~~~

::

    swift-ring-builder account.builder add z<ZONE>-<STORAGE_LOCAL_NET_IP>:6002/loop0 100
    swift-ring-builder container.builder add z<ZONE>-<STORAGE_LOCAL_NET_IP>:6001/loop0 100
    swift-ring-builder object.builder add z<ZONE>-<STORAGE_LOCAL_NET_IP>:6000/loop0 100
    
检查每个 ring 中的内容
~~~~~~~~~~

::

    swift-ring-builder account.builder
    swift-ring-builder container.builder
    swift-ring-builder object.builder
    
重新平衡 ring
~~~~~~~~~~~

::

    swift-ring-builder account.builder rebalance
    swift-ring-builder container.builder rebalance
    swift-ring-builder object.builder rebalance
    
复制 account.ring.gz, container.ring.gz 和 object.ring.gz 到存储节点。

确认所以配置文件为swift用户所有
~~~~~~~~~~

::

    chown -R swift:swift /etc/swift

启动 proxy 服务
~~~~~~~~~~

::

    swift-init proxy start
    
存储节点
----------

安装
~~~~~~~~~~

::

    yum install openstack-swift-account openstack-swift-container \
        openstack-swift-object xfsprogs
        
配置虚拟磁盘
~~~~~~~~~~

::

    dd if=/dev/zero of=swift.img bs=10M count=1024
    iosetup --show -f swift.img
    mkfs.xfs -i size=1024 /dev/loop0
    mkdir -p /srv/node/loop0
    mount /dev/loop0 /srv/node/loop0
    chown -R swift:swift /srv/node
    
配置 rsync
~~~~~~~~~~

**/etc/rsyncd.conf** ::

    uid = swift
    gid = swift
    log file = /var/log/rsyncd.log
    pid file = /var/run/rsyncd.pid
    address = <STORAGE_LOCAL_NET_IP>
    [account]
    max connections = 2
    path = /srv/node/
    read only = false
    lock file = /var/lock/account.lock
    [container]
    max connections = 2
    path = /srv/node/
    read only = false
    lock file = /var/lock/container.lock
    [object]
    max connections = 2
    path = /srv/node/
    read only = false
    lock file = /var/lock/object.lock
    
启动 rsync
~~~~~~~~~~

::
    
    rsync --daemon --config=/etc/rsyncd.conf
    
配置 swift
~~~~~~~~~~

::

**/etc/swift/account-server.conf** ::

    [DEFAULT]
    bind_ip = <STORAGE_LOCAL_NET_IP>
    workers = 2
    [pipeline:main]
    pipeline = account-server
    [app:account-server]
    use = egg:swift#account
    [account-replicator]
    [account-auditor]
    [account-reaper]

**/etc/swift/account-container.conf** ::

    [DEFAULT]
    bind_ip = <STORAGE_LOCAL_NET_IP>
    workers = 2
    [pipeline:main]
    pipeline = container-server
    [app:container-server]
    use = egg:swift#container
    [container-replicator]
    [container-auditor]
    [container-reaper]
    
**/etc/swift/object-server.conf** ::

    [DEFAULT]
    bind_ip = <STORAGE_LOCAL_NET_IP>
    workers = 2
    [pipeline:main]
    pipeline = object-server
    [app:object-server]
    use = egg:swift#object
    [object-replicator]
    [object-updater]
    [object-auditor]
    [object-expirer]
    
启动服务
~~~~~~~~~~

::

    swift-init object-server start
    swift-init object-replicator start
    swift-init object-updater start
    swift-init object-auditor start
    swift-init container-server start
    swift-init container-replicator start
    swift-init container-updater start
    swift-init container-auditor start
    swift-init account-server start
    swift-init account-replicator start
    swift-init account-auditor start


验证安装
----------

::

    swift -V 2.0 -A http://127.0.0.1:35357/v2.0 \
        -U demo:admin -K admin stat
    swift -V 2.0 -A http://127.0.0.1:35357/v2.0 \
        -U demo:admin -K admin upload myfile <file> # 上传文件
    swift -V 2.0 -A http://127.0.0.1:35357/v2.0 \
        -U demo:admin -K admin download myfile # 下载文件









