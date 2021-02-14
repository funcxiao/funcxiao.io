+++
title = "linux虚拟化——qemu和libvirt"
description = "qemu和libvirt的简单配置使用"
draft = false
[taxonomies]
tags = ["linux", "qemu","virtualization"]
[extra]
feature_image = "qemu_logo.jpeg"
feature = true
link = ""
+++

![virsh](raw_sc.avif)

qemu,libvirt,kvm
******

## 概述

[QEMU](https://www.qemu.org)是啥？
> QEMU is a generic and open source machine emulator and virtualizer.

![qemu_logo](qemu_logo.jpeg)
从官网上介绍来看就是模拟器和虚拟机，可以在一种架构下运行另一种架构的程序(如在X86上模拟RISC-V),也可以使用其他虚拟机管理程序(hypervisors)来运用处理器扩展(HVM)进行虚拟化，比如使用KVM和XEN

[KVM](http://www.linux-kvm.org)又是啥？
> KVM (for Kernel-based Virtual Machine) is a full virtualization solution for Linux on x86 hardware containing virtualization extensions (Intel VT or AMD-V).

![kvm_logo](kvm_logo.png)
KVM是Linux的一部分，并且使用常规的Linux调度和内存管理。KVM使用的是处理器扩展来虚拟化，不支持XEN那样的半虚拟化

[libvirt](https://libvirt.org)又又是啥？
> The libvirt project:is a toolkit to manage virtualization platforms;is accessible from C, Python, Perl, Go and more;supports KVM, QEMU, Xen, Virtuozzo, VMWare ESX, LXC, BHyve and more.

![libvirt_logo](libvirt_logo.png)
Libvirt是一组工具集，提供了虚拟化平台的管理以及一套管理虚拟化相关功能的API

![qemu_vm](qemu_vm.png)
用人话总结一下，大概就是KVM负责搞定CPU这块，QEMU模拟其他硬件，这两个就已经可以用得很好了，然后加上libvirt方便管理。

## 安装使用

操作都是在Manjaro下进行

### 用到的包

相关安装包照着`ArchWiki`上说的来的，相关组件如下：

- `qemu`包提供`X86_64`架构模拟器，可以进行全系统模拟(`qemu-system-x86_64`)
- `qemu-arch-extra`包提供了`x86_64`用户模式模拟(`qemu-x86_64`)和其他架构的全系统模拟和用户模拟,当然我安装它主要是为了支持其他架构，比如`arm`和`risc-v`
- `qemu-img`包提供qemu的磁盘镜像管理工具
- `libvirt`包提供libvirt的基础工具，包含了API和守护进程(libvirtd)以及命令行工具(virsh)
- `virt-install`包提供命令行工具来创建`Domains`(也就是虚拟机)
- `virt-manager`包提供了一个较简单的libvirt图形化客户端，图形客户端还有其他的选择
- `ebtables`和`dnsmasq`包用于默认的NAT/DHCP网络
- `bridge-utils`包用于桥接网络
- `openbsd-netcat`包通过SSH来远程管理

### 后续配置

首先安装上面提到的包：

```shell
$yay -R qemu qemu-arch-extra qemu-img libvirt virt-install virt-manager ebtables dnsmasq bridge-utils openbsd-netcat
```

#### 配置KVM启用嵌套虚拟化

```shell
# 检测硬件是否支持虚拟化，不支持就不用看了
$LC_ALL=C lscpu | grep Virtualization
# 查看内核模块是否已经加载，没有就modprobe手动加载一下
$lsmod |grep kvm
# 启动kvm_intel模块嵌套虚拟化功能
$modprobe kvm_intel nested=1
# modprobe只是临时加载，所以添加参数到conf文件去：
$sudo tee /etc/modprobe.d/kvm-intel.conf <<< "options kvm_intel nested=1"
# 查看嵌套虚拟化是否激活
$systool -m kvm_intel -v | grep nested
```

#### 配置普通用户使用授权

如果使用的非root用户使用libvirt需要口令可以进行如下配置

##### libvirt使用`polkit`授权

修改授权以读写模式访问socket的组，如想授权kvm组，可创建文件`/etc/polkit-1/rules.d/50-libvirt.rules`：

```conf
/* Allow users in kvm group to manage the libvirt daemon without authentication
*/
polkit.addRule(function(action, subject) {
    if (action.id == "org.libvirt.unix.manage" &&
        subject.isInGroup("kvm")) {
            return polkit.Result.YES;
    }
});
```

然后添加用户到`kvm`组重新登录生效

##### libvirt基于文件的授权

取消`/etc/libvirt/libvirtd.conf`文件中对应注释，给libvirt用户定义基于文件的权限以管理虚拟机：

```conf
#unix_sock_group = "libvirt"
#unix_sock_ro_perms = "0777"  
#unix_sock_rw_perms = "0770"
#auth_unix_ro = "none"
#auth_unix_rw = "none"
```

```shell
# 创建并加入libvirt用户组
$newgrp libvirt
$sudo usermod -aG libvirt $USER
$sudo systemctl restart libvirtd
```

## 使用实践

### 使用`Fedora CoreOS`镜像创建虚拟机

```shell
# 安装coreos-installer
$cargo install coreos-installer
# 下载对应qemu的镜像
$export STREAM="stable"
$coreos-installer download -s "${STREAM}" -p qemu -f qcow2.xz --decompress -C ~/.local/share/libvirt/images/
# 下载coreos配置转换工具fcct
$wget https://github.com/coreos/fcct/releases/download/v0.10.0/fcct-x86_64-unknown-linux-gnu
```

创建配置文件`myconf.fcc`：

```fcc
variant: fcos
version: 1.2.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa ......
```

生成`ignition`配置文件,并安装：

```shell
$./fcc --pretty --strict < myconf.fcc > myconf.ign
# 使用virt-install创建虚拟机
virt-install --connect="qemu:///system" --name="fcos-test-01" --vcpus="2" --memory="4096" \
        --os-variant="fedora-coreos-stable" --import --graphics=none \
        --disk "path=/home/xxxx/myStorage/myVirt/fcos-test-01.qcow2,size=20,backing_store=/home/xxxx/myStorage/images/fedora-coreos-33.20210117.3.2-qemu.x86_64.qcow2" \
        --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/home/xxxx/myConf/coreos/myconf.ign"
```

如果运行结果正常，那么虚拟机就创建成功了，接下来通过`SSH`连接上去(也可以使用virt-manager来管理)：

```shell
# 查看所有虚拟机
$virsh list --all
# 查看虚拟机fcos-test-01的ip
$virsh domifaddr fcos-test-01  --source arp
# coreos默认不使用密码登录，之前已经配置过公钥了
$ssh core@<ip address>
# virsh关机
$virsh shutdown <name>
# 强制关机
$virsh destroy <name>
# 挂起
$virsh suuspend <name>
# 恢复
$virsh resume <name>
# 重启
$virsh reboot <name>
# 更多看virsh -h
```

依据上面的步骤，再创建两个，留着给后面用。

### qemu创建一个risc-v虚拟机

使用`qemu-system-riscv64`创建即可，后面再详细记录

### 使用`OSX-KVM`创建osx虚拟机

具体参考[OSX-KVM](https://github.com/kholia/OSX-KVM)
装完了可以正常运行，不过我的是N卡，`appleid`还登录不上，暂时就搁置了

## 参考

- [QEMU ArchWiki](https://wiki.archlinux.org/index.php/QEMU)
- [KVM ArchWiki](https://wiki.archlinux.org/index.php/KVM)
- [Libvirt ArchWiki](https://wiki.archlinux.org/index.php/Libvirt)
- [QEMU-KVM 虚拟化环境的搭建与使用
](https://ryan4yin.space/qemu-kvm-usage)
- [KVM/QEMU/qemu-kvm/libvirt 概念全解
](https://blog.csdn.net/Jmilk/article/details/68947277)
