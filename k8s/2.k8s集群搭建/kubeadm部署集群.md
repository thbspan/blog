# kubeadmin部署集群

## 部署机器要求

- 一台或多台机器或虚拟机，操作系统 Centos 7 或 Ubuntu Server
- 硬件配置：2GB内存或更多、硬盘30GB或更多
- 集群中所有机器之间可以网络互通
- 可以访问网络，需要安装软件和拉取镜像
- 警用swap分区

## 部署

### Ubuntu Server部署

#### 安装docker

> 可以参考官方文档 https://docs.docker.com/engine/install/ubuntu/

安装完成后，设置docker引擎的cgroup driver为systemd

修改 /etc/docker/daemon.json

``` json
{
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ],
  "registry-mirrors": [
    "https://73x7zsf7.mirror.aliyuncs.com"
  ]
}
```

#### 安装k8s

可以参考官方文档：https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

0. 屏蔽系统休眠

    ``` shell
    # 查询状态
    systemctl status sleep.target
    # 禁启用
    sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
    # 最后再次检测状态
    systemctl status sleep.target
    ```

1. 关闭防火墙

   ``` shell
   sudo ufw disable
   ```

2. 关闭swap

   ``` shell
   # 临时关闭
   sudo swapoff -a
   # 永久关闭
   sudo sed -ri 's/.*swap.*/#&/' /etc/fstab
   ```

3. 设置主机名

   ``` shell
   hostnamectl set-hostname <hostname>
   ```

4. 在hosts文件中添加主机名和IP映射

   ```
   10.0.3.4 spark-master
   10.0.3.5 spark-slave1
   10.0.3.6 spark-slave2
   ```

5. 将桥接的ipv4流量传递到iptables

   ``` shell
   cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
   br_netfilter
   EOF
   
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   
   sudo sysctl --system  # 生效
   ```

6. 同步时间

   ```shell
   sudo apt install ntpdate
   # 将系统时间与网络同步
   sudo ntpdate cn.pool.ntp.org
   # 将时间写入硬件
   sudo hwclock --systohc
   ```

7. 安装 kubectl kubeadm kubelet

   ``` shell
   # 更新包索引并安装依赖
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl
   
   # 下载 Google Cloud 公开签名秘钥（这里修改为阿里的）
   sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg
   
   # 添加 Kubernetes apt 仓库：
   echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
   
   # 更新 apt 包索引
   sudo apt-get update
   # 安装指定的版本，使用命令 apt-cache madison kubectl 可以查看软件可以安装的所有版本
   sudo apt-get install -y kubelet<=version> kubeadm<=version> kubectl<=version>
   # sudo apt-get install -y kubelet=1.21.4-00 kubeadm=1.21.4-00 kubectl=1.21.4-00
   # 锁定软件版本，阻止软件更新
   sudo apt-mark hold kubelet kubeadm kubectl
   
   # 设置开机自启动
   sudo systemctl enable kubelet
   ```

8. 部署k8s Master节点

   ``` shell
   sudo kubeadm init --apiserver-advertise-address=10.0.3.4 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.21.4 --service-cidr=172.16.0.1/20 --pod-network-cidr=192.168.0.1/16
   
   --apiserver-advertise-address： API 服务器所公布的其正在监听的 IP 地址。如果未设置，则使用默认网络接口
   
   --image-repository： 指定阿里镜像仓库
   
   --kubernetes-version=v1.21.4   这个参数是下载的k8s软件版本号
   
   --service-cidr=172.16.0.1/20   service-cidr 的选取不能和PodCIDR及本机网络有重叠或者冲突
   
   --pod-network-cidr=192.168.0.1/16   k8s内部的pod节点之间网络可以使用的IP段
   
   
   安装失败，执行以下命令，重新安装：sudo kubeadm reset
   如果是报某些镜像找不到，可以参考一下解决方案：
   例如报错：
   kubeadm init：failed to pull image coredns:v1.8.0: error
   解决方法：
   sudo docker pull coredns/coredns:1.8.0
   
   sudo docker tag coredns/coredns:1.8.0 registry.aliyuncs.com/google_containers/coredns:v1.8.0
   
   # 拉取镜像
   sudo kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.21.4
   
   # 执行成功后会打印加入的命令，也可以通过下面的命令重新生成
   sudo kubeadm token create --print-join-command
   ```

9. 配置运行kubectl

   ``` shell
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   # 查看节点：
   kubectl get nodes
   ```

10. 加入其他的kube node节点

   ``` shell
   # Master 节点执行sudo kubeadm init后会生成其他node加入的命令
   # sudo kubeadm token create --print-join-command 可以重新查看加入命令
   # kubeadm join 10.0.3.4:6443 --token 2sa7gn.gxisejpfi6g4sot3 --discovery-token-ca-cert-hash sha256:f31ac15fa82a1088c213f6c70660c2eba0698af6036401efc80766835f82a8c1
   ```

11. Master部署CNI网络插件

``` shell
kubectl get node
```

    上面的状态还是NotReady，下面我们需要网络插件，来进行联网访问

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

