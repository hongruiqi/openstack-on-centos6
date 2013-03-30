配置 Hypervisor —— KVM为例
==========

检查硬件虚拟化支持
----------

::

    egrep '(vmx|svm)' --color=always /proc/cpuinfo
    
若无结果返回，说明硬件不支持虚拟化，不能使用 KVM，需使用其它虚拟化技术。

检查 KVM 模块加载
----------

::

    lsmod |grep kvm
    
若模块未加载

::

    modprobe kvm
    modprobe kvm-intel # for intel cpu
    modprobe kvm-amd   # for amd cpu
    
修改 nova 配置文件
----------
    
**/etc/nova/nova.conf** ::
    
    compute-driver=libvirt.LibvirtDriver
    libvirt_type=kvm
