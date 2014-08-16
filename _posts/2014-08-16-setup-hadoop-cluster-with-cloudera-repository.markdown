--------------
title: Setup Hadoop cluster with Cloudera Repository
layout: post
-------------
Install HDFS Cluster
====================

##### Install CDH 5 Repository

```
wget http://archive.cloudera.com/cdh5/one-click-install/redhat/6/x86_64/cloudera-cdh-5-0.x86_64.rpm
sudo yum --nogpgcheck localinstall cloudera-cdh-5-0.x86_64.rpm
```

##### Install ZooKeeper

```
sudo yum install zookeeper
sudo yum install zookeeper-server
sudo service zookeeper-server init -- After new installation only
sudo service zookeeper-server start
```

##### Install Resource Manager (MapReduce node)

```
sudo yum clean all; sudo yum install hadoop-yarn-resourcemanager
```

##### Install Name Node 

```
sudo yum clean all; sudo yum install hadoop-hdfs-namenode
```

##### Install Secondary NameNode host (if used)

```
sudo yum clean all; sudo yum install hadoop-hdfs-secondarynamenode
```

##### Install All cluster hosts except the Resource Manager

```
sudo yum clean all; sudo yum install hadoop-yarn-nodemanager hadoop-hdfs-datanode hadoop-mapreduce
```

##### One host in the cluster

```
sudo yum clean all; sudo yum install hadoop-mapreduce-historyserver hadoop-yarn-proxyserver
```

##### Install Hadoop client on machine that connect to hadoop

```
sudo yum clean all; sudo yum install hadoop-client
```

Config HDFS
===========

### Change hadoop default conf directory (using `alternatives`)

1. Copy the default configuration to your custom directory:

```
 sudo cp -r /etc/hadoop/conf.empty /etc/hadoop/conf.my_cluster
 ```

2. CDH uses the alternatives setting to determine which Hadoop configuration to use. 
Set alternatives to point to your custom directory, as follows.
To manually set the configuration on Red Hat-compatible systems:

```
sudo alternatives --install /etc/hadoop/conf hadoop-conf /etc/hadoop/conf.my_cluster 50 
sudo alternatives --set hadoop-conf /etc/hadoop/conf.my_cluster
```

### Config cluster

Update your conf directory following below steps and upload to your host

#### Config Namenode address and permission (on every host)

1. core-site.xml

`fs.defaultFS`: Namenode address, port

```
<property>
 <name>fs.defaultFS</name>
 <value>hdfs://namenode-host.company.com:8020</value>
</property>
```

2. hdfs-site.xml

`dfs.permissions.superusergroup` Specifies the UNIX group containing users that will be treated as superusers by HDFS. You can stick with the value of 'hadoop' or pick your own group depending on the security policies at your site

```
<property>
 <name>dfs.permissions.superusergroup</name>
 <value>hadoop</value>
</property>
```

#### Config Namenode data directory

1. `hdfs-site.xml` on the NameNode

`dfs.name.dir` or `dfs.namenode.name.dir` This property specifies the URIs of the directories where the NameNode stores its metadata and edit logs. Cloudera recommends that you specify at least two directories. One of these should be located on an NFS mount point, unless you will be using a High Availability (HA) configuration.
*Ensure that configured directory are existing and accessible (drwx------) on corresponding host*

```
<property>
 <name>dfs.namenode.name.dir</name>
 <value>file:///data/1/dfs/nn,file:///nfsmount/dfs/nn</value>
</property>
```

2. `hdfs-site.xml` on each DataNode

`dfs.data.dir` or `dfs.datanode.data.dir` This property specifies the URIs of the directories where the DataNode stores blocks. Cloudera recommends that you configure the disks on the DataNode in a JBOD configuration, mounted at /data/1/ through /data/N, and configure dfs.data.dir or dfs.datanode.data.dir to specify file:///data/1/dfs/dn through file:///data/N/dfs/dn/.
*Ensure that configured directory are existing and accessible (drwx------) on corresponding host*

```
<property>
 <name>dfs.datanode.data.dir</name>
 <value>file:///data/1/dfs/dn,file:///data/2/dfs/dn,file:///data/3/dfs/dn,file:///data/4/dfs/dn</value>
</property>
```

#### Formatting Namenode host

```
sudo -u hdfs hdfs namenode -format
```

#### Enabling WEBHDFS on Namenode

1. Update or add following to `hdfs-site.xml`

```
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>
```

### Start DFS

##### Push your custom directory (for example /etc/hadoop/conf.my_cluster) to each node in your cluster; for example:

```
scp -r /etc/hadoop/conf.my_cluster myuser@myCDHnode-<n>.mycompany.com:/etc/hadoop/conf.my_cluster
```

##### Manually set alternatives on each node to point to that directory, as follows.

```
sudo alternatives --verbose --install /etc/hadoop/conf hadoop-conf /etc/hadoop/conf.my_cluster 50 
sudo alternatives --set hadoop-conf /etc/hadoop/conf.my_cluster
```

##### Start

```
for x in `cd /etc/init.d ; ls hadoop-hdfs-*` ; do sudo service $x start ; done
```

*Make sure your hdfs `/tmp` directory exists and has correct permission*

```
sudo -u hdfs hadoop fs -mkdir /tmp
sudo -u hdfs hadoop fs -chmod -R 1777 /tmp
```

Config & deploy YARN
====================

##### Start ResourceManager service (on ResourceManager host)

```
sudo service hadoop-yarn-resourcemanager start
```

##### Start NodeManager (on every DataNode)

```
sudo service hadoop-mapreduce-historyserver start
```

##### Create hdfs home directory for each user (LINUX user)

```
sudo -u hdfs hadoop fs -mkdir  /user/{user}
sudo -u hdfs hadoop fs -chown {user} /user/{user}
```


Config hadoop daemmon start at boot
===================================

[link] http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_init_configure.html?scroll=topic_27_2

```
sudo chkconfig zookeeper-server on
sudo chkconfig hadoop-hdfs-namenode on
sudo chkconfig hadoop-yarn-resourcemanager on
sudo chkconfig hadoop-hdfs-secondarynamenode on
sudo chkconfig hadoop-yarn-nodemanager on
sudo chkconfig hadoop-hdfs-datanode on
sudo chkconfig hadoop-mapreduce-historyserver on
```

Install Supporting Components
=============================



















