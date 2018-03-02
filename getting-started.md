---
title: Quick Start Guide
description: Run a single-node Harp on your machine
aliases:
  - /docs/install.html
---

These instructions have only been tested on:

* Mac OS X
* Ubuntu

If you are using windows, we suggest you to install an Ubuntu system on a virtualization software (e.g. VirtualBox) with at least 4GB memory in it.

## Step 1 --- Install Hadoop 2.6.0

1. Make sure your computer can use `ssh` to access `localhost` and can install `Java`, `vim`, and `Maven` When required.

2. Download and extract the hadoop-2.6.0 binary into your machine. It's available at [hadoop-2.6.0.tar.gz](https://archive.apache.org/dist/hadoop/core/hadoop-2.6.0/hadoop-2.6.0.tar.gz).

3. Set the environment variables in `~/.bashrc`. Access it by using the command `vim ~/.bashrc`.
Note: If you don't have it already, you will need to install `vim` first before you can run this step.
```bash
export JAVA_HOME=<where Java locates>
#e.g. ~/jdk1.8.0_91
export HADOOP_HOME=<where hadoop-2.6.0 locates>
#e.g. ~/hadoop-2.6.0
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$HADOOP_HOME/bin:$JAVA_HOME/bin:$PATH
```
4. Run `$ source ~/.bashrc` to make sure the changes are applied.

5. Check if environment variabls are set correctly by running `$hadoop`. The results should look similar to the example below.
```bash
$ hadoop
Usage: hadoop [--config confdir] COMMAND
       where COMMAND is one of:
  fs                   run a generic filesystem user client
  version              print the version
  jar <jar>            run a jar file
  checknative [-a|-h]  check native hadoop and compression libraries availability
  distcp <srcurl> <desturl> copy file or directories recursively
  archive -archiveName NAME -p <parent path> <src>* <dest> create a hadoop archive
  classpath            prints the class path needed to get the
  credential           interact with credential providers
                       Hadoop jar and the required libraries
  daemonlog            get/set the log level for each daemon
  trace                view and modify Hadoop tracing settings
 or
  CLASSNAME            run the class named CLASSNAME
Most commands print help when invoked w/o parameters.
```

6. Follow steps a-d to modify the following files in Apache Hadoop distribution.

  a.`vim $HADOOP_HOME/etc/hadoop/core-site.xml`:
```html
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9010</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/tmp/hadoop-${user.name}</value>
    <description>A base for other temporary directories.</description>
  </property>
</configuration>
```
 
   b.`vim $HADOOP_HOME/etc/hadoop/hdfs-site.xml`:
```html
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

   c.`vim $HADOOP_HOME/etc/hadoop/mapred-site.xml`:
You will be creating this file. It doesn’t exist in the original package.
```html
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.resource.mb</name>
    <value>512</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.command-opts</name>
    <value>-Xmx256m -Xms256m</value>
  </property>
</configuration>
```

   d.`vim $HADOOP_HOME/etc/hadoop/yarn-site.xml`:
```html
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>4096</value>
  </property>
  <property>
    <description>Whether virtual memory limits will be enforced for containers.</description>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
  </property>
  <property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>512</value>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>2048</value>
  </property>
</configuration>
```

7. Format the file system using the code `$ hdfs namenode -format`. You should be able to see it exits with status 0. The results should look similar to the example below.
```bash
$ hdfs namenode -format
...
xx/xx/xx xx:xx:xx INFO util.ExitUtil: Exiting with status 0
xx/xx/xx xx:xx:xx INFO namenode.NameNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at xxx.xxx.xxx.xxx
```

8. Run the two lines of code shown below separatly. This will Launch NameNode daemon, DataNode daemon, ResourceManager daemon and NodeManager Daemon.
```bash
$ $HADOOP_HOME/sbin/start-dfs.sh
$ $HADOOP_HOME/sbin/start-yarn.sh
```

9. Check if the daemons started successfully by running `$ jps`. The output should look like the following except with actual inputs for 'NameNode', "SecondaryNameNode', etc. If it doesn't look like the example, then you will need to recheck steps 6 through 8 again.
```bash
$ jps
xxxxx NameNode
xxxxx SecondaryNameNode
xxxxx DataNode
xxxxx NodeManager
xxxxx Jps
xxxxx ResourceManager
```

10. You can browse the web interface for the NameNode at [http://localhost:50070](http://localhost:50070) and for the ResourceManager at [http://localhost:8080](http://localhost:8080).

## Step 2 --- Install Harp

1. Clone Harp repository. It is available at [DSC-SPIDAL/harp](https://github.com/DSC-SPIDAL/harp.git).
```bash
git clone https://github.com/DSC-SPIDAL/harp.git
```

2. Follow the [maven official instruction](http://maven.apache.org/install.html) to install maven.

3. Add environment variables to `~/.bashrc`.  Access it by using the command `vim ~/.bashrc`.
```bash
export HARP_ROOT_DIR=<where Harp locates>
#e.g. harp/harp-project
export HARP_HOME=$HARP_ROOT_DIR/harp-project
```
4. Run the same source command `source ~/.bashrc` to set the envrionment variables.

5. If hadoop is still running, stop it first with the following code:
```bash
$ $HADOOP_HOME/sbin/stop-dfs.sh
$ $HADOOP_HOME/sbin/stop-yarn.sh
```

6. Enter "harp" home directory useing the code `cd $HARP_ROOT_DIR`.

7. Compile harp using the code `mvn clean package` Note: If you don't have it already, you will need to isntall Maven to run this step.

8. Install harp plugin to hadoop as demonstrated in the example below:
```bash
cp harp-project/target/harp-project-1.0-SNAPSHOT.jar $HADOOP_HOME/share/hadoop/mapreduce/
cp third_party/fastutil-7.0.13.jar $HADOOP_HOME/share/hadoop/mapreduce/
```

9. Edit mapred-site.xml in $HADOOP_HOME/etc/hadoop by using the code `vim $HADOOP_HOME/etc/hadoop`, add java opts settings for map-collective tasks. For example:
  ```xml
   <property>
     <name>mapreduce.map.collective.memory.mb</name>
     <value>512</value>
   </property>
   <property>
     <name>mapreduce.map.collective.java.opts</name>
     <value>-Xmx256m -Xms256m</value>
   </property>
   ```

10. Now you've finished the instalation! To develop Harp applications, remember to add the following property in job configuration:
```bash
jobConf.set("mapreduce.framework.name", "map-collective");
```

## Step 3 Run harp kmeans example

1. Copy harp examples to $HADOOP_HOME useing the code `cp harp-app/target/harp-app-1.0-SNAPSHOT.jar $HADOOP_HOME`

2. Start Hadoop. You will first want to move to the $HADOOP_HOME location as demonstrated in the example below.
```bash
    cd $HADOOP_HOME
    sbin/start-dfs.sh
    sbin/start-yarn.sh
```

3. Run Kmeans Map-collective job.
The usage is
   ```bash
   hadoop jar harp-app-1.0-SNAPSHOT.jar edu.iu.kmeans.regroupallgather.KMeansLauncher <num of points> <num of centroids> <vector size> <num of point files per worker> <number of map tasks> <num threads> <number of iteration> <work dir> <local points dir>
   ```
   * `<num of points>` --- the number of data points you want to generate randomly
   * `<num of centriods>` --- the number of centroids you want to clustering the data to
   * `<vector size>` --- the number of dimension of the data
   * `<num of point files per worker>` --- how many files which contain data points in each worker
   * `<number of map tasks>` --- number of map tasks
   * `<num threads>` --- how many threads to launch in each worker
   * `<number of iteration>` --- the number of iterations to run
   * `<work dir>` --- the root directory for this running in HDFS
   * `<local points dir>` --- the harp kmeans will firstly generate files which contain data points to local directory. Set this argument to determine the local directory.

    For example:
   ```bash
   hadoop jar harp-app-1.0-SNAPSHOT.jar edu.iu.kmeans.regroupallgather.KMeansLauncher 1000 10 100 5 2 2 10 /kmeans /tmp/kmeans
   ```

4. To fetch the results, use the following command:
```bash
$ hdfs dfs –get <work dir> <local dir>
#e.g. hdfs dfs -get /kmeans ~/Document
```
