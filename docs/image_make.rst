附A：安装镜像制作
==========

Windows 7
----------

创建虚拟机磁盘 ::

    qemu-kvm create -f qcow2 win7.img 10g

下载虚拟机驱动 ::

    wget -c \
        http://alt.fedoraproject.org/pub/alt/virtio-win/latest/images/bin/virtio-win-0.1-52.iso
    
启动虚拟机 ::

    qemu-kvm -m 1024 -cdrom windows7.iso \
        -drive file=win7.img,if=virtio \
        -drive file=virtio-win-0.1-52.iso,index=3,media=cdrom \
        -net nic,model=virtio \
        -net user 
        -nographic -vnc :0
    
其中 `windows7.iso` 为 Win7 安装镜像地址，需从别处下载。然后使用 `vncviewer` 访问相应 `-vnc` 选项端口进行系统安装。
安装时，需先加载相应的虚拟驱动，否则无法看到磁盘。
安装后将 `win7.img` 上传即可。 

::

    glance image-create --name="Win7" \
        --is-public=true \
        --container-format=bare \
        --disk-format=qcow2 < win7.img
    
Ubuntu
----------

创建虚拟磁盘命令同上，无需下载和加载虚拟机驱动镜像，加载安装镜像运行即可 ::

    qemu-kvm -m 1024 -cdrom ubuntu.iso \
        -drive file=ubuntu.img,if=virtio \
        -net nic,model=virtio \
        -net user \
        -nographic -vnc :0

