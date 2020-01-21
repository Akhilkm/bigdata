# Create a 3 node apache spark cluster over yarn

## Prerequisites
```
3 running ubuntu machines with the below specification
    1. 4 cores
    2. 8 GB RAM
    3. 64-bit (x86)
    4. Inter communication enabled
```

#### Add hostname and ip for all the machines 
```
hostnamectl set-hostname hadoop-server<number>
hostname -F /etc/hostname

echo """
<Ip of hadoop-server1> hadoop-server1
<Ip of hadoop-server2> hadoop-server2
<Ip of hadoop-server3> hadoop-server3
""" > /etc/hosts
```

#### Set Locale and add JAVA_HOME environment variable
```
echo """
LANGUAGE=\"en_US.UTF-8\"
LANG=\"en_US.UTF-8\"
LC_ALL=\"en_US.UTF-8\"
JAVA_HOME=\"/usr/lib/jvm/java-8-openjdk-amd64\"
""" >> /etc/environment

exit and reconnect to all the machines
```

#### Update the OS and install java8
```
apt-get update && apt-get -y upgrade
apt-get install openjdk-8-jdk-headless
```

## Install and configure a three-node Hadoop cluster

#### Create hadoop user in all the nodes
```
adduser hadoop
su hadoop
```

#### Distribute Authentication Key-pairs for the Hadoop User
```
Login to server1
ssh-keygen -b 4096
cat /home/hadoop/.ssh/id_rsa.pub

Copy your key file into the authorized key store of other machines
copy to ~/.ssh/authorized_keys
```

#### Download and Unpack Hadoop Binaries
```
cd
wget http://apachemirror.wuchna.com/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz
tar -zxvf hadoop-3.1.3.tar.gz
rm hadoop-3.1.3.tar.gz
ln -s hadoop-3.1.3 hadoop
```

#### Set Environment Variables
```
Add the below lines to /home/hadoop/.profile
PATH=/home/hadoop/hadoop/bin:/home/hadoop/hadoop/sbin:$PATH

Edit .bashrc
export HADOOP_HOME=/home/hadoop/hadoop
export PATH=${PATH}:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
```


#### Set NameNode Location (Update  ~/hadoop/etc/hadoop/core-site.xml)
```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://hadoop-server1:9000</value>
        </property>
    </configuration>
```

#### Set path for HDFS (Update ~/hadoop/etc/hadoop/hdfs-site.xml)
```
<configuration>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>/home/hadoop/data/nameNode</value>
    </property>

    <property>
            <name>dfs.datanode.data.dir</name>
            <value>/home/hadoop/data/dataNode</value>
    </property>

    <property>
            <name>dfs.replication</name>
            <value>2</value>
    </property>
</configuration>
```

#### Set YARN as Job Scheduler (Update ~/hadoop/etc/hadoop/mapred-site.xml) 
##### Note: This is required when you are running mapreduce jobs in this cluster
```
<configuration>
    <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
    </property>
    <property>
            <name>yarn.app.mapreduce.am.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
            <name>mapreduce.map.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
            <name>mapreduce.reduce.env</name>
            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
</configuration>
```

#### Configure YARN (Update ~/hadoop/etc/hadoop/yarn-site.xml)
```
<configuration>
    <property>
            <name>yarn.acl.enable</name>
            <value>0</value>
    </property>

    <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>hadoop-master1</value>
    </property>

    <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

#### Configure workers (Update ~/hadoop/etc/hadoop/workers)
```
hadoop-server2
hadoop-server3
```
#### Configure Memory Allocation

##### ~/hadoop/etc/hadoop/yarn-site.xml
```
    <property>
            <name>yarn.nodemanager.resource.memory-mb</name>
            <value>3072</value>
    </property>

    <property>
            <name>yarn.scheduler.maximum-allocation-mb</name>
            <value>3072</value>
    </property>

    <property>
            <name>yarn.scheduler.minimum-allocation-mb</name>
            <value>256</value>
    </property>

    <property>
            <name>yarn.nodemanager.vmem-check-enabled</name>
            <value>false</value>
    </property>
```

##### ~/hadoop/etc/hadoop/mapred-site.xml
```
    <property>
            <name>yarn.app.mapreduce.am.resource.mb</name>
            <value>1024</value>
    </property>

    <property>
            <name>mapreduce.map.memory.mb</name>
            <value>512</value>
    </property>

    <property>
            <name>mapreduce.reduce.memory.mb</name>
            <value>512</value>
    </property>
```

## Duplicate Config Files on Each Node
```
cd /home/hadoop/
scp hadoop-*.tar.gz node1:/home/hadoop
scp hadoop-*.tar.gz node2:/home/hadoop
```

## Format and start HDFS
```
hdfs namenode -format
start-dfs.sh
```

## Run YARN
```
start-yarn.sh
```