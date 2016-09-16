# Set Up Hadoop Distributed File System

**Hadoop** is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models[4]. It includes 

- Hadoop common
- Hadoop Distributed FIle System (HDFS)
- Hadoop YARN
- Hadoop MapReduce.

For our cluster, we only need the **HDFS** part working as a distributed data storage. **MapReduce** will **NOT** be covered. Instead, we use **Spark** to do the distributed computing.

Why we use HDFS? It can help sava huge amount of data that can't be handled by single machine. Additionally, Hadoop will take the responsibility of scaling, fault toelerance, and resources management. You can add more nodes seamlessly to save & process more data; You don't need to worry about your data even if any one of your nodes fails.

To install Hadoop [5], there are a few steps to follow.


<br><br>


## 1. Install Java

the first step is to install Java

```{bash}
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

You can use 

```{bash}
java -version
```
to confirm Java is installed properly.


<br><br>


## 2. Password-less Access to Workers

Hadoop uses SSH to communicate between master node and each of the worker nodes. How to set up this was introduce in section ***Network Basics***.

<br><br>


## 3. Download & Install Hadoop

You can download the binary package of Hadoop from link [http://www-us.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz](http://www-us.apache.org/dist/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz). But it's more recommended to download from the mirror site which is the closest to you. Myself used the mirror in Singapore. The mirror list can be found [here](http://www.apache.org/mirrors/).

```{bash}
wget http://mirror.nus.edu.sg/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz 

# Unzip the downloaded file
tar xfz hadoop-2.7.3.tar.gz 

# Move the Hadoop installation to path /usr/local/hadoop
mv hadoop-2.7.3 /usr/local/hadoop
```

<br><br>


## 4. Edit Hadoop Configuration Files

There are a few configration files that we need to modify. Not all of them are compulsory for our needs since we would only use the HDFS feature while some configurations are for MapReduce. But we would still cover all of them here for completeness.

- **~/.bashrc**:
- **<hadoop_directory>/etc/hadoop/hadoop-env.sh**: To set Hadoop-specific environment variables, including *JAVA_HOME*.
- **<hadoop_directory>/etc/hadoop/core-site.xml**: To set site-specific properties, including *fs.default.name*, which contains the path and port of the namenode. 
- **<hadoop_directory>/etc/hadoop/hdfs-site.xml**: This has to be configured for each node in the cluster. We will need to specify the directiories which are used as namenode (on master node) and the datanode (on all nodes).
- **<hadoop_directory>/etc/hadoop/yarn-site.xml**:
- **<hadoop_directory>/etc/hadoop/mapred-site.xml**:
- **<hadoop_directory>/etc/hadoop/slaves**: for master node only.



### ~/.bashrc

The main purposes for editing `.bashrc` include

- Setting `JAVA_HOME`
- Include the path of Hadoop directory into `$PATH` so that we can invoke Hadoop more conveniently.

First we need to find the Java path by executing

```{bash}
update-alternatives --config java
```

On my machine, the result is `/usr/lib/jvm/java-8-openjdk-armhf/jre/bin/java`, so that the `JAVA_HOME` should be `/usr/lib/jvm/java-8-openjdk-armhf`. 

Then we can edit `.bashrc` file. What we need to do is to append the following content at the end of the `.bashrc` file.

```
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
```

After saving your change, executing 

```{bash}
source ~/.bashrc
```

### hadoop-env.sh

For `hadoop-env.sh`, what we need to amend is the line that exports `JAVA_HOME` variable. Change it into the corresponding value on your machines. For my machine, I made the amendment as below

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-armhf
```

### core-site.xml

In `core-site.xml`, we need to specify the namenode. Enter the following content in between the tags `<configuration>` and `</configuration>`.

```
<property>
   <name>fs.default.name</name>
   <value>hdfs://<URI_of_Namenode>:<Port_of_Namenode></value>
</property>
```

**Note**: If you're gonna start the namenode on your local machine, use `<value>hdfs://localhost:9000</value>`. However, if you're gonna run a multi-node cluster, please do remember to use `<value>hdfs://<URI_of_Nanmenode>:<Port_you_desire></value>` on **ALL** your nodes.

For my cluster, I used

```
<property>
   <name>fs.default.name</name>
   <value>hdfs://172.24.210.100:9000</value>
</property>
```
on **ALL** of my nodes.



### hdfs-site.xml

`hdfs-site.xml` has to been configured on each of your nodes. It helps specify the directories which are used as *namenode* and *datanode* on each nodes.

First, let's create two directories for *namenode* and *datanode*.

```
mkdir -p /usr/local/hadoop_store/hdfs/namenode

mkdir -p /usr/local/hadoop_store/hdfs/datanode

```

Of course yo can choose any other locations to creat these two directories, but do remember to make amendment in `hdfs-site.xml` accordingly [7].

Then add the content below between tags `<configuration>` and `</configuration>` in `hdfs-site.xml`.

```
<property>
   <name>dfs.replication</name>
   <value>2</value>
 </property>
 <property>
   <name>dfs.namenode.name.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/namenode</value>
 </property>
 <property>
   <name>dfs.datanode.data.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/datanode</value>
 </property>
```

The another property we have set is `dfs.replication`. It's the number of replications you're gonna to have for each block in your HDFS. The default value is 3. The actual number of replications can also be specified when the file is created [8].



### yarn-site.xml

Similarly, add the content below between tags `<configuration>` and `</configuration>` in `yarn-site.xml`

```
<property>
   <name>yarn.nodemanager.aux-services</name>
   <value>mapreduce_shuffle</value>
</property>
<property>
   <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
   <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```

### mapred-site.xml

For `mapred-site.xml`, you may need to create it based on the template given `mapred-site.xml.template`. After you create `mapred-site.xml` under path `etc/hadoop/`, similarly, add the content below between tags `<configuration>` and `</configuration>` in `mapred-site.xml`

```
<property>
   <name>mapreduce.framework.name</name>
   <value>yarn</value>
</property>
```


<br><br>


## 5. Format the New File System

AFTER we finish the configurations and BEFORE we can really use the HDFS, we need to format the Hadoop file system by executing the command

```{bash}
hdfs namenode -format
```

**NOTE**: This step is only needed when you start your new HDFS. It will re-build your file system and all existing data will be erased.




<br><br>


## 6. Start HDFS

Okay, now it's time to launch your Hadoop cluster by simply executing

```{bash}
start-dfs.sh
```

It will start `NameNode` on your master node and `DataNode` on all worker nodes listed in `etc/hadoop/slaves`, plus `SecondaryNameNode` on master node. Actually from now on, the HDFS is already working and you can already put data into it or get data from it.

To use the resource management feature, then you can execute

```{bash}
start-yarn.sh
```

It will start `ResourceManager` on your master node and `NodeManager` on all nodes.

Please note that you can use **`jps`** command to check if the processes mentioned above are running. If not, please check the log to find out what's wrong.

![jps_checking](https://github.com/XD-DENG/DIY-A-Cluster/raw/master/images/jps_checking.png)


<br><br>


## 7. Use Hadoop Shell

After the configuration and starting the cluster, we can already fetch data from HDFS or put data into it. There are multiple ways to do this, like Java, Spark (which will be introduced in this document too), etc. But I would like to introduce the most basic one here, Haddop File System (FS) Shell. 

The File System (FS) shell includes various shell-like commands that directly interact with HDFS [7]. It's invoked by

```{bash}
# <hadoop_directory>/bin
hadoop fs <args>
```

You may be confused about the difference between `hadoop fs` and `hadoop dfs`. It's quite simple: if HDFS is being used, `hdfs dfs` is a synonym.

As the FS shell is very straightforward (as long as you are familiar with basic shell operation on Linux), I would only give a few most basic and common-use commands here.

### **ls**

```{bash}
hadoop fs -ls [-h] [-R] <args>
```
- -h: Format file sizes in a human-readable fashion (eg 64.0m instead of 67108864).
- -R: Recursively list subdirectories encountered.

For example, to list the files and folders in my local HDFS, I can enter

```{bash}
hadoop fs -ls /
```

### **mkdir**

We can use this command to make a directory in HDFS. For example, I want to build a directory named "test" under the root directory in my HDFS.

```{bash}
hadoop fs -mkdir /test
```

### **mv**

The same as the command with the same name in Linux, `mv` can be used to move file or folder, or rename a file or folder, like

```{bash}
hadoop fs -mv /test/file1 /test/file2
```

### **put**

`put` is used to dump your local data into HDFS. For instance, I want to put my `syslog` into the path `/test` of my HDFS, 

```{bash}
hadoop fs -put /var/log/syslog /test/
```

### **get**

In contrast to `put`, `get` helps fetch data from HDFS. For instance, I'd like to copy file `hdfs_file1` to my local machine and rename it as `localfile1`,

```{bash}
hadoop fs -get /hdfs_file1 localfile1
```

There are many other commands available, like `rm`, `checksum`, `chmod`, etc. You can directly refer to [7].


<br><br>


## 8. Stop HDFS

To stop the Hadopp cluster in a safe fashion, you can execute

```{bash}
stop-yarn.sh # if you have run start-yarn.sh

stop-dfs.sh
```

You can execute `jps` again, and you should have found that only process `Jps` is running now.

