**kubesphere****备份恢复（**Multinode**模式）**

## 目的

备份kubesphere集群，在另外多台主机上恢复kubesphere集群

 

## 前提

![image-20210205150424480](http://image.z5689.com/blog/image-20210205150424480.png)

l 集群A(master,node1.node2)：已经部署KubeSphere-v3.0.0

l 集群B(master,node1,node2)：已经部署kubernetes-v1.17.9

l 存储端minio主机

l Centos7环境

 

## 预估时间

10-20 分钟

 

## 操作指南

**1.**   **配置存储端****minio****主机**

安装docker

\# yum install docker –y

\# systemctl enable docker && service docker start

 

安装minio

\# mkdir –p /private/mnt/data

\# docker run -p 9000:9000 \

 --name minio1 \

 -v /private/mnt/data:/data \

 -e "MINIO_ACCESS_KEY=minio" \

 -e "MINIO_SECRET_KEY=minio123" \

 minio/minio server /data

 

安装完成

![img](http://image.z5689.com/blog/clip_image002.jpg)

 

使用access key和secret key登录

![img](http://image.z5689.com/blog/clip_image004.jpg)

设置bucket

![img](http://image.z5689.com/blog/clip_image006.jpg)

写入backup，enter键提交，velero会用到这个bucket

![img](http://image.z5689.com/blog/clip_image008.jpg)

![img](http://image.z5689.com/blog/clip_image010.jpg)

 

**2.** **集群****A****，****B****部署****velero**

集群A部署velero

\#wget https://github.com/vmware-tanzu/velero/releases/download/v1.5.2/velero-v1.5.2-linux-amd64.tar.gz

\# tar zxvf velero-v1.5.2-linux-amd64.tar.gz && cd velero-v1.5.2-linux-amd64

\# cp -r velero /usr/local/bin/

 

创建minio凭证

\# cat > credentials-velero << EOF

[default]

aws_access_key_id = minio

aws_secret_access_key = minio123

EOF 

 

安装velero

\# velero install \

  --provider aws \

  --bucket backup \

  --use-restic \

  --secret-file ./credentials-velero \

  --use-volume-snapshots=false \

  --plugins velero/velero-plugin-for-aws:v1.1.0 \

​    --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://192.168.0.3:9000

 

  --provider aws 因为兼容minio，所以需要用到供应商aws选项

  --bucket backup 在minio新建的bucket backup

  --use-restic 表示使用restic备份pv数据

  --secret-file minio凭证，与我们创建minio存储时的凭证保持一致

  --use-volume-snapshots=false 禁止使用快照功能

  --plugins velero/velero-plugin-for-aws:v1.1.0 使用aws的velero插件

  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://192.168.0.3:9000

  备份的配置信息，region备份区域在minio，s3ForcePathStyle s3强制路径样式为true

  s3Url s3的地址为http://192.168.0.3:9000

 

安装完成

![img](http://image.z5689.com/blog/clip_image012.jpg)

 

集群B部署velero也和上述集群A部署velero一样

注意：集群B需要提前安装好kubesphere所需依赖组件

\# yum install curl openssl ebtables socat ipset conntrack yum-utils -y

**3.** **部署验证服务（****wordpress****）**

为了验证数据的有效性部署1个test工作空间和对应的名称空间

登录kubesphere

![img](http://image.z5689.com/blog/clip_image014.jpg)

 

新建工作空间test

![img](http://image.z5689.com/blog/clip_image016.jpg)

![img](http://image.z5689.com/blog/clip_image018.jpg)

![img](http://image.z5689.com/blog/clip_image020.jpg)

![img](http://image.z5689.com/blog/clip_image022.jpg)

 

创建项目（名称空间）

![img](http://image.z5689.com/blog/clip_image024.jpg)

![img](http://image.z5689.com/blog/clip_image026.jpg)

 

创建完成

![img](http://image.z5689.com/blog/clip_image028.jpg)

 

在test项目里面部署wordpress

\# helm repo add bitnami https://charts.bitnami.com/bitnami

\# helm install wordpress bitnami/wordpress   -n test   --set service.type=NodePort   --set wordpressUsername=admin   --set wordpressPassword=Pass@Word

 

安装完成

![img](http://image.z5689.com/blog/clip_image030.jpg)

 

对外服务端口是30086

![img](http://image.z5689.com/blog/clip_image032.jpg)

 

我们已nodeip 30086端口登录wordpress

 

账号 admin  密码 Pass@Word

![img](http://image.z5689.com/blog/clip_image034.jpg)

 

添加一遍文档已示标记

![img](http://image.z5689.com/blog/clip_image036.jpg)

 

写好就pulish

![img](http://image.z5689.com/blog/clip_image038.jpg)

 

我们看到多了一片test文档

![img](http://image.z5689.com/blog/clip_image040.jpg)

 

 

**4.** **集群****A****备份****kubesphere****集群**

登录集群A master端

 

查看master端的污点

kubectl describe node master | grep Taint

 

取消master端的污点

kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-

 

备份存储对象文件

\# kubectl get sc local -o yaml > openebs-sc.yaml

\# scp -r root@192.168.0.8:/root/

 

注释pv，只有被注释才能被 restic备份

 

\# kubectl get pv -A

 

![img](http://image.z5689.com/blog/clip_image042.jpg)

 

标记kubesphere-monitoring-system名称空间的PV

\# kubectl describe pod prometheus-k8s-0 -n  kubesphere-monitoring-system

 

只截取Volumes的数据

 

Volumes:

 prometheus-k8s-db:

  Type:    PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)

  ClaimName: prometheus-k8s-db-prometheus-k8s-0

  ReadOnly:  false

 config:

  Type:    Secret (a volume populated by a Secret)

  SecretName: prometheus-k8s

  Optional:  false

 tls-assets:

  Type:    Secret (a volume populated by a Secret)

  SecretName: prometheus-k8s-tls-assets

  Optional:  false

 config-out:

  Type:    EmptyDir (a temporary directory that shares a pod's lifetime)

  Medium:  

  SizeLimit: <unset>

 prometheus-k8s-rulefiles-0:

  Type:   ConfigMap (a volume populated by a ConfigMap)

  Name:   prometheus-k8s-rulefiles-0

  Optional: false

 secret-kube-etcd-client-certs:

  Type:    Secret (a volume populated by a Secret)

  SecretName: kube-etcd-client-certs

  Optional:  false

 prometheus-k8s-token-lz2jc:

  Type:    Secret (a volume populated by a Secret)

  SecretName: prometheus-k8s-token-lz2jc

  Optional:  false

 

不需要标记token，在pod生成之后，会重新生成token

\# kubectl -n kubesphere-monitoring-system annotate pod prometheus-k8s-0 backup.velero.io/backup-volumes=prometheus-k8s-db,config,tls-assets,config-out,prometheus-k8s-rulefiles-0,secret-kube-etcd-client-certs

 

标记test名称空间的PV

\# kubectl describe pod  wordpress-mariadb-0  -n  test

 

只截取Volumes的数据

 

Volumes:

 data:

  Type:    PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)

  ClaimName: data-wordpress-mariadb-0

  ReadOnly:  false

 config:

  Type:   ConfigMap (a volume populated by a ConfigMap)

  Name:   wordpress-mariadb

  Optional: false

 wordpress-mariadb-token-rslv4:

  Type:    Secret (a volume populated by a Secret)

  SecretName: wordpress-mariadb-token-rslv4

  Optional:  false

 

\# kubectl -n test annotate pod wordpress-mariadb-0  backup.velero.io/backup-volumes=data,config

 

 

\# kubectl describe pod  wordpress-69b68bc578-lw6dq  -n  test

 

只截取Volumes的数据

 

Volumes:

 wordpress-data:

  Type:    PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)

  ClaimName: wordpress

  ReadOnly:  false

 default-token-2wh84:

  Type:    Secret (a volume populated by a Secret)

  SecretName: default-token-2wh84

  Optional:  false

 

\# kubectl -n test annotate pod wordpress-69b68bc578-lw6dq  backup.velero.io/backup-volumes=wordpress-data

 

标记kubesphere-system名称空间的PV

 

\# kubectl describe pod openldap-0 -n  kubesphere-system

 

只截取Volumes的数据

Volumes:

 openldap-pvc:

  Type:    PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)

  ClaimName: openldap-pvc-openldap-0

  ReadOnly:  false

 default-token-wdlbc:

  Type:    Secret (a volume populated by a Secret)

  SecretName: default-token-wdlbc

  Optional:  false

 

\# kubectl -n kubesphere-system annotate pod openldap-0  backup.velero.io/backup-volumes=openldap-pvc

 

\# kubectl describe pod redis-6b479dfd69-4twtp -n  kubesphere-system

 

只截取Volumes的数据

 

Volumes:

 redis-pvc:

  Type:    PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)

  ClaimName: redis-pvc

  ReadOnly:  false

 default-token-wdlbc:

  Type:    Secret (a volume populated by a Secret)

  SecretName: default-token-wdlbc

  Optional:  false

 

\# kubectl -n kubesphere-system annotate pod redis-6b479dfd69-4twtp  backup.velero.io/backup-volumes=redis-pvc

 

含有pv数据的pod都标记完成

 

  

备份名称空间（kube-system，kubesphere-controls-system，kubesphere-monitoring-system，kubesphere-system，test）

 

\# velero backup create kube-system-bak --include-namespaces kube-system

\# velero backup create kubesphere-controls-system-bak --include-namespaces kubesphere-controls-system

\# velero backup create kubesphere-monitoring-system-bak --include-namespaces kubesphere-monitoring-system

\# velero backup create kubesphere-system-bak --include-namespaces kubesphere-system

\# velero backup create test-bak --include-namespaces test

 

备份全部完成

![img](http://image.z5689.com/blog/clip_image044.jpg)

 

minio已经生成备份数据

![img](http://image.z5689.com/blog/clip_image046.jpg)

 

 

**5.** **集群****B****还原****kubesphere****集群**

登入集群B  master端

 

\# velero get backup

 

可以看到备份数据

![img](http://image.z5689.com/blog/clip_image048.jpg)

 

查看所有pod信息

\#kubectl get pod -A -o wide![img](http://image.z5689.com/blog/clip_image050.jpg)

还原kube-system名称空间的数据

\# velero restore create --from-backup kube-system-bak

\# kubectl get pod -A

增加了openebs，metrics，snapshot的pod

![img](http://image.z5689.com/blog/clip_image052.jpg)

创建存储对象

\# kubectl apply -f  openebs-sc.yaml

\# kubectl get sc

![img](http://image.z5689.com/blog/clip_image054.jpg)

 

还原kubesphere-system名称空间的数据

取消master污点

\# kubectl taint nodes master-restore node-role.kubernetes.io/master:NoSchedule-

\# velero restore create --from-backup kubesphere-system-bak

\# kubectl get pod -n kubesphere-system

 

![img](http://image.z5689.com/blog/clip_image056.jpg)

 

还原kubesphere-controls-system名称空间的数据

\# velero restore create --from-backup kubesphere-controls-system-bak

\# kubectl get pod -n kubesphere-controls-system

![img](http://image.z5689.com/blog/clip_image058.jpg)

 

还原kubesphere-monitoring-system名称空间的数据

\# velero restore create --from-backup kubesphere-monitoring-system-bak

\# kubectl get pod -n kubesphere-monitoring-system

![img](http://image.z5689.com/blog/clip_image060.jpg)

 

还原test名称空间的数据

\# velero restore create --from-backup test-bak

\# kubectl get pod -n test

![img](http://image.z5689.com/blog/clip_image062.jpg)

 

所有名称空间的pod全部恢复完成

![img](http://image.z5689.com/blog/clip_image064.jpg)

PV全部绑定成功

![img](http://image.z5689.com/blog/clip_image066.jpg)

 

**6.**   **验证服务**

登录集群B kubesphere，验证服务

![img](http://image.z5689.com/blog/clip_image068.jpg)

 

我们看到只有一个工作空间，因为这部分关联数据存储在etcd

![img](http://image.z5689.com/blog/clip_image070.jpg)

单独创建test工作空间

![img](http://image.z5689.com/blog/clip_image072.jpg)

 

![img](http://image.z5689.com/blog/clip_image074.jpg)

 

![img](http://image.z5689.com/blog/clip_image076.jpg)

直接对应上了项目(名称空间)test，名称空间含有工作空间的标签，所以自动关联上了

![img](http://image.z5689.com/blog/clip_image078.jpg)

 

\# kubectl describe ns test

![img](http://image.z5689.com/blog/clip_image080.jpg)

 

 

验证wordpress

\# kubectl get svc -n test

![img](http://image.z5689.com/blog/clip_image082.jpg)

 

登录wordpress

 

![img](http://image.z5689.com/blog/clip_image084.jpg)

 

 

![img](http://image.z5689.com/blog/clip_image086.jpg)

 

![img](http://image.z5689.com/blog/clip_image088.jpg)

 

数据正确

 

 