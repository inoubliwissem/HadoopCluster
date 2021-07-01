#  Install and Set Up a Hadoop Cluster!
![alt text](hadoop.jpg "apache hadoop")
This guide present how we can install and configure hadoop on clusters of machines.  Firstly, an overview will be presented in order to understand the main features of this software, and global architecture will be presented.

## Hadoop Overviw
### What is Hadoop?
Hadoop is an open-source Apache project that implements two distributed frameworks that was introduced by Google in 2004, these frameworks are designed for both  parallel processing applications (**Mapreduce**) and distributed file system for  large data sets (**GFS**). Therefore Hadoop  provides an implementation of Mapreduce framework and a distibuted file system which is called **Hadoop Distributed File System (HDFS) ** . Today lots of Big Companies ( Facebook, Yahoo, Netflix, eBay, etc)  are using Apache Hadoop to deal with big data in several use cases.
Apache Haddop provides as a software is composed of the following components:
1) **Hadoop Common** contains libraries and utilities needed by other Hadoop modules.
2) **Hadoop Distributed File System (HDFS) ** a distributed file system that stores data on commodity machines, providing very high aggregate bandwidth across the cluster and fault tolerance.
3)  **Hadoop YARN** (Yet Another Resource Negotiator was introduced sience  2012 )  is a cluster manager that supervises and manages computing resources in clusters and also ensure  scheduling users’ applications.
4) **Hadoop MapReduce** an implementation of the MapReduce framework.

### Hadoop Architecture
From an architectural point of view, Hadoop is based on muli-nodes (machines) and  Master/Slave (or Controller/Peripharal)  architecture , where the  master node is  responsible for the coordination between the slave machines and slave nodes  running the requested query by the master node. In the following the hodoop's services and their :
1) **Resource Manager** (master/YARN): manages the YARN/Mapreduce jobs and ensure the scheduling and executing processes of submiteed application  on slave nodes.
2)  **Node Manager** (slave/yarn): manages execution of tasks on the node
3) **NameNode**  (master/HDFS): manages the distributed file system and knows where stored data blocks inside the cluster are.
4) **DataNode** (slave/HDFS) manages the physical data stored on the node.
5) **SecondryNameNode** (master/HDFS/YARN) : It is a service used by the Resource manager to ensure the backup in order to ensure the fault tolerance. 

## Configuration
Let's consider that we have three machines with an Ubuntu operating system.
##### Create a new user
Create a new user and change as a root user
```
sudo adduser huser
sudo adduser huser sudo
```
##### Install java on all machines 
```
sudo apt install openjdk-8-jdk
```
##### Edit hostname file 
For each node to communicate with each other by name, open the  /etc/hosts file and  add the private IP addresses of the three machine.

```sudo nano /etc/hots```
Put the following lines:
```
10.0.2.1 master
10.0.2.2 slave1 
10.0.2.3 slave2
```
####  Setup  SSH Passwordless 
##### **Install the SSH server and client** (all machines)
```{r, engine='bash', count_lines}
sudo apt-get install openssh-client
sudo apt-get install openssh-server
```
Create a new OpenSSL key pair with the following commands:
```{r, engine='bash', count_lines}
$ ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/home/winoubli/.ssh/id_rsa):
**Created directory '/home/winoubli/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:
Your identification has been saved in /home/winoubli/.ssh/id_rsa.
Your public key has been saved in /home/winoubli/.ssh/id_rsa.pub.
```
Enable **ssh** to access to the local machine with adding the public key into the **authorized_keys** file
```{r, engine='bash', count_lines}
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
```
Now we can test it this configuration is done using this command in order to access to the local machine without password
```{r, engine='bash', count_lines}
ssh localhost
```
Share the **.ssh** folder to all other slave machines
```{r, engine='bash', count_lines}
scp -r .ssh winoubli@slave1:/home/winoubli
scp -r .ssh winoubli@slave2:/home/winoubli
```
Verify that can connect to machine without password
```{r, engine='bash', count_lines}
ssh slave1
```
### Download and extract the last binary and stable version of hadoop
```{r, engine='bash', count_lines}
wget https://mirror.ibcp.fr/pub/apache/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz
tra xvf hadoop-3.2.2.tar.gz
mv hadoop-3.2.2 hadoop
```
### set up hadoop 
#### Set Environment Variables
Add Hadoop binaries to the PATH. Edit `/home/winoubli/.profile` and add the following line:
```{r, engine='bash', count_lines}
nano /home/winoubli/.profile
```
```{r, engine='bash', count_lines}
PATH=/home/winoubli/hadoop/bin:/home/winoubli/hadoop/sbin:$PATH
```
Add Hadoop to your PATH for the shell. Edit `.bashrc` and add the following lines:
```{r, engine='bash', count_lines}
nano /home/winoubli/.bashrc
```
```{r, engine='bash', count_lines}
export HADOOP_HOME=/home/winoubli/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
```
#### Haddop configuration
ALL configurations will be performed on **master** and replicated to other slaves.
Edit `~/hadoop/etc/hadoop/hadoop-env.sh` and replace this line:
```{r, engine='bash', count_lines}
export  JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
```
###### Set NameNode Location
Update  core-site.xml  file under the haddop/etc directory  to set the NameNode location to **master** on port `9000`:
```{r, engine='bash', count_lines}
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```
```{r, engine='bash', count_lines}
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/winoubli/hadoop/hdfs/</value>
    </property>
</configuration>
```
###### Set path for HDFS
Create HDFS folders that can contain both data and metadata used by namenode and datanode
```{r, engine='bash', count_lines}
mkdir -p hdfs/datanode
mkdir -p hdfs/datanode
```
Edit  hdfs-site.xml configuration file:
```{r, engine='bash', count_lines}
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
```{r, engine='bash', count_lines}
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/winoubli/hadoop/hdfs/namenode</value>
    </property>
        <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/winoubli/hadoop/hdfs/datanode</value>
    </property>

</configuration>
```

####  Set YARN as cluster manager

Edit the mapred-site.xml file:
```{r, engine='bash', count_lines}
nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
```{r, engine='bash', count_lines}
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/home/winoubli/hadoop</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/home/winoubli/hadoop</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/home/winoubli/hadoop</value>
    </property>
      <property>
 <name>yarn.app.mapreduce.am.resource.mb</name>
 <description> The amount of memory the MR AppMaster needs (1/8).</description>
 <value>4096</value>
</property>
<property>
 <name>mapreduce.map.memory.mb</name>
 <description> The amount of memory of map operation (mapreduce/2)</description>
 <value>2048</value>
</property>
<property>
 <name>mapreduce.reduce.memory.mb</name>
 <description> The amount of memory of reduce operation (mapreduce/2)</description>
 <value>2048</value>
</property>
</configuration>
```
Edit the yarn-site.xml file:
```{r, engine='bash', count_lines}
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
```{r, engine='bash', count_lines}
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
        <name>yarn.nodemanager.linux-container-executor.nonsecure-mode.limit-users</name>
        <value>false</value>
    </property>

<!-- Site specific YARN configuration properties -->

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property> 
<property>
        <name>yarn.acl.enable</name>
        <value>0</value>
    </property>
<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>tempograph-worker1</value>
    </property>
<property>
 <name>yarn.nodemanager.resource.memory-mb</name>
 <description> Amount of physical memory, in MB, that can be allocated for containers. 28GG/32 3/4</description>
 <value>28672</value>
</property>
<property>
	<name>yarn.scheduler.minimum-allocation-mb</name>
 <description>The minimum allocation for every container request at the RM, in MBs. Memory requests lower than this won't take effect, and the specified value will get allocated at minimum.(1/16)</description>
 <value>2048</value>
</property>
<property>
	<name>yarn.scheduler.maximum-allocation-mb</name>
 <description> The maximum allocation for every container request at the RM, in MBs. Memory requests higher than this won't take effect, and will get capped to this value.</description>
 <value>28672</value>        
</property>
<property>
 <name>yarn.nodemanager.resource.cpu-vcores</name>
 <description> Number of CPU cores that can be allocated for containers. </description>
 <value>8</value>
</property>
<property>
 <name>yarn.resourcemanager.webapp.adress</name>
 <value>0.0.0.0:8088</value>
</property>
</configuration>
```
#### Configure Workers
