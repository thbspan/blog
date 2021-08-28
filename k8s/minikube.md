# minikube

## minikube安装

新版本的minikuba已经支持国内安装，可以通过添加参数`--image-mirror-country=cn`或`--image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'`来安装，更多详细的信息可以参考 `minikuba start --help`

## minikube使用

### Dashboard堆外暴露端口

Minikube启动Dashboard后默认只能localhost访问，如果需要开放外部访问需要添加一层代理

#### 具体操作方式

1. 启动Dashboard

   ``` shell
   minikuba dashboard
   ```

2. 使用proxy代理到指定的IP和端口

   ```shell
   kubectl proxy --port=8001 --address='本机IP' --accept-hosts='^.*' &
   ```

3. 启动后宿主机访问下面链接就可以看到 `dashboard`页面信息了

   > http://xxxx:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/overview?namespace=default
   >
   > 注意是http:kubernetes-dashboard，命令行提示中可能会是https，可以手动修改为http

## 参考链接

[minikuba get started](https://minikube.sigs.k8s.io/docs/start/)

[Minikube - Kubernetes本地实验环境](https://developer.aliyun.com/article/221687)

[Dashboard对外暴露访问链接](https://www.jianshu.com/p/da8b6725c010)