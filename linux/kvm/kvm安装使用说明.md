# kvm 安装使用说明（Cento 7）

## 一、安装kvm

## 1、检查cpu是否支持虚拟化
    egrep '(vmx|svm)' /proc/cpuinfo
    如果有vmx信息输出，就说明支持VT;如果没有任何的输出，说明你的cpu不支持，将无法使用KVM虚拟机

## 2、确保BIOS中开启了虚拟化功能
    lsmod | grep kvm

## 3、安装
    yum install qemu-kvm libvirt virt-install bridge-utils -y

## 4、启动kvm服务
    systemctl start libvirtd

## 5、查看kvm启动状态
    systemctl status libvirtd

## 二、配置桥接网络

### 1、可以通过命令brctl创建桥接网络

### 2、通过创建配置文件的方式创建桥接网络

    在 /etc/sysconfig/network-scripts/目录下创建ifcfg-br0文件，内容如下（可以参考默认网卡配置）
    BOOTPROTO=static
    DEVICE=br0
    TYPE=Bridge
    NM_CONTROLLED=no
    IPADDR=192.168.101.100
    NETMASK=255.255.255.0
    GATEWAY=192.168.101.1
    DNS1=114.114.114.114
    DNS2=8.8.8.8

    修改默认网卡配置文件，改为桥接方式
    BOOTPROTO=none
    DEVICE=enp0s25
    NM_CONTROLLED=no
    ONBOOT=yes
    BRIDGE=br0

### 重启网络服务
    systemctl restart network

    使用ifconfig验证是否多了一块网卡

## 三、安装虚拟机

### 1、准备好系统镜像

### 2、使用virt-install命令创建虚拟机
    具体用户可以通过 virt-install --help查看，有中文说明

### 3、根据配置的vncport端口，打开防火墙上对应的端口
    firewall-cmd --zone=public --add-port=xxxx/tcp --permanent
    firewall-cmd --reload

### 4、使用vnc Viewer连接虚拟机，进行系统安装相关的工作


