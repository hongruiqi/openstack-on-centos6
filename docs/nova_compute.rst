其它计算节点配置
==========

安装 openstack-nova
----------

::

    yum install openstack-nova
    
网卡接口设置
----------

安装桥接管理软件 ::

    yum install bridge-utils

将网卡设置为混杂模式 ::

    ifconfig eth0 promisc
    
设置 selinux 为 permissive ::

    setenforce permissive
    
修改配置文件
----------

::

    [DEFAULT]
    # debug=True
    # verbose=True
    compute_scheduler_driver=nova.scheduler.filter_scheduler.FilterScheduler
    logdir = /var/log/nova
    state_path = /var/lib/nova
    lock_path = /var/lib/nova/tmp
    volumes_dir = /etc/nova/volumes
    iscsi_helper = tgtadm
    # 此处为管理节点主机 IP
    sql_connection = mysql://nova:nova@192.168.1.1:3306/nova
    compute_driver = libvirt.LibvirtDriver
    firewall_driver = nova.virt.libvirt.firewall.IptablesFirewallDriver
    #rpc_backend = nova.openstack.common.rpc.impl_qpid
    rootwrap_config = /etc/nova/rootwrap.conf
    libvirt_type = kvm
    # 本机 IP
    my_ip=192.168.1.253
    # 注意此处路径需存在，且对nova用户具有权限
    instances_path=/state/partition1/openstack/instance

    #AUTH
    auth_strategy = keystone
    # 管理节点
    rabbit_host=192.168.1.1
    glance_host=192.168.1.1
    api_paste_config=/etc/nova/api-paste.ini

    #NETWORK
    dhcpbridge = /usr/bin/nova-dhcpbridge
    dhcpbridge_flagfile = /etc/nova/nova.conf
    force_dhcp_release = False
    injected_network_template = /usr/share/nova/interfaces.template
    libvirt_nonblocking = True
    libvirt_inject_partition = -1
    network_manager = nova.network.manager.FlatDHCPManager
    #fixed_range=192.168.100.0/24
    #floating_range=192.168.1.0/24
    flat_network_bridge = br100
    public_interface=eth0
    flat_interface=eth0

    # 外网 IP
    novncproxy_base_url=http://202.38.192.97:6080/vnc_auto.html
    # 本机内部 IP
    vncserver_proxyclient_address=192.168.1.253
    vncserver_listen=192.168.1.253
    
    [keystone_authtoken]
    admin_tenant_name = service
    admin_user = nova
    admin_password = nova
    auth_host = 192.168.1.1
    auth_port = 35357
    auth_protocol = http
    signing_dir = /tmp/keystone-signing-nova
    
启动服务
----------

::

    service openstack-nova-compute restart

