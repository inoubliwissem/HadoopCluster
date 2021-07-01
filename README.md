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
#### Configure ssh between all machines
