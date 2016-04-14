====

安装hadoop和yarn步骤

# 主机配置

| 主机名      | IP          | Role                 |
| ----------- |:-----------:| :------------------: |
| admin       | 10.0.2.6    | namenode             |
| mon1        | 10.0.2.3    | resourcemanager      |
| osd1        | 10.0.2.4    | datanode/nodemanager |
| osd2        | 10.0.2.5    | datanode/nodemanager |

# 安装java 设置 JAVA_HOME
```sh
cat /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_66
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
```
# 解压hadoop tarball
```sh
tar -xzvf /home/Hongtao/hadoop-2.7.2.tar.gz -C /opt
sudo chown -R Hongtao:Hongtao /opt/hadoop-2.7.2
```
hadoop的HOME目录就是HADOOP_HOME=/opt/hadoop-2.7.2 以下都是基于这个目录来操作

# 配置文件修改,主要涉及到core-site.xml, hdfs-site.xml, mapred-site.xml和yarn-site.xml

## core-site.xml 设置

core-site.xml 有个默认设置可以参考官方文档[链接](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)  
core-site.xml位于${HADOOP_HOME}/etc/hadoop目录下

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://10.0.2.6:9000</value>
    </property>

    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>

    <!-- 设置namenode和datanode的根目录 -->
    <!-- dfs.namenode.name.dir: file://${hadoop.tmp.dir}/dfs/name  -->
    <!-- dfs.datanode.name.dir: file://${hadoop.tmp.dir}/dfs/data -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop</value>
    </property>
</configuration>
```

## hdfs-site.xml配置
hdfs-site.xml 有个默认设置可以参考官方文档[链接](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)  
hdfs-site.xml位于${HADOOP_HOME}/etc/hadoop目录下

```xml
<?xml version="1.0"?>
<configuration>
    <!-- 设置数据备份 -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.handler.count</name>
        <value>100</value>
    </property>

    <!-- 设置namenode数据存放目录 如果这个设置了 core-site.xml中hadoop.tmp.dir可以不用设置 -->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/data/hadoop/dfs/name</value>
    </property>

    <!-- 设置datanode数据存放目录 如果这个设置了 core-site.xml中hadoop.tmp.dir可以不用设置 -->
    <property>
        <name>dfs.datanode.name.dir</name>
        <value>/data/hadoop/dfs/data</value>
    </property>
</configuration>

```

## mapred-site.xml配置
mapred-site.xml 有个默认设置可以参考官方文档[链接](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)  
mapred-site.xml位于${HADOOP_HOME}/etc/hadoop目录下
```xml
<?xml version="1.0"?>
<configuration>
    <!-- 注意通常情况下这个配置的值都设置为 Yarn，如果没有配置这项，那么提交的 Yarn job 只会运行在local模式，而不是分布式模式 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

## yarn-site.xml配置

yarn-site.xml 有个默认设置可以参考官方文档[链接](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)  
yarn-site.xml位于${HADOOP_HOME}/etc/hadoop目录下

```xml
<?xml version="1.0"?>
<configuration>

    <!-- The address of the applications manager interface in the RM. -->
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>10.0.2.3:8032</value>
    </property>

    <!-- The address of the scheduler interface. -->
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>10.0.2.3:8030</value>
    </property>

    <!-- The http address of the RM web application. -->
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>10.0.2.3:8088</value>
    </property>

    <!-- 新框架中 NodeManager 需要向 RM 报告任务运行状态供 Resouce 跟踪，因此 NodeManager 节点主机需要知道 RM 主机的 tracker 接口地址 -->
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>10.0.2.3:8031</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>


</configuration>
```

# 启动HADOOP

## boot up HDFS
```sh
# The first time you bring up HDFS, it must be formatted
# running on namenode
bin/hdfs namenode -format

# running on namenode
sbin/hadoop-daemon.sh start namenode

# running on datanodes
sbin/hadoop-daemon.sh start datanode

# Alternatively, 如果你设置了etc/hadoop/slaves 所有的hdfs processes can be bootup use the script
# 这个script会启动namenode和datanodes 它会从namenode上ssh到datanode启动datanode
# 如果需要修改默认ssh配置 需要设置HADOOP_SSH_OPTS环境变量 如: export HADOOP_SSH_OPTS="-p 16688"通过16688端口instead of端口22
sbin/start-dfs.sh

```

## boot up yarn

```sh
# running on resourcemanager
sbin/yarn-daemon.sh start resourcemanager

# running on nodemanager
sbin/yarn-daemon.sh start nodemanager

# Alternatively, 如果你设置了etc/hadoop/slaves 所有的yarn processes can be bootup use the script
# 这个script会启动resourcemanager和nodemanager 它会从resourcemanager上ssh到nodemanager启动nodemanager
# 如果需要修改默认ssh配置 需要设置HADOOP_SSH_OPTS环境变量 如: export HADOOP_SSH_OPTS="-p 16688"通过16688端口instead of端口22
sbin/start-yarn.sh
```

# HADOOP shutdown 过程与启动差不多
## shutdown HDFS
```sh
# running on namenode
sbin/hadoop-daemon.sh start namenode

# running on datanodes
sbin/hadoop-daemon.sh start datanode


# Alternatively, 如果你设置了etc/hadoop/slaves 所有的hdfs processes can be bootup use the script
# 这个script会启动namenode和datanodes 它会从namenode上ssh到datanode启动datanode
# 如果需要修改默认ssh配置 需要设置HADOOP_SSH_OPTS环境变量 如: export HADOOP_SSH_OPTS="-p 16688"通过16688端口instead of端口22
sbin/stop-dfs.sh
```

## shutdown yarn
```sh
# running on resourcemanager
sbin/yarn-daemon.sh stop resourcemanager

# running on nodemanager
sbin/yarn-daemon.sh stop nodemanager

# Alternatively, 如果你设置了etc/hadoop/slaves 所有的yarn processes can be bootup use the script
# 这个script会启动resourcemanager和nodemanager 它会从resourcemanager上ssh到nodemanager启动nodemanager
# 如果需要修改默认ssh配置 需要设置HADOOP_SSH_OPTS环境变量 如: export HADOOP_SSH_OPTS="-p 16688"通过16688端口instead of端口22
sbin/stop-yarn.sh
```


