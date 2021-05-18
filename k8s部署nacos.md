###                                                                                                                         kubesphere快速部署nacos

### 一  环境搭建

#### 1. 登录kubesphere

以admin用户登录

![image-20210510104649270](https://image.z5689.com/blog/20210510165907.png)

进入工作空间

![image-20210510104751283](https://image.z5689.com/blog/20210510165931.png)

创建ks-test工作空间

![image-20210510104836141](https://image.z5689.com/blog/image-20210510104836141-1620636921844.png)

![image-20210510104906095](https://image.z5689.com/blog/image-20210510104906095.png)

进入ks-test工作空间

![image-20210510105021155](https://image.z5689.com/blog/image-20210510105021155.png)

#### 2. 添加helm库

进入App Repositories

![image-20210510105130206](https://image.z5689.com/blog/image-20210510105130206.png)

![image-20210510105248874](https://image.z5689.com/blog/image-20210510105248874.png)

添加ks-test  helm库

![image-20210510105414578](https://image.z5689.com/blog/image-20210510105414578.png)

添加成功

![image-20210510105447200](https://image.z5689.com/blog/image-20210510105447200.png)



### 二 快速部署

#### 1. 创建test项目

创建test项目

![image-20210510105716840](https://image.z5689.com/blog/image-20210510105716840.png)

![image-20210510105742236](https://image.z5689.com/blog/image-20210510105742236.png)

进入test项目

![image-20210510105815603](https://image.z5689.com/blog/image-20210510105815603.png)

#### 2. 部署nacos

进入Apps

![image-20210510140811030](https://image.z5689.com/blog/image-20210510140811030.png)

![image-20210510140901711](https://image.z5689.com/blog/image-20210510140901711.png)

选择添加的ks-test  helm库

![image-20210510140952166](https://image.z5689.com/blog/image-20210510140952166.png)

找到nacos

![image-20210510141027836](https://image.z5689.com/blog/image-20210510141027836.png)

![image-20210510141104753](https://image.z5689.com/blog/image-20210510141104753.png)

写入应用名称

![image-20210510141202109](https://image.z5689.com/blog/image-20210510141202109.png)

默认快速启动模式

![image-20210510141319260](https://image.z5689.com/blog/image-20210510141319260.png)

部署成功

![image-20210510142215613](https://image.z5689.com/blog/image-20210510142215613.png)



#### 3. 查看状态

进入nacos应用

![image-20210510142356559](https://image.z5689.com/blog/image-20210510142356559.png)



服务映射在nodeip:30000端口

![image-20210510142426511](https://image.z5689.com/blog/image-20210510142426511.png)



#### 4. 访问nacos服务（nodeip 30000端口）

![image-20210426112107751](https://image.z5689.com/blog/image-20210426112107751.png)

![image-20210426112133227](https://image.z5689.com/blog/image-20210426112133227.png)



更多部署参数请参考已下连接  https://github.com/nacos-group/nacos-k8s/tree/master/helm



### 三  部署服务注册nacos

#### 1. 环境准备  

​	下载示例包

```
git clone  https://gitee.com/misteruly/gateway.git
git clone  https://gitee.com/misteruly/service_one.git
git clone  https://gitee.com/misteruly/service_two.git
```

![image-20210518093110921](https://image.z5689.com/blog/image-20210518093110921.png)

修改gateway，service_one，service_two配置文件application.yml

![image-20210512141346567](https://image.z5689.com/blog/image-20210512141346567.png)

修改成你自己的nacos地址并上传到自己的git，这里是k8s跨名称空间连接服务，格式：服务名.名称空间.svc.cluster.local:端口号



#### 2. 服务打包镜像并上传镜像

​	使用kubesphere的S2I功能，需要在kubesphere启用DevOps组件和Logging组件，请参考文档：https://kubesphere.com.cn/docs/quick-start/enable-pluggable-components/

​	新建一个项目build-images，然后进入这个项目

![image-20210518094032815](https://image.z5689.com/blog/image-20210518094032815.png)

![image-20210518094101848](https://image.z5689.com/blog/image-20210518094101848.png)

创建一个密钥

![image-20210518094500572](https://image.z5689.com/blog/image-20210518094500572.png)

取名dockerhub-id

![image-20210518094535777](https://image.z5689.com/blog/image-20210518094535777.png)

类型选择镜像库密钥

![image-20210518094652979](https://image.z5689.com/blog/image-20210518094652979.png)

填写你自己的dockerhub的账号和密码

![image-20210518094840227](https://image.z5689.com/blog/image-20210518094840227.png)

创建成功

![image-20210518095005435](https://image.z5689.com/blog/image-20210518095005435.png)



创建一个构建镜像，源代码选择java，然后下一步

![image-20210518095109970](https://image.z5689.com/blog/image-20210518095109970.png)

![image-20210518094249346](https://image.z5689.com/blog/image-20210518094249346.png)

填入自己的代码链接和分支，镜像名称和版本，以及刚才创建自己的dockerhub-id

![image-20210518095338388](https://image.z5689.com/blog/image-20210518095338388.png)

正在构建，需要等待一段时间

![image-20210518095619198](https://image.z5689.com/blog/image-20210518095619198.png)

进入查看构建日志，已构建成功

![image-20210518095753432](https://image.z5689.com/blog/image-20210518095753432.png)



​	已经上传镜像到自己的dockerhub库

![image-20210512140903848](https://image.z5689.com/blog/image-20210512140903848.png)

以相同的方式把service_one和service_two服务构建镜像，并上传镜像。

![image-20210512143155784](https://image.z5689.com/blog/image-20210512143155784.png)



#### 3. 部署服务

部署gateway服务

新建一个项目deploy-java

![image-20210512160823331](https://image.z5689.com/blog/image-20210512160823331.png)

进入deploy-java

![image-20210512161024730](https://image.z5689.com/blog/image-20210512161024730.png)

创建一个服务

![image-20210512161115281](https://image.z5689.com/blog/image-20210512161115281.png)

选择无状态的服务

![image-20210512161229273](https://image.z5689.com/blog/image-20210512161229273.png)

取名gateway，然后下一步

![image-20210512161305106](https://image.z5689.com/blog/image-20210512161305106.png)



填写自己gateway的镜像

![image-20210512161545281](https://image.z5689.com/blog/image-20210512161545281.png)

填写服务名，容器端口号，和服务端口，点击√，然后next

![image-20210512161912559](https://image.z5689.com/blog/image-20210512161912559.png)

然后next

![image-20210512162423999](https://image.z5689.com/blog/image-20210512162423999.png)

internet Access选择NodePort，然后create

![image-20210512162509727](https://image.z5689.com/blog/image-20210512162509727.png)

gateway服务访问的端口是：nodeip:31698

![image-20210512162653991](https://image.z5689.com/blog/image-20210512162653991.png)

负载已经运行

![image-20210512163055210](https://image.z5689.com/blog/image-20210512163055210.png)

我们以相同的方式部署service_one，service_two服务

![image-20210512163511211](https://image.z5689.com/blog/image-20210512163511211.png)

​	都已经注册成功到nacos

![image-20210512165036640](https://image.z5689.com/blog/image-20210512165036640.png)




#### 4. 服务访问

​	访问service_one服务

![image-20210512163714382](https://image.z5689.com/blog/image-20210512163714382.png)



​	访问service_two服务

![image-20210512163746905](https://image.z5689.com/blog/image-20210512163746905.png)

参考文档：https://blog.csdn.net/weixin_43517302/article/details/109701269?spm=1001.2014.3001.5501	

#### 小结：Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。本实例使用Nacos 实现路由转发功能和服务注册功能，适用于SpringCloud服务框架。



