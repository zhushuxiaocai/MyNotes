# Kubernetes

* #### 环境准备

> 由于默认仓库 k8s.gcr.io 被停止，所以我们使用 registry.k8s.io 仓库  , 拉取更新后的软件包，挂载到`mnt`目录 ，安装`kubeeasy`，通过`kubeeasy`安装`depend`依赖包，部署免秘钥
```
kubeeasy install depend \
  --host 192.168.1.100,192.168.1.101,192.168.1.102 \
  --user root \
  --password 000000 \
  --offline-file /mnt/dependencies/base-rpms.tar.gz
```
```
kubeeasy check ssh \
  --host 192.168.1.100,192.168.1.101,192.168.1.102 \
  --user root \
  --password 000000
```

```
kubeeasy create ssh-keygen \
  --master 192.168.1.100 \
  --worker 192.168.1.101,192.168.1.102 \
  --user root \
  --password 000000
```

* #### 部署kubernetes集群

> 通过kubeeasy部署kubernetes_**注意要标注版本为1.22.1**_ , 
```
kubeeasy install kubernetes \
  --master 192.168.1.100 \
  --worker 192.168.1.101,192.168.1.102 \
  --user root \
  --password 000000 \
  --version 1.22.1 \
  --pod-cidr 192.168.1.0/24 \
  --offline-file ./kubernetes.tar.gz
```
> 通过`kubectl cluster-info`查看集群状态，通过`kubectl top nodes --use-protocol-buffers`查看节点负载情况，客户端`master_ip:30080`访问页面，默认登录账号为`admin` 密码为`Kuboard123` ,完成搭建后。使用nginx镜像在default命名空间下创建一个名为`exam`的`pod`，设置环境变量为`exam`，值为`2022`，加载本地nginx镜像，编写`pod.yaml`文件:

```yaml
apiVersion: v1                   #使用 Kubernetes API 的版本
kind: Pod                          #对象的类型
metadata:                         #Pod 相关的元数据
  name: exam                    # Pod 的名称，是唯一的
spec:                                 # Pod 的规范部分，配置和行为
  containers:                      #包含一个或多个容器的列表，将在Pod中运行
  - name: nginx                  #容器的名称
    image: nginx                 #使用的容器镜像
    imagePullPolicy: IfNotPresent            #拉取容器镜像策略，本地不存在镜像时尝试拉取
    env:                             #包含容器环境变量的列表，定义了环境变量
    - name: "exam"            #包含容器环境变量的列表
      value: "2022"             #环境变量的值
```

> 通过`kubectl apply -f pod.yaml`创建，`kubectl cluster-info`查看集群状态，`kubectl exec exam -- printenv`查看pod

* #### 部署Kubevirt

> `cp /mnt/kubevirt.tar.gz /root/`
> 
> kubeeasy add --virt kubevirt **#kubeeasy add --名 值**
>
> > 查看 `kubectl -n kubevirt get pods`
> >
> > > * 创建vmi `kubectl create -f vmi.yaml`
> > > * 查看vmi `kubectl get vmi`
> > > * 删除vmi `kubectl delete vmis <vmi-name>`

> > > * 启动虚拟机 `virtctl start <vmi-name>`
> > > * 停止虚拟机 `virtctl stop <vmi-name>`
> > > * 重启虚拟机 `virtctl restart <vmi-name>`

> > 查看 `kubectl -n kubevirt get deployment` **\`\`-n **_**指定命名空间**_ **`kubevirt`** _**命名空间**_ **`get`** _**获取或列出信息**_ **`deployment`** _**资源类型获取的是部署deployment类型的资源**_

* #### 部署istio

> `cp istio.tar.gz /opt/`
> 
> kubeeasy add --istio istio安装，kubectl -n istio-system get pods查看pod，通istioctl version查看istio版本，master\_ip:33000访问grafana，:30090访问prometheus，:30686访问jaeger，:20001访问kiali，

```yaml
istioctl profile list          #istio配置档的名称
istioctl profile dump demo      #配置档的配置信息
istioctl profile diff default demo     #配置文件的差异
istioctl proxy-status，istioctl ps    #概览服务网络，缺少某代理-当前未连接到polit实例，stale-存在网络问题或需要扩展pilot
istioctl proxy-config cluster <pod-name> [flasgs]      #检索特定Pod中Envoy实例的集群配置的信息
istioctl proxy-config bootstrap <pod-name> [flags]    #检索特定Pod中Envoy实例的bootstrap配置的信息
istioctl proxy-config listener <pod-name> [flags]     #检索特定Pod中Envoy实例的监听器配置的信息
istioctl proxy-config route <pod-name> [flags]        #检索特定Pod中Envoy实例的路由配置的信息
istioctl proxy-config endpoints <pod-name> [flags]     #检索特定Pod中Envoy实例的endpoint配置的信息
```

> 完成istio安装后，新建命名空间exam,开启自动注入Sidecar
>
> > * `kubectl create namespace exam`
> > * `kubectl label namespace exam istio-injection=enabled`

> 查看在istio-system空间中所有资源 `kubectl -n istio-system get all` 查看exam命名空间，显示标签 `kubectl get ns exam --show-labels`

* #### 部署harbor

>`cp harbor-offline.tar.gz /opt/`
>
> `kubeeasy add --registry harbor` 安装并查看状态 master\_ip访问

```yaml
helm version        #查看版本
helm list           #查看当前安装的Charts
helm search <chart-name>            #查询Charts
helm status redis           #查看charts状态
helm delete --purge <chart-name>        #删除charts
helm create helm_charts          #创建charts
helm lint          #测试charts语法
cd helm_charts && helm package ./          #打包charts
helm template helm_charts-xxx.tgz         #查看生成的yaml文件
```

> 完成harbor安装后，使用nginx自定义一个chart,deployment名称为nginx，副本为1，chart部署到default命名空间下，release名称为web
>
> > * helm create mychart **创建**
> > * rm -rf mychart/templates/\* **清空模板文件**

vi mychart/templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

> * `helm install web mychart` **`mychart` 要安装的名称，`web` 新安装创建的k8s资源名称**
> * `helm status web` **查看`web`信息**

* #### 基础运维

如果安装过程中失败，可以重置集群 `kubeeasy reset`

> * 在集群中添加节点 `kubeeasy install depend --host new_IP --user root --password Abc@1234 --offline-file /opt/depend.tar.gz`
> * 加入集群 `kubeeasy add --worker new_IP --user root --password Abc@1234 --offline-file /opt/kubernetes.tar.gz`
