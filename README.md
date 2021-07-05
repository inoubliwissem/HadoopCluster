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

## Some deyails about the memeory allocation
The Memory allocation configuration is a primordial step, this configuration allows us to make good use of the cluster  because default values are not suitable for nodes with less than 8GB of RAM. This section will highlight how memory allocation works for MapReduce jobs.
#### The Memory Allocation Properties
A YARN job is executed with two kind of resources:
-   An Application Master (AM) is responsible for monitoring the application and coordinating distributed executors in the cluster.
-   Some executors that are created by the AM actually run the job. For a MapReduce jobs, they’ll perform map or reduce operation, in parallel.
Both are run in  _containers_  on worker nodes. Each worker node runs a  _NodeManager_  daemon that’s responsible for container creation on the node. The whole cluster is managed by a  _ResourceManager_  that schedules container allocation on all the worker-nodes, depending on capacity requirements and current charge.

Four types of resource allocations need to be configured properly for the cluster to work. These are:
1.  How much memory can be allocated for YARN containers on a single node. This limit should be higher than all the others; otherwise, container allocation will be rejected and applications will fail. However, it should not be the entire amount of RAM on the node.
    
    This value is configured in  **yarn-site.xml**  with  **yarn.nodemanager.resource.memory-mb**.
    
2.  How much memory a single container can consume and the minimum memory allocation allowed. A container will never be bigger than the maximum, or else allocation will fail and will always be allocated as a multiple of the minimum amount of RAM.
    
    Those values are configured in  **yarn-site.xml** with  **yarn.scheduler.maximum-allocation-mb**  and  **yarn.scheduler.minimum-allocation-mb**.
    
3.  How much memory will be allocated to the ApplicationMaster. This is a constant value that should fit in the container maximum size.
    
    This is configured in  **mapred-site.xml**  with  **yarn.app.mapreduce.am.resource.mb**.
    
4.  How much memory will be allocated to each map or reduce operation. This should be less than the maximum size.
    
    This is configured in  **mapred-site.xml**  with properties  **mapreduce.map.memory.mb** and  **mapreduce.reduce.memory.mb**.

The relationship between all those properties can be seen in the following figure:





#### Configure Workers
In the first steps of this tutorial we:
	*  Created a new user for hadoop  (all machines)
	*  Added all ip adresse into the hostname file (all machines)
	*  Installed java on all machine (all machines)
	* Genreated a pair of keys using RSA and share it with all machines
	* Configured hadoop files on master machine
Now we scopy the configureted hadoop reposetory into all slave machines:

```{r, engine='bash', count_lines}
scp -r hadoop slave1@home/winoubli
scp -r hadoop slave2@home/winoubli
scp -r hadoop slave3@home/winoubli
```
## Using and administration of the Hadoop cluster
## HDFS
### Framat HDFS
Like the classic file system before using it be formated in order to check the metadata, in Hadoop, we can format  HDFS using the next command:
```{r, engine='bash', count_lines}
hdfs namenode -format
```
### Start and stop Hadoop services (deamons)
As we descussed in the Hadoop architecture, Hadoop provides tow services Yarn and HDFS, for each services deamons be started in both master and slaves machines.
In order to start the HDFS service, we run the next script on the master machine:
```{r, engine='bash', count_lines}
start-dfs.sh
```
This script will start **NameNode** and **SecondaryNameNode** on master machine , and **DataNode** on  slave machines. 
We can check that the deamons have been started and are  running, using  the **jps** command on each node we can check that.
 On **master machine **, you should see the following deamons (the PID number will be different):
 ```{r, engine='bash', count_lines}
90437 Jps
90440 NameNode
90441 SecondaryNameNode
```
And on *slave machines ** we should see the following deamons:
 ```{r, engine='bash', count_lines}
91576 Jps
91580 DataNode
```
To stop HDFS cluster run the following command  from **master machine**:

 ```{r, engine='bash', count_lines}
stop-dfs.sh
```
Also HDFS provide a web user interface in order to manage the HDFS cluster, this interface can be accecible using this URL: http://mastermachineIP:9870


## YARN
### Start and Stop YARN
 ```{r, engine='bash', count_lines}
start-yarn.sh
```
This script will start **ResourceManager** on master machine , and **NodeManager** on  slave machines. 
We can check that the deamons have been started and are  running, using  the `jps` command on each node we can check that.
 On **master machine **, you should see the following deamons (the PID number will be different):
 ```{r, engine='bash', count_lines}
90437 Jps
90440 ResourceManager
```
And on *slave machines ** we should see the following deamons:
 ```{r, engine='bash', count_lines}
91576 Jps
91580 NodeManager
```
### Monitor YARN
The **yarn** command provides utilities to manage your YARN cluster. You can also print a report of running nodes with the command:
 ```{r, engine='bash', count_lines}
yarn node -list
```
Also using **yarn** we can get a list of running applications using the newt command:
 ```{r, engine='bash', count_lines}
yarn application -list
```
As with HDFS, YARN provides a web user interface that will be accessible using this URL: http://mastermachineIP:8088/cluster
