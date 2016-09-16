
## Set Up Apache Spark

### Install Spark

The setting up of Apache Spark is quite straightforward. First, make sure the Java JDK is installed (we've already done it when we set up HDFS).

```{bash}
sudo apt-get install openjdk-8-jdk
```

Next step is to download pre-built package of Spark at http://spark.apache.org/downloads.html. Unzip the package and check if you can run command

```{bash}
cd <spark_path>

./bin/pyspark
```

If you see the Spark logo in the terminal, you've installed Apache Spark (single machine node) successfully.

![pyspark_running](https://github.com/XD-DENG/DIY-A-Cluster/raw/master/images/spark_running.png)


This needs to be done on all the machines. 


<br><br>


### Start Spark Cluster


Then we can try to start the Spark cluster. A Spark cluster is made up of one master node and a few worker nodes (master node can work as worker node too). Here we would start the cluster in a more "manual" way.

**NOTE**: before you try to start the cluster, do make sure your master machine can access all the woker machines in password-less way using private key. This was introduced in [Network Basics](https://github.com/XD-DENG/DIY-A-Cluster/blob/master/chapters/network_basics.md).

First, we start the master process on your master machine

```{bash}
# in the Spark path
# here we use the IP address directly insead of using hostname
./sbin/start-master.sh -h 172.24.210.100
```
Then we start worker process on worker machines by running the command below on your worker machines

```{bash}
# in the Spark path
# ./sbin/start-slave.sh <master-spark-URL>
./sbin/start-slave.sh spark://172.24.210.100:7077
```

Then we can use web UI http://172.24.210.100:8080 to check whether the master is started and whether the workers have joined the cluster.

![spark_webUI](https://github.com/XD-DENG/DIY-A-Cluster/raw/master/images/spark_webUI.png)


<br><br>



### Invoke Spark Cluster


To use the cluster interactively, enter

```{bash}
# PySpkark
./bin/pyspark --master spark://172.24.210.100:7077
```
or

```{bash}
# Scala
./bin/spark-shell --master spark://172.24.210.100:7077
```

To submit a task to the cluster, enter

```{bash}
./bin/spark-submit \
  --class <main-class> \
  --master <master-url> \   # if this is left blank, the application will be run locally instead of invoking the cluster
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  ... # other options
  <application-jar> \
  [application-arguments]
```

`spark-submit` will automatically identify which type of application this is (Scala, Java, or Python). Some of the commonly used options are [2]:

- **--class**: The entry point for your application (e.g. org.apache.spark.examples.SparkPi)
- **--master**: The master URL for the cluster (e.g. spark://23.195.26.187:7077)
- **--deploy-mode**: Whether to deploy your driver on the worker nodes (cluster) or locally as an external client (client) (default: client) †
- **--conf**: Arbitrary Spark configuration property in key=value format. For values that contain spaces wrap “key=value” in quotes (as shown).
- **application-jar**: Path to a bundled jar including your application and all dependencies. The URL must be globally visible inside of your cluster, for instance, an hdfs:// path or a file:// path that is present on all nodes.
- **application-arguments**: Arguments passed to the main method of your main class, if any

For our case, we can test the cluster by running an example using

```{bash}
./bin/spark-submit --master spark://172.24.210.100:7077 examples/src/main/python/pi.py 8
```


<br><br>


### Stop Spark Cluster

We still use the "manual" way to do this.

On the worker nodes, execute

```{bash}
./sbin/stop-slave.sh
```

Then execute the command below on the master node

```{bash}
./sbin/stop-master.sh
```
