### Kubernetes学习笔记

[Kubernetes学习笔记-已更新到第25篇，持续更新中······](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzUzNTY5MzU2MA==&action=getalbum&album_id=1394839706508148737#wechat_redirect)

### 安装MySQL

- 创建ConfigMap 把my.cnf里的配置放在ConfigMap对象里，后面作为配置文件会挂载进Pod
```
kubectl apply -f mysql-singleton/mysql-configmap.yaml
```
- 创建MySQL应用的Pod和向外提供服务的Service
```
kubectl apply -f mysql-singleton/deployment-service.yaml
```

- 从电脑链接MySQL
```
# 假设电脑上Kubernetes用的是Minikube， 这里只是例子， 也可以用K3d，用什么安装工具在电脑上安装Kubernetes不重要
# 获得集群对外提供访问的ip, 在每个人的电脑上ip不一样。
# MySQL的Service对外暴露的端口为30306
IP=$(minikube ip)  

mysql -uroot -puser -h ${IP} -P 30306
```

### 安装Redis

- 创建ConfigMap
```
kubectl apply -f mysql-singleton/mysql-config.yaml
```

- 创建调度Redis应用的Deployment和向外提供服务的Service
```
kubectl apply -f redis-singleton/deployment-service.yaml
```

- 从电脑上访问Redis

```
# Redis Service对外暴露端口为：31379
IP=$(minikube ip)  

redis-cli -h ${IP} -p 31379

```

### 安装ETCD

- 创建命名空间etcd, etcd的资源都放在这个空间下:
```
kubectl create namespace etcd
```

- 创建用到的Services:
```
kubectl apply -f resources/services.yml -n etcd
```

- 创建etcd 集群:
```
cat resources/etcd.yml.tmpl | resources/config.bash | kubectl apply -n etcd -f -
```
备注： 上面的命令只在minikue的k8s上有效，如果用的是Docker Desktop自带的K8s集群，请执行下面的命令
```
kubectl apply -f resources/etcd-statefulset-newl.yaml
```


- 从电脑上访问etcd
```
IP=$(minikube ip)

PORT=$(kubectl get services -o jsonpath="{.spec.ports[].nodePort}" etcd-client -n etcd)
ETCDCTL_API=3 ./etcdctl --endpoints ${IP}:${PORT}  get root --prefix
```

### 安装MongoDB

安装的是Standalone（单点）类型的MongoDB

- 创建root用户的用户名和密码的secret对象
```
kubectl apply -f mongodb-singleton/mongo-secret.yaml
```

- 创建Deployment和Service
```
kubectl apply -f mongodb-singleton/deployment-service.yaml
```

- 客户端访问，推荐使用MongoDB Compass
```
IP=$(minikube ip) # 假设$IP是 192.168.64.4

mongodb://username:password@192.168.64.4:30017/?authSource=admin&readPreference=primary&appname=MongoDB%20Compass&ssl=false
```
- 在客户端里为数据库创建用户
```
> use my-database

> db.createUser(
  {
    user: "my-user",
    pwd: "passw0rd",
    roles: [ { role: "readWrite", db: "my-database" } ]
  }
)
```
### 安装Nacos
- 创建Nacos的Deployment和Service

```shell
kubectl apply -f nacos/deployment.yaml

kubectl apply -f nacos/service.yaml
```

- 本地访问Nacos
```
# 登录的用户名和密码都是nacos

http://localhost:30848/nacos
```

- 用域名访问

也可以添加Ingress，用域名访问Nacos

创建Ingress
```shelll
kubectl apply -f ingress.yaml
```
hosts中设置 dev.nacos.com 后可通过域名进行访问
```
http://dev.nacos.com/nacos
```
如果是在Spring中使用nacos时需要这么配置
```
spring.cloud.nacos.discovery.server-addr=127.0.0.1:30848
// 或者是下面这样，
//不加端口，Spring Nacos的Jar包默认会去访问8848端口，
//所以在程序里用ingress配置的域名访问时也要加端口
spring.cloud.nacos.discovery.server-addr=dev.nacos.com:80
```

### 安装RocketMQ
- 安装MQ
```shell
kubectl apply -f rocketmq/rocketmq-deployment-service.yaml
```
- 安装Web Console
```shell script
kubectl apply -f rocketmq/rocketmq-console.yaml
```
- 访问Web Console
地址：http://localhost:30875

