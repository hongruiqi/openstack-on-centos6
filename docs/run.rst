运行虚拟机
==========

注册虚拟机镜像
----------

::

    wget -c https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img \
        -O cirros.img
    glance image-create \
        --name=cirros-0.3.0-x86_64 \
        --disk-format=qcow2 \
        --container-format=bare < cirros.img
        
设置安全组
----------

::

    nova secgroup-add-rule default tcp 22 22 0.0.0.0/0  # 开放 22 端口
    nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0 # 开放 ping
    
添加密钥
----------

::

    nova keypair-add --pub_key ~/.ssh/id_rsa.pub key # 上传 ssh 访问公钥
    
创建虚拟机
----------

::

    nova boot --flavor 2 --image <cirros的image-id> \
        --key_name=key --security_group default cirros
    
查看虚拟运行日志
----------

::

    nova console-log
