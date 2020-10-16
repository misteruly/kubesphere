**一  环境要求**

1.kubesphere v3.0.0 （并部署OpenPitrix组件）

2.使用project-regular，更多用户，角色设置请参考    [https://kubesphere.com.cn/en/docs/quick-  start/create-workspace-and-project/](https://kubesphere.com.cn/en/docs/quick-start/create-workspace-and-project/)

**二  部署 porter** 

登录project-regular用户

点击进入 app store

![img](https://i.loli.net/2020/10/16/s4ezWQGiZ6fyEUb.png)

![img](https://i.loli.net/2020/10/16/hSTG7tNFduIKJCb.png)

![img](https://i.loli.net/2020/10/16/CAx3FsZDp9Rwkr5.png)

![img](https://i.loli.net/2020/10/16/Iovrz8qbaQnCuVZ.png)

Deployment Location  所有选着完成打 √  确认，点击Next

![img](https://i.loli.net/2020/10/16/tpJQESvlZFNL3PA.png)

配置默认点击 Deploy

![img](https://i.loli.net/2020/10/16/rqgaUFfDTd4vQuZ.png)

现在状态创建中（creating），等待成运行状态（active）

![img](https://i.loli.net/2020/10/16/r9DmUBxOJSuGieC.png)

进入应用，还有一个负载正在更新，继续等待。

![img](https://i.loli.net/2020/10/16/hkJtCF9O3GY42LB.png)

部署完成

![img](https://i.loli.net/2020/10/16/2dRf3qXMsm4O7LB.png)

**三  配置layer2模式**

登录kubesphere  master

```
$ cat << EOF > layer2.yaml
apiVersion: network.kubesphere.io/v1alpha1
kind: Eip
metadata:
    name: eip-sample-pool
spec:
    # 修改ip地址段为实际环境的ip地址段。可以为单个地址或者是地址段
    address: 172.20.0.2
    protocol: layer2
    disable: false
EOF
$ kubectl apply -f layer2.yaml
eip.network.kubesphere.io/eip-sample-pool created
```

部署nginx

在Kubernetes集群上:

```
$ cat << EOF > nginx-layer2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
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
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    lb.kubesphere.io/v1alpha1: porter
    protocol.porter.kubesphere.io/v1alpha1: layer2
  name: nginx-service
spec:
  selector:
    app: nginx
  type:  LoadBalancer 
  ports:
    - name: http
      port: 8088
      targetPort: 80
EOF
$ kubectl apply -f nginx-layer2.yaml
deployment.apps/nginx-deployment created
service/nginx-service created
$ kubectl get svc/nginx-service
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
nginx-service   LoadBalancer   10.233.15.225   172.20.0.2    8088:32539/TCP   88m
```

**四  防火墙  ACL  放通8088端口**

ACL加入放行端口

![img](https://i.loli.net/2020/10/16/C2Vvy3g4FNobDZA.png)

防火墙加入放行端口

![img](https://i.loli.net/2020/10/16/FSZRujDlyYoPEhQ.png)

访问成功

![img](https://raw.githubusercontent.com/misteruly/images/master/20201016171457.png)

添加dns主机

![img](https://raw.githubusercontent.com/misteruly/images/master/20201016171458.png)

使用自己域名访问

![img](https://raw.githubusercontent.com/misteruly/images/master/20201016171427.png)

