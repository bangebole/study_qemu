## 1.介绍
* 学习qemu的使用

## 2.创建盘
* ```qemu-image create debian.img 2G```
* ```qemu-image create -f qcow2 debian.qcow 2G```
* -f qcow2:qcow2镜像格式是QEMU模拟器支持的一种磁盘镜像。它也是可以用一个文件的形式来表示一块固定大小的块设备磁盘。与普通的镜像相比有一下特性:
> * 更小的空间占用,及时文件系统不支持空洞;
> * 支持写时拷贝(copy-on-write),镜像文件只反应底层磁盘的变化;
> * 支持快照(snaoshot),镜像文件能够包含多个快照历史;
> * 可选择基于zlib的压缩方式;
> * 可以选择AES加密
* ```dd if=/dev/zero of=rootfs.img bs=1M count=1024```
* ```mkfs.ext4 rootfs.img```

## 3.通过iso安装系统
* ```qemu-system-x86_64 -hda debian.img -cdrom debian-tesing-amd64-netinit.iso -boot d -m 512```
* -m 512:运行内存
* -enabel-kvm:启动kvm加速
## 4.运行系统
* ```qemu-system-x86_64 -hda debian.img -m 512```

## 5.配置网络
### (1)NAT
* NAT:默认情况下,qemu调用-nic和-user选项将单个网络适配器添加到虚拟机并提供NAT外部网络访问
### (2)桥接
* 1.安装桥接工具:```sudo apt-get install bridge-utils```
* 2.修改宿主系统配置:
```
#/etc/network/interfaces
#以下是之前未添加网桥时的配置
    # The primary network interface
    #auto enp3s0
    #iface enp3s0 inet static
    #       address 192.168.66.149
    #       netmask 255.255.255.0
    #       network 192.168.66.0
    #       broadcast 192.168.66.255
    #       gateway 192.168.66.1
    #       dns-nameservers 114.114.114.114
    #       dns-search foolsky
#添加网桥br0
auto br0
        iface br0 inet static  #之前上网时采用静态IP，所以这里依然使用此
        address 192.168.66.149 #将之前上网的ip地址分配给网桥。
        network 192.168.66.0
        netmask 255.255.255.0
        broadcast 192.168.66.255
        gateway 192.168.66.1
        bridge_ports enp3s0 tap0 #为网桥添加两个接口，分别是enp3s0（之前默认的上网网口）和tap0
        bridge_stp off
        bridge_fd 0
        bridge_maxwait 0
        dns-nameservers 114.114.114.114
#添加接口enp3s0,上网方式采用自动
    auto enp3s0
    iface eth0 inet manual
```
* 3.启动:```sudo qemu-system-x86_64 -hda debian.qcow -net nic -net tap,ifname=tap0,script=no,downscript=no```

## 6.参考资料
* **qcow2文件格式详解(1):**<https://www.jianshu.com/p/d087d3a566f4>
* **ubuntu下使用qemu安装虚拟机并配置桥接网络:**<https://blog.csdn.net/u010817321/article/details/52117344>
