# Mysql-Cluster 数据库集群

### 注意事项
* 一定要先创建一个`docker`网络，然后把所有结点放进去，不然会到`db1`、`db2`等无法分配虚拟网络的IP地址，从而无法进行集群
* 用Docker进行集群大概率会失败，一个是Docker中只能启动一个MySQL，并且MysqlCluster只能用局域网建立，不然会出现各种节点连接不上的情况

### 操作步骤
1. 创建一个docker网络，名叫`cluster`
    * `docker network create cluster`
2. 启动`docker-compose`
    * `docker-compose up`
3. 分别进入个子容器
    * `docker ps`
    * `docker exec -it <id> /bin/bash`
4. 为子容器添加环境变量
    * `echo 'export PATH=$PATH:/mc/mysql-cluster/bin' >> /etc/profile`
    * `source /etc/profile`
5. 开启管理节点
    * `ndb_mgmd --config-file=/mc/config.ini --configdir=/ndbm/config --initial`
    * 只在`db1`中开启管理节点
6. 开启数据节点
    * `ndbd --ndb-connectstring=db1 --initial`
7. 开启MySQL相关依赖
    * 安装依赖:`apt-get install libaio-dev`
    * 安装依赖:`apt-get install numactl`
8. 初始化MySQL数据库
    * db1:`mysqld --initialize --basedir=/mc/mysql-cluster --datadir=/mc/ndb1/mdata`
    * db2:`mysqld --initialize --basedir=/mc/mysql-cluster --datadir=/mc/ndb2/mdata`
9. 启动MySQL节点(即MySQL)
    * mysqld --user=root --skip-grant-tables

### 添加远程登录用户
* 登录MySQL
    * mysql -u root -p
* 刷新权限后添加用户，然后再刷新权限
    * flush privileges;
    * grant all on *.* to admin@'%' identified by '123456' with grant option; 
    * flush privileges;

### 使用过的命令
```
[建立网络]
docker network create test
[容器]
docker exec -it 6f028c203d9d /bin/bash
docker exec -it cd40482e2d75 /bin/bash
[建立文件夹]
mkdir /ndbm & mkdir /ndb
mkdir /ndbm/logs & mkdir /ndbm/config & mkdir /ndb/data & mkdir /ndb/mdata
[环境变量]
echo 'export PATH=$PATH:/mc/mysql-cluster/bin' >> /etc/profile
source /etc/profile
[管理节点]
ndb_mgmd --config-file=/mc/config.ini --configdir=/ndbm/config --initial
ndb_mgm
[数据节点]
ndbd --ndb-connectstring=db1 --initial
[SQL节点]
mysqld --initialize --basedir=/mc/mysql-cluster --datadir=/mc/ndb1/mdata
mysqld --initialize --basedir=/mc/mysql-cluster --datadir=/mc/ndb2/mdata
[启动SQL]
mysqld --user=root --skip-grant-tables
```