====

安装hadoop和yarn步骤


# 安装java 设置 JAVA_HOME
```sh
cat /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_66
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
```

# 配置文件修改,主要涉及到core-site.xml, hdfs-site.xml, mapred-site.xml和yarn-site.xml

## core-site.xml 设置

core-site 有个默认设置可以参考官方文档[链接](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://10.0.2.6:9000</value>
    </property>

    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>

    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop</value>
    </property>
</configuration>
```
