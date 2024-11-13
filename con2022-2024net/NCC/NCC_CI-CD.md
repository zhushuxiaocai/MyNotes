# 基于 Kubernetes 构建 CI/CD 持续集成

## 本篇来源于 华为云社区 https://bbs.huaweicloud.com/blogs/413637

```
部署Kubernetes,上传软件包 `BlueOcean.tar.gz`
```

* #### 部署 Harbor

> 1. 解压 BlueOcean.tar.gz , 安装docker-compose 
>
> > 1. `tar -zxvf BlueOcean.tar.gz` 
> > 2. `cp BlueOcean/tools/docker-compose-Linux-x86_64 /usr/bin/docker-compose`  
> > 3. `docker-compose version`  

> 2. 安装 Harbor , 默认账号密码 `admin` `Harbor12345` , `http://master_ip`  
>
> > 1. tar -zxf BlueOcean/harbor-offline-installer.tar.gz -C /opt/  
> > 2. sh /opt/harbor/install.sh
> > 3. 登录后新建 `springcloud` , 访问级别设置为公开
> > 4. 上传镜像到 Harbor 
> > >  - docker login -uadmin -pHarbor12345 IP 
> > >  - docker load -i BlueOcean/images/maven_latest.tar  
> > >  - docker tag maven IP/library/maven 
> > >  - docker push IP/library/maven
> > >  - docker load -i BlueOcean/images/java_8-jre.tar  
> > >  - docker load -i BlueOcean/images/jenkins_jenkins_latest.tar   
> > >  - docker load -i BlueOcean/images/gitlab_gitlab-ce_latest.tar

* #### 部署 Jenkins

> 1. 安装 Jenkins  
>
> > 1. 新建命名空间 `kubectl create ns devops`
> > ```
> > 部署Jenkins需要使用到一个拥有相关权限的serviceAccount，名称为jenkins-admin，可以给jenkins-admin赋予一些必要的权限，也可以直接绑定一个cluster-admin的集群角色权限，此处选择给予集群角色权限。
> > ```
> > 2. 编写资源清单文件 `vi jenkins-deploy.yaml `
```yaml
apiVersion: v1   
kind: Service  
metadata: 
  name: jenkins
  labels:
    app: jenkins
spec:
  type: NodePort
  ports:
  - name: http
    port: 8080
    targetPort: 8080
    nodePort: 30880
  - name: agent
    port: 50000
    targetPort: agent
    nodePort: 30850
  selector:
    app: jenkins

---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: jenkins
  labels:
    app: jenkins
spec:
  selector:
    matchLabels: 
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccountName: jenkins-admin
      containers:
      - name: jenkins
        image: jenkins/jenkins:latest 
        imagePullPolicy: IfNotPresent
        securityContext: 
          runAsUser: 0
          privileged: true
        ports:
        - name: http
          containerPort: 8080
        volumeMounts:
        - mountPath: /var/jenkins_home
          name: jenkinshome
        - mountPath: /usr/bin/docker
          name: docker
        - mountPath: /var/run/docker.sock
          name: dockersock
        - mountPath: /usr/bin/kubectl
          name: kubectl
        - mountPath: /root/.kube
          name: kubeconfig
      volumes:
      - name: jenkinshome
        hostPath:
          path: /home/jenkins_home
      - name: docker
        hostPath:
          path: /usr/bin/docker
      - name: dockersock
        hostPath:
          path: /var/run/docker.sock
      - name: kubectl
        hostPath: 
          path: /usr/bin/kubectl
      - name: kubeconfig
        hostPath:
          path: /root/.kube 
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkinshome 
  annotations:
    volume.beta.kubernetes.io/storage-class: local-path
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1024Mi 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin 
  labels:
    name: jenkins
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata: 
  name: jenkins-admin
  labels:
    name: jenkins
subjects: 
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin 
  apiGroup: rbac.authorization.k8s.io 
```

> > 3. `kubectl -n devops apply -f jenkins-deploy.yaml` 
> > 4. `kubectl -n devops get pods`
> > 5. `kubectl -n devops cp BlueOcean/plugins/ jenkins-cc97fd4fc-v5dh2:/var/jenkins_home`
> > 6. `kubectl -n devops rollout restart deployment jenkins`
> 2. 访问 Jenkins
> > 1.  查看 Jenkins Service 端口 `kubectl -n devops get svc`
> > 2.  Web 访问 http://master_IP:30880
> > 3.  获取 Jenkins 密码 `kubectl -n devops exec deploy/jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword`
> > 4.  输入密码后单击“继续”按钮,选择“跳过插件安装”,如图：
> > > ![插件安装](https://bbs-img.huaweicloud.com/blogs/img/20231024/1698142010333732951.png)
> > 6.  安装完成后进入用户创建页面，创建一个Username 为 jenkins，密码为 000000 ，其他项随便填 , 如图：
> > > ![创建账户](https://bbs-img.huaweicloud.com/blogs/img/20231025/1698229461407799933.JPG)
> > > ![创建账户](https://bbs-img.huaweicloud.com/blogs/img/20231024/1698142491379952155.JPG)
> > > 
> > 7.单击“开始使用 Jenkins ”按钮并使用新创建的用户登录 Jenkins

*  #### 部署 Gitlab

> ```
> GitLab是利用Ruby on Rails一个开源的版本管理系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。与GitHub类似，GitLab能够浏览源代码，管理缺陷和注释，可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库，团队成员可以利用内置的简单聊天程序（Wall）进行交流。GitLab还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。本项目GitLab与Harbor共用一台服务器。
> ```
> 1. 编写 Gitlab 清单文件 `vi gitlab-deploy.yaml `
```yaml
apiVersion: v1
kind: Service
metadata:
  name: gitlab
spec:
  type: NodePort
  ports:
  - port: 443
    nodePort: 30443
    targetPort: 443
    name: gitlab-443
  - port: 80
    nodePort: 30888
    targetPort: 80
    name: gitlab-80
  selector:
    app: gitlab
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
spec:
  selector:
    matchLabels:
      app: gitlab
  revisionHistoryLimit: 2
  template:
    metadata:
      labels:
        app: gitlab
    spec:
      containers:
      - image: gitlab/gitlab-ce:latest
        name: gitlab
        imagePullPolicy: IfNotPresent
        env:
        - name: GITLAB_ROOT_PASSWORD  
          value: admin@123  # 此处为root用户对应的密码
        - name: GITLAB_PORT
          value: "80"
        ports:
        - containerPort: 443
          name: gitlab-443
        - containerPort: 80
          name: gitlab-80
```

> 2. 部署 Gitlab `kubectl -n devops apply -f gitlab-deploy.yaml`
> 3. 查看 Pods `kubectl -n devops get pods`
> 4. 查看 Gitlab Service `kubectl -n devops get svc`
> 5. 因为 Gitlab 启动较慢 大致 1-3 分钟 , 查看启动状态 `kubectl logs`,完成后通过 Web 访问 `http://master_IP:30888` , 默认 用户名为 `root` 密码为 `admin@123`
> 6. 单机 `New project` 按钮 , 单击 `Create blank project` 按钮创建项目 `springcloud` ，可见等级选择`Public` , 单机 `Create project` 后进入项目 , `push` 源代码到`springcloud` 项目 :
> > - cd BlueOcean/springcloud/
> > - git config --global user.name "administrator"
> > - git config --global user.email "admin@example.com"
> > - git remote remove origin
> > - git remote add origin http://192.168.1.100:30888/root/springcloud.git 
> > - git add .
> > - git commit -m "initial commit"
> > ```
> > # On branch master
> > nothing to commit, working directory clean
> > ```
> > - git push -u origin master
> > ```
> > Username for 'http://192.168.1.100:30888': root
> > Password for 'http://root@192.168.1.100:30888': 
> > Counting objects: 3192, done.
> > Delta compression using up to 8 threads.
> > Compressing objects: 100% (1428/1428), done.
> > Writing objects: 100% (3192/3192), 1.40 MiB | 1.70 MiB/s, done.
> > Total 3192 (delta 1233), reused 3010 (delta 1207)
> > remote: Resolving deltas: 100% (1233/1233), done.
> > remote: 
> > remote: To create a merge request for master, visit:
> > remote:   http://gitlab-6778c45f9-xx5gs/root/springcloud/-/merge_requests/new?merge_request%5Bsource_branch%5D=master
> > remote: 
> > To http://192.168.1.100:30888/root/springcloud.git
> >  * [new branch]      master -> master
> > Branch master set up to track remote branch master from origin.
> > ```
> 7. 配置 `Jenkins` 连接 `Gitlab` , 配置 `Outbound requests` , `http://master_IP:30888/admin` , 在左侧导航栏选择 `Settings→Network` ，设置 `Outbound requests` ，勾选 `Allow requests to the local network from web hooks and services` 复选框 , 保存.
> 8. 创建 `GitLab API Token` , 单击 `GitLab` 用户头像图标 , 在左侧导航栏选择 `Preferences` , 在左侧导航栏选择`Access Tokens`添加 Token , 单击`Create personal access token`按钮生成 Token , 记录 Token 备用 .
> 9. 配置 `Jenkins` , 登录 `Jenkins`首页，选择 `系统管理→系统配置` ，配置 `GitLab` 信息，取消勾选 `Enable authentiviion for ‘/project’ end-point` ，输入 `Connection name”和“Gitlab host URL` , 添加 `Credentials` ，单击 ` “添加”→“Jenkins” ` 按钮添加认证信息，将 `Gitlab API Token` 填入 , 选择`新添加的证书` ，然后单击 `Test Connection` 按钮 , 返回结果为 `Success` ，说明 `Jenkins` 可以正常连接 `GitLab` .

*  #### Jenkinsfile

> 1. 登录 `Jenkins` 首页，新建任务 `springcloud` ，任务类型选择 `流水线` , 单击 `确定` 按钮，配置`构建触发器` , 记录下 `GitLab webhook URL` 的地址`  http://192.168.1.100:30880/project/springcloud  `  ，后期配置 `webhook` 需要使用 , 配置 `流水线` ，在定义域中选择 `Pipeline script from SCM` ，此选项指示 Jenkins从源代码管理（SCM）仓库获取流水线。在SCM域中选择 `Git` ，然后输入 `Repository URL` ，在 `Credentials` 中选择 `添加` ，凭据类型选择 `Username with password` ，然后输入对应信息 , 单击 `保存` 按钮，回到流水线中，在 `Credentials` 域选择刚才添加的凭证 , 保存.

> 2. 编写流水线 , `Pipeline` 有两种创建方法——可以直接在Jenkins的Web UI界面中输入脚本；也可以通过创建一个 `Jenkinsfile` 脚本文件放入项目源码库中。一般推荐在 `Jenkins` 中直接从源代码控制（SCMD）中直接载入 `Jenkinsfile Pipeline` 这种方法。登录 `GitLab` 进入 `springcloud` 项目，选择新建文件 ， 将流水线脚本输入到 `Jenkinsfile` 中 .  Pipeline 包括声明式语法和脚本式语法。声明式和脚本式的流水线从根本上是不同的。声明式是 Jenkins 流水线更友好的特性。脚本式的流水线语法，提供更丰富的语法特性。声明式流水线使编写和读取流水线代码更容易设计。   此处选择声明式Pipeline , 代码写入后 , 开启 `Jenkins` 匿名访问登录 `Jenkins` 首页，选择 `系统管理→全局安全配置` ，授权策略选择 `任何用户可以做任何事（没有任何限制）` .  完整的流水线脚本如下：
```sh
pipeline{
    agent none
    stages{
        stage('mvn-build'){
            agent {
                docker {
                    image '192.168.1.100/library/maven'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps{
                sh 'cp -rfv /opt/repository /root/.m2/ && ls -l /root/.m2/repository'
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
            }
        }
        stage('image-build'){
            agent any
            steps{
                sh 'cd gateway && docker build -t 192.168.1.100/springcloud/gateway -f Dockerfile .'
                sh 'cd config && docker build -t 192.168.1.100/springcloud/config -f Dockerfile .'
                sh 'docker login 192.168.1.100 -u=admin -p=Harbor12345'
                sh 'docker push 192.168.1.100/springcloud/gateway'
                sh 'docker push 192.168.1.100/springcloud/config'
            }
        }
        stage('cloud-deploy'){
            agent any
            steps{
                sh 'sed -i "s/sqshq\\/piggymetrics-gateway/192.168.1.100\\/springcloud\\/gateway/g" yaml/deployment/gateway-deployment.yaml'
                sh 'sed -i "s/sqshq\\/piggymetrics-config/192.168.1.100\\/springcloud\\/config/g" yaml/deployment/config-deployment.yaml'
                sh 'kubectl create ns springcloud'
                sh 'kubectl apply -f yaml/deployment/gateway-deployment.yaml'
                sh 'kubectl apply -f yaml/deployment/config-deployment.yaml'
                sh 'kubectl apply -f yaml/svc/gateway-svc.yaml'
                sh 'kubectl apply -f yaml/svc/config-svc.yaml'
            }
        }
    }
}
```
* #### 构建 CI/CD
> 1. 在 `GitLab` 的项目中，通常会使用 `Webhook` 的各种事件来触发对应的构建，通常配置好后会向设定好的 `URL` 发送 `post` 请求。登录 `GitLab` ，进入 `springcloud` 项目，现在左侧导航栏` Settings→Webhooks ` ，将前面记录的 `GitLab webhook URL` 地址填入 `URL` 处，禁用 `SSL` 认证，单击 ` Add webhook ` 按钮添加 `webhook` ， 单击 ` Test→Push events ` 按钮进行测试 , 结果返回 ` HTTP 200 `则表明 `Webhook` 配置成功.
> 2. 登录 `Jenkins` ，可以看到 `springcloud` 项目已经开始构建，选择左侧导航栏 `打开Blue Ocean` , Blue Ocean是pipeline的可视化UI，同时兼容经典的自由模式的job。Jenkins Pipeline从头开始设计，但仍与自由式作业兼容，Blue Ocean减少了经典模式下的混乱并为团队中的每个成员增加了清晰度。   单击项目名称 `springcloud` , 单击正在构建的 `pipeline` 可以查看阶段视图 , 单击任意  ` > ` 符号可查看每个 `Step` 的构建详情 , 若构建成功，`Blue Ocean`界面会变为`绿色` , 退出阶段试图界面 , 返回Jenkins首页 .
> 3. 进入 `Harbor` 仓库 `springcloud` 项目查看镜像列表，可以看到已自动上传了一个 `gateway`镜像 , 通过端口 `30010` 访问服务.




