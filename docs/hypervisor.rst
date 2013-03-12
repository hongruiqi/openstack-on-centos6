配置 Hypervisor —— KVM为例
==========

检查 KVM 模块加载
----------

::

    lsmod |grep kvm
    
修改 nova 配置文件
----------
    
**/etc/nova/nova.conf** ::
    
    compute-driver=libvirt.LibvirtDriver
    libvirt_type=kvm
