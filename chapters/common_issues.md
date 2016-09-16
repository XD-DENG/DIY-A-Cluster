
## Common Issues

1. Make sure the owner of the Hadoop data folder is the same as the user name you used to launch the HDFS service, instead of `root`. Otherwise you may fail the launch the HDFS service or fail to fetch fata from the HDFS.
2. When we use the HDFS, sometimes we may encounter error like "connection refused", even if we confirm the hadoop cluster is running. This error is quite likely due to wrong configuration of hostnames. I encountered the error when I set the `fs.default.name` to be "hdfs://0.0.0.0:9000" on the master node (I have set it to be `hdfs://<master-node-ip>:9000` on the worker nodes). The error was gone after I set the `fs.default.name` to be `hdfs://<master-node-ip>:9000` on the master node too. 


