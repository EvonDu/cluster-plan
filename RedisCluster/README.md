# RedisCluster
### 使用方法
1. 启动docker-compose
    * `docker-compose up`
2. 查看docker容器id并进入
    * `docker ps`
    * `docker exec -it xxxxxxxxx /bin/bash`
3. 执行集群命令
    * `redis-cli --cluster create 172.19.0.11:8011 172.19.0.12:8012 172.19.0.13:8013 172.19.0.14:8014 172.19.0.15:8015 172.19.0.16:8016 --cluster-replicas 1`

### 注意事项
* 注意Redis配置时的三个网络参数
    * cluster-announce-ip：集群各节点IP地址（也就是该节点的公网IP）
    * cluster-announce-port：集群节点映射端口（也就是该节点的公网端口号）
    * cluster-announce-bus-port：集群总线端口（配置就行一般是端口加10000）
* cluster-announce-ip:这个IP需要特别注意一下，如果要对外提供访问功能，需要填写宿主机的IP，如果填写docker分配的IP（172.x.x.x），可能会导致部分集群节点在跳转时失败。
* cluster-announce-bus-port：Redis集群通过总线进行节点间的数据交换，每个Redis节点都会开辟一个额外端口（Redis port加上10000）与总线进行TCP连接，以二进制形式进行数据交换。
* 执行集群命令时，不能用host别名，必须用IP，否则会失败
* 集群完成前，不能设置密码
