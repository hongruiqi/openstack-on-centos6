配置 nova_volume
==========

根据文件创建虚拟设备 ::

    dd if=/dev/zero of=nova-volumes.img bs=10M count=1024
    vgcreate nova-volumes $(losetup --show -f nova-volumes.img)
    
启动服务 ::

    service tgt start        # 主节点
    service nova-volume start
    service open-iscsi start # 计算节点
