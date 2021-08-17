## 搭建zookeeper集群
```
# 解压
tar -zxvf apache-zookeeper-3.7.0-bin.tar.gz

# 对解压文件重命名，方便管理
mv apache-zookeeper-3.7.0-bin zookeeper

# 进入zookeeper配置文件夹
cd zookeeper/conf/

# 复制默认配置文件进行自定义
cp zoo_sample.cfg zoo.cfg

# 修改配置文件中属性
dataDir=/tmp/zookeeper/1    # 添加一个编号
clientPort=2181             # 每个zookeeper的端口号都不相同
admin.serverPort=10808      # 添加admin.serverPort防止端口占用,每个节点都不相同

# 在dataDir对应目录下分别创建myid文件， 根据上一步指定的dataDir，找到对应的文件地址，如果没有则进行创建，进入目录后输入
echo 1 > myid
# 数字按顺序递增，这个非常重要，用于后面配置集群

# 配置集群信息，每个节点对应一条数据，需要添加到每一个节点中去
# server.服务器id=服务器IP地址:服务器直接通信端口:服务器之间选举投票端口
server.1=127.0.0.1:2881:3881
server.2=127.0.0.1:2882:3882
server.3=127.0.0.1:2883:3883
...

# 启动zookeeper节点，每个节点都要启动
cd bin/
./zkServer.sh start

# 查看节点状态（一个leader，多个follower）
cd bin/
./zkServer.sh status
```