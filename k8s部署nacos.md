

###                                                                                                                         k8s部署nacos

#### 一  环境搭建

已经部署好了k8s

```
kubectl get pod -A
```

![image-20210426110302442](http://image.z5689.com/blog/image-20210426110302442.png)

添加helm私库

```
helm repo add  ks-apps  http://192.168.0.3:8080
```

![image-20210426110542171](http://image.z5689.com/blog/image-20210426110542171.png)

查看私库

```
helm search repo ks-apps
```

![image-20210426110739057](http://image.z5689.com/blog/image-20210426110739057.png)



#### 二 快速部署

helm快速部署nacos

```
helm install  nacos   ks-apps/nacos  --set global.mode=quickstart  
```

![image-20210426111208997](http://image.z5689.com/blog/image-20210426111208997.png)

查看状态

```
kubectl get pod 
```

![image-20210426111323935](http://image.z5689.com/blog/image-20210426111323935.png)

![image-20210426112626836](http://image.z5689.com/blog/image-20210426112626836.png)

查看服务端口

```
kubectl get svc
```

![image-20210426111903031](http://image.z5689.com/blog/image-20210426111903031.png)



#### 三  服务访问

访问nacos服务（nodeip 30000端口）

![image-20210426112107751](http://image.z5689.com/blog/image-20210426112107751.png)

![image-20210426112133227](http://image.z5689.com/blog/image-20210426112133227.png)



更多部署参数请参考已下连接  https://github.com/nacos-group/nacos-k8s/tree/master/helm

