# OverView

Kubernetes 提供了内建的服务发现机制，允许应用程序组件通过服务名称相互通信，而不需要了解其 IP 地址。此外，Kubernetes 还支持负载均衡，可以在多个副本之间分发流量。

在本次练习中我们使用以下工具模拟K8S的一些常用操作:

- `Minikube`启动一个轻量级的Kubernetes集群.

- `kubectl`作为K8S的命令行工具, 与集群交互.

# 环境安装

## minikube

[**Minikube**](https://minikube.sigs.k8s.io/docs/)是一个轻量级的开源工具，使开发人员能够在自己的机器上本地运行和测试 Kubernetes 集群。

1. 运行以下命令安装 (参考链接: https://minikube.sigs.k8s.io/docs/start/)

   ```bash
   # 请以官网的最新步骤为主
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   
   sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
   ```

2. 运行`minikube`

   ```bash
   minikube start
   ```

   ```bash
   😄  minikube v1.32.0 on Ubuntu 22.04 (amd64)
   ✨  Automatically selected the docker driver. Other choices: none, ssh
   📌  Using Docker driver with root privileges
   ❗  For an improved experience it's recommended to use Docker Engine instead of Docker Desktop.
   Docker Engine installation instructions: https://docs.docker.com/engine/install/#server
   👍  Starting control plane node minikube in cluster minikube
   🚜  Pulling base image ...
   💾  Downloading Kubernetes v1.28.3 preload ...
       > preloaded-images-k8s-v18-v1...:  403.35 MiB / 403.35 MiB  100.00% 13.39 M
       > gcr.io/k8s-minikube/kicbase...:  453.90 MiB / 453.90 MiB  100.00% 10.61 M
   🔥  Creating docker container (CPUs=2, Memory=3400MB) ...
   🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
       ▪ Generating certificates and keys ...
       ▪ Booting up control plane ...
       ▪ Configuring RBAC rules ...
   🔗  Configuring bridge CNI (Container Networking Interface) ...
   🔎  Verifying Kubernetes components...
       ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
   🌟  Enabled addons: storage-provisioner, default-storageclass
   🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
   ```
   
   

## kubectl

kubectl 是 Kubernetes 的命令行工具，用于与 Kubernetes 集群进行交互和管理。通过 kubectl，用户可以执行各种操作，包括创建、修改、删除和查看 Kubernetes 资源，以及管理集群的配置和状态。

1. 使用`snap` 安装kubectl

   > 使用`snap`是因为apt包管理器中没有kubectl, 笔者WSL2安装完成后默认就有`snap`,如果读者环境中没有`snap`, 可自行安装,或提`Issue`

   ```bash
   # --classic 使 Snap安装的应用有权限可以访问系统上的所有文件, 否则只能在应用自己的沙箱中运行
   sudo snap install kubectl --classic
   ```

> kubectl 1.29.3 from Canonical✓ installed

2. cli 验证访问

   ```bash
   kubectl cluster-info
   ```

> Kubernetes control plane is running at https://127.0.0.1:13026
> CoreDNS is running at https://127.0.0.1:13026/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
>
> To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

3. 查看节点信息

   ```bash
   kubectl get nodes
   ```

> NAME       STATUS   ROLES           AGE   VERSION
> minikube   Ready    control-plane   24m   v1.28.3
# Quick Start

1. 首先确保`minikube` 服务已启动, 如果没有请运行: `minikube start --container-runtime=docker --vm=true` [参考链接](https://minikube.sigs.k8s.io/docs/tutorials/docker_desktop_replacement/)

2. [可选步骤] 运行`kubectl config use-context minikube`, 命令设定`K8S`的上下文

   > 如有需要, 可使用`kubectl config use-context -` 切换至上一个上下文

3. 运行`eval $(minikube -p minikube docker-env)`, 将`docker`指向Minikube 中的Docker守护进程. [参考链接](https://minikube.sigs.k8s.io/docs/tutorials/docker_desktop_replacement/)

   > 这么做的原因是后面`Deploy`yaml 文件的时候, 我们自己`build`的镜像是不在Minikube 的docker引擎中的.
   >
   > 在运行此命令前, 我们使用`docker image ls`会看到以前的镜像, 运行此命令后再次运行看到的将是`Minikube `引擎中的镜像.
   >
   > 关掉此shell, 这条命令即失效

4. 重新build docker 镜像.

5. 如果`Deployment`的时候报错了: 参阅此[解决方案](https://stackoverflow.com/questions/65896681/exec-docker-credential-desktop-exe-executable-file-not-found-in-path)

## Deployment

1. 在`./backend-demo`中创建`fast-api-server-deployment.yaml`文件:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: fast-api-server-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: fast-api-server
     template:
       metadata:
         labels:
           app: fast-api-server
       spec:
         containers:
         - name: fast-api-server
           image: fastapi-docker-image  # docker-demo 练习中我们创建的image
           imagePullPolicy: Never
   
           ports:
           - containerPort: 8000
   ```

   

2. apply运行此文件

   ```bash
   kubectl apply -f backend-demo/fast-api-server-deployment.yaml
   ```

3. 查看deployment 状况

   ```bash
   kubectl get deployments
   ```

   > NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
   > fast-api-server-deployment   3/3     3            3           40m

4. 查看详细部署情况

   ```bash
   kubectl describe deployment <deployment-name>
   ```

   输出中将包含有关 Deployment 的详细信息，如下所示：

   - Deployment 的名称、命名空间和标签。
   - 副本控制器（ReplicaSet）的名称。
   - 部署的 Pod 模板。
   - 副本数量以及正在运行、准备好和可用的 Pod 数量。
   - 部署的策略，如滚动更新策略。
   - 与部署相关的事件和状态。

## Service

1. 在`./backend-demo`中创建`fast-api-server-service.yaml`文件:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: fast-api-server-service
   spec:
     selector:
       app: fast-api-server
     ports:
       - protocol: TCP
         port: 8000
         targetPort: 8000
     type: LoadBalancer
   ```

2. apply运行此文件

   ```bash
   kubectl apply -f backend-demo/fast-api-server-service.yaml
   ```

3. 查看所有的service

   ```bash
   minikube service list
   ```

4. 查看service的状况

   ```bash
   kubectl get services fast-api-server-service
   ```

   > NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
   > kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d

5. 如果`PORT(S)`是`<pending>`状态, 则运行下面的命令:

   ```bash
   minikube service fast-api-server-service
   ```

   > 会输出两个URL:
   >
   > 1. 集群中其他 Pod 或服务之间进行通信时使用的地址
   > 2. 隧道（tunnel）建立后暴露在本地的地址(方便开发和调试)

## 性能测试

```bash
Requests per second:    3750.02 [#/sec] (mean)
Time per request:       26.666 [ms] (mean)
Time per request:       0.267 [ms] (mean, across all concurrent requests)
Transfer rate:          549.32 [Kbytes/sec] received

Connection Times (ms)
        min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       2
Processing:     4   25  25.6     15      82
Waiting:        2   25  25.5     14      82
Total:          4   25  25.5     15      82

Percentage of the requests served within a certain time (ms)
50%     15
66%     17
75%     19
80%     67
90%     76
95%     79
98%     80
99%     81
100%     82 (longest request)
```

## 停止服务

停止`Deployment`就停止了部署的`Pod`, 停止`Service`就停止了流量路由

1. 停止`Deployment`

   - `kubectl scale deployment <deployment-name> --replicas=0`

     保留配置和历史的条件下, 将副本数量缩减为 0, 便于重启

   - `kubectl delete deployment <deployment-name>`

     停止正在运行的 Pod , 并删除Deployment 的所有相关信息, 不可逆

2. 停止`Service`

   ```bash
   kubectl delete service <service-name>
   ```

   删除负载均衡器，并停止将流量路由到对应的`Deployment`
   

# Advanced

## service互联

因为我们的`fast-api-server`代码中的`redis`地址是支持从环境变量中获取并连接的, 所以我们可以很灵活的通过在`Deployment`的时候注入`redis`服务:

1. 创建`redis`集群

   - `redis-deployment.yaml`

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: redis-deployment
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: redis
       template:
         metadata:
           labels:
             app: redis
         spec:
           containers:
           - name: redis
             image: redis:latest
             ports:
             - containerPort: 6379  # 端口号后面要用
     ```

   - `redis-service.yaml`

     ```bash
     apiVersion: v1
     kind: Service
     metadata:
       name: redis-service  # 服务名后面要用
     spec:
       selector:
         app: redis
       ports:
         - protocol: TCP
           port: 6379
           targetPort: 6379
     ```

     

2. 添加环境变量值

   在`fast-api-server-deployment.yaml`中添加`redis`服务的信息:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: fast-api-server-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: fast-api-server
     template:
       metadata:
         labels:
           app: fast-api-server
       spec:
         containers:
         - name: fast-api-server
           image: fastapi-docker-image
           imagePullPolicy: Never
   
           ports:
           - containerPort: 8000
           env:   # 从这里开始为我们新增的
           - name: REDIS_URL  # 与代码中的变量保持一致
             value: redis://redis-service:6379 # 上面定义的
   ```

3. 启动服务

   ```bash
   # 先启动fast-api-server服务
   kubectl apply -f backend-demo/fast-api-server-deployment.yaml
   kubectl apply -f backend-demo/fast-api-server-service.yaml
   
   kubectl apply -f backend-demo/redis-deployment.yaml
   kubectl apply -f backend-demo/redis-service.yaml
   ```

## Nginx反向代理

1. yaml文件定义

   - `deployment.yaml`

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: nginx-deployment
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: nginx
       template:
         metadata:
           labels:
             app: nginx
         spec:
           containers:
           - name: nginx
             image: nginx:latest
             ports:
             - containerPort: 80
             volumeMounts:
             - name: config-volume
               mountPath: /etc/nginx/nginx.conf
               subPath: nginx.conf
           volumes:
           - name: config-volume
             configMap:
               name: nginx-config
               items:  # 缺少这个项让我踩了一会儿坑
                   - key: nginx.conf
                     path: nginx.conf
     
     ```

   - `nginx-service.yaml`

     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: nginx-service
     spec:
       selector:
         app: nginx
       ports:
         - protocol: TCP
           port: 80
           targetPort: 80
     ```

2. conf文件

   在`./backend-demo`中新建`nginx.conf`

   ```bash
   events {
       worker_connections 1024;
   }
   
   http {
       server {
           listen 80;
   
           location / {
               proxy_pass http://fast-api-server-service:8000;
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;
           }
       }
   }
   ```

3. 创建ConfigMap

   ```bash
   kubectl create configmap nginx-config --from-file=nginx.conf=./backend-demo/nginx.conf
   
   # 查看刚创建的configmap
   kubectl describe configmap nginx-config
   ```

   - `nginx-config` 为 ConfigMap 对象的名称
   - `nginx.conf` 为`ConfigMap` 对象的Key
   - `./backend-demo/nginx.conf` 对应的文件是Key的值

4. 部署并运行Nginx

   ```bash
   kubectl apply -f backend-demo/nginx-deployment.yaml
   kubectl apply -f backend-demo/nginx-service.yaml
   
   minikube service nginx-service
   ```

5. 性能测试

   运行`ab -n 1000 -c 100 {your-path}`

   ```bash
   Requests per second:    2282.18 [#/sec] (mean)
   Time per request:       43.818 [ms] (mean)
   Time per request:       0.438 [ms] (mean, across all concurrent requests)
   Transfer rate:          387.79 [Kbytes/sec] received
   
   Connection Times (ms)
                 min  mean[+/-sd] median   max
   Connect:        0    0   0.2      0       1
   Processing:     4   42  36.1     15      95
   Waiting:        4   42  36.1     15      95
   Total:          4   42  36.0     15      95
   
   Percentage of the requests served within a certain time (ms)
     50%     15
     66%     83
     75%     85
     80%     86
     90%     88
     95%     91
     98%     93
     99%     94
    100%     95 (longest request)
   ```

   感觉我们上了这么多组件性能提升很小啊...

   实测直接call fastapi后端的`RPS`要比 `nginx`的高, 应该是我们业务内容太简单导致的.

## ConfigMap





   

   

   

   
