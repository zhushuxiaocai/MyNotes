# Kubevirt

* #### 环境准备

> 上传所需镜像 `chinaskills_cloud_paas_v2.0.1_merged.iso` ,确保`Kubernetes`集群正常运行,
>
> > * 初始化集群 `kubectl cluster-info`
> > * 检查集群节点 `kubectl get nodes`
> > * 检查CPU是否支持虚拟化 `egrep -c 'vmx|svm' /proc/cpuinfo`

* #### 部署KubeVirt环境

> 将镜像中的 Kubevirt.tar.gz 解压后 tools 文件夹中的 `virtctl-v0.47.1-linux-amd64` 复制到 /usr/bin/virtctl,查看版本 `virtctl version`

### _**VM资源清单文件模板**_

```yaml
apiVersion: kubevirt.io/v1  # API版本号
kind: VirtualMachine  # 对象类型，VM
metadata:  # 元数据
  creationTimestamp: null
  labels:  # 自定义标签的名称和值
    kubevirt-vm: vm-${NAME}
    kubevirt.io/os: fedora27
  name: ${NAME}  # VM名称
spec:
  running: false  # 决定是否启动VMI
  template:
    metadata:
      creationTimestamp: null
      labels:
        kubevirt-vm: vm-${NAME}
        kubevirt.io/os: fedora27
    spec:
      domain:
        cpu:
          cores: ${{CPU_CORES}}  # vCPUS数量
        devices:
          disks:
          - disk:  # 提供用户数据user-data和元数据meta-data
              bus: virtio
            name: cloudinitdisk
          - disk:  # 系统盘（必选）
              bus: virtio
            name: containerdisk
          interfaces:  # 网络接口
          - masquerade: {}
            model: virtio
            name: default
          networkInterfaceMultiqueue: true
          rng: {}
        resources:  # 资源请求
          requests:
            memory: ${MEMORY}
      networks:
      - name: default
        pod: {}  # 使用Kubernetes默认Pod网络
      terminationGracePeriodSeconds: 0
      volumes:
      - cloudInitNoCloud:  # 初始化操作
          userData: |-
            #cloud-config
            password: fedora
            chpasswd: { expire: False }
        name: cloudinitdisk
      - containerDisk:
          image: ${IMAGE}  # 系统镜像
        name: containerdisk
```

> 创建VM资源清单文件 `vi vm.yaml`

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  labels:
    kubevirt.io/vm: vm-fedora
  name: vm-fedora
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/vm: vm-fedora
    spec:
      domain:
        resources:
          requests:
            memory: 1Gi
        devices:
          disks:
          - name: containerdisk
            disk:
              bus: virtio
      volumes:
      - name: containerdisk
        containerDisk:
          image: kubevirt/fedora-cloud:v1.0
          imagePullPolicy: IfNotPresent
```

> * 创建VM `kubectl apply -f vm.yaml`
> * 查看VM / 详细信息 `kubectl get vm` / `kubectl describe vmi {name}`
> * 查看对应VMI `kubectl get vmi`
> * 启动VM `virtctl start {vmi-name}`
> * 停止VM `virtctl stop {vmi-name}`
> * 重启VM `virtctl restart {vmi-name}`
> * 删除VM `kubectl delete vmi {vmi-name}`
> * 进入VM `virtctl console {vmi-name}`
