安装和配置 nova -- 主节点
==========

网络设置
----------

网卡接口设置
~~~~~~~~~~

安装桥接管理软件 ::

    yum install bridge-utils
    
删除原有网卡上的桥接 ::

    brctl show
    brctl delbr <要删除的桥接网卡>

.. note:: 注意不要改动外网网卡的桥接，以免远程访问断开

重新设置 ip ::
    
    ifconfig peth0 192.168.1.1 # 假设 peth0 为内部连接端口
    
将网卡设置为混杂模式 ::

    ifconfig peth0 promisc

设置 selinux 为 permissive ::

    setenforce permissive

安装 dnsmasq
~~~~~~~~~~

::

    yum install dnsmasq-utils # 用作虚拟机 DHCP 自动获取 IP

配置数据库 
----------
   
在mysql中建立 nova 数据库和用户，并赋予权限
   
::
    
    mysql -u root -p
    create database nova;
    grant all on nova.* to 'nova'@'%' identified by nova;
    grant all on nova.* to 'nova'@'%' identified by nova;
    
修改配置文件
----------

**/etc/nova/nova.conf** ::
    
    [DEFAULT]
    # debug=True
    # verbose=True
    compute_scheduler_driver=nova.scheduler.filter_scheduler.FilterScheduler
    logdir = /var/log/nova
    state_path = /var/lib/nova
    lock_path = /var/lib/nova/tmp
    volumes_dir = /etc/nova/volumes
    iscsi_helper = tgtadm
    sql_connection = mysql://nova:nova@127.0.0.1:3306/nova
    compute_driver = libvirt.LibvirtDriver
    firewall_driver = nova.virt.libvirt.firewall.IptablesFirewallDriver
    #rpc_backend = nova.openstack.common.rpc.impl_qpid  # 默认使用 rabbitmq
    rootwrap_config = /etc/nova/rootwrap.conf
    libvirt_type = kvm
    my_ip=192.168.1.1
    # 注意此处路径需存在，且对nova用户具有读写权限
    instances_path=/state/partition1/openstack/instance

    #AUTH
    auth_strategy = keystone
    # 消息队列主机，因为 rabbitmq 部署在主节点上，与 nova 处于同一主机，故 ip 可设置为127.0.0.1
    rabbit_host = 127.0.0.1
    
    api_paste_config = /etc/nova/api-paste.ini

    #NETWORK
    dhcpbridge = /usr/bin/nova-dhcpbridge
    dhcpbridge_flagfile = /etc/nova/nova.conf
    force_dhcp_release = False
    libvirt_inject_partition = -1
    injected_network_template = /usr/share/nova/interfaces.template
    libvirt_nonblocking = True

    network_manager = nova.network.manager.FlatDHCPManager
    fixed_range=192.168.100.0/24  # VM IP 范围
    flat_network_bridge = br100   # 虚拟网桥名称
    # 因为几台服务器间只有一个端口相互连接，所以public和flat接口设置为同一个
    public_interface=peth0        
    flat_interface=peth0

    #VNC
    # 外网可访问的地址
    novncproxy_base_url=http://202.38.192.97:6080/vnc_auto.html

    [keystone_authtoken]
    admin_tenant_name = service
    admin_user = nova
    admin_password = nova
    auth_host = 127.0.0.1
    auth_port = 35357
    auth_protocol = http
    signing_dir = /tmp/keystone-signing-nova
    
同步数据库
----------

::

    nova-manage db sync
    
启动服务
----------

::

    for svc in api objectstore compute network volume scheduler cert;
    do 
        service openstack-nova-$svc start; # 启动服务 
        chkconfig openstack-nova-$svc on;  # 设置开机启动服务
    done
    
    
创建网络
----------

::

    nova-manage network create private --fixed_range_v4=192.168.100.0/24 \
        --bridge_interface=br100 --num_networks=1 --network_size=256
    
验证 nova 安装
----------

::

    nova-manage service list
    
此命令获取 nova 各服务的运行状况，在返回中笑脸为正常运行，`X` 为错误。

设置账户认证信息
----------

建立一个 openrc 文件 ::

    export OS_USERNAME=admin
    export OS_TENANT_NAME=demo
    export PASSWORD=admin
    export OS_AUTH_URL=http://127.0.0.1:5000/v2.0/
    export OS_REGION_NAME=scut
    
读入 openrc ::

    source openrc
    
验证效果 ::

    nova image-list

