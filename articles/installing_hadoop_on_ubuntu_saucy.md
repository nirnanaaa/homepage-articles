---
title: Installing Cloudera Hadoop on Ubuntu 13.10 Saucy
date: 2014-03-12 13:14:00 GMT+02
description: "How to install the Cloudera Hadoop distribution on Ubuntu 13.10 Saucy"
keywords:
  - hadoop
  - saucy
  - 13.10
  - cloudera
  - mapreduce
  - wordcount
---

[[http://www.datameer.com/images/technology/hadoop-pic1.png]]

In this tutorial I will describe the required steps for setting up a pseudo-distributed, single-node YARN Hadoop cluster.backed by the Hadoop Distributed File System (HDFS), running on Ubuntu Linux 13.10 codename Saucy.

This tutorial is based on [this](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/) tutorial which is a bit out of date.

Hadoop is a framework written in Java for running applications on large clusters of commodity hardware and incorporates features similar to those of MapReduce and GFS.


The main goal here is to get a simple Hadoop installation so you can play with `MapReduce` and other cool stuff.

This tutorial has been tested with the following versions:

* Ubuntu Linux 13.10 Saucy Salamander
* Cloudera Hadoop 4 (CDH4)


## Prerequisites

### Sun Java 7

Hadoop requires a working Java 1.6+ installation. However in this tutorial I will use Java 1.7 for running Hadoop.

```text
The following information were taken from a Stackoverflow thread: http://stackoverflow.com/questions/16263556/installing-java-7-on-ubuntu
```

```sh
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer
```


This will keep java7 up-to-date:

```sh
fkasper@hadoop:~$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
```


### Install the Cloudera APT Repository

Cloudera provides an one-click-installer, which automatically adds the cloudera apt repository. Simply execute this litte snippet of code:

```sh
wget http://archive.cloudera.com/cdh4/one-click-install/precise/amd64/cdh4-repository_1.0_all.deb
sudo dpkg -i cdh4-repository_1.0_all.deb
rm cdh4-repository_1.0_all.deb
sudo apt-get update
```

## Install Hadoop and HDFS

```sh
sudo apt-get install hadoop-yarn-resourcemanager hadoop-hdfs-namenode hadoop-yarn-nodemanager hadoop-hdfs-datanode hadoop-mapreduce hadoop-mapreduce-historyserver hadoop-yarn-proxyserver hadoop-client
```

This will install the following Hadoop features:

* ResourceManager (analogous to MRv1 JobTracker)
* HDFS NodeManager
* Hadoop MapReduce
* Hadoop HistoryServer
* Hadoop Yarn ProxyServer
* Hadoop Client

And thats it for the installation part.

## Configuration

The main configuration is done in `/etc/hadoop/conf/*-site.xml`.

### HDFS Configuration


First change the host of your HDFS storage:

`core-site.xml`:

```xml
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://localhost/</value>
</property>
```


And set a group for superuser permissions:

`hdfs-site.xml`:

```xml
<property>
  <name>dfs.permissions.superusergroup</name>
  <value>hadoop</value>
</property>
<property>
  <name>dfs.name.dir</name>
  <value>/var/lib/hadoop-hdfs/cache/hdfs/dfs/name</value>
</property>
```


Create and setup the local storage directories used by HDFS:

```sh
sudo mkdir -p /var/lib/hadoop-hdfs/cache/hdfs/dfs/name
sudo chown -R hdfs:hdfs /var/lib/hadoop-hdfs/cache/hdfs/dfs/name
```

#### Formatting the HDFS storage

```sh
sudo -u hdfs hadoop namenode -format
```

### MapReduce v2 (YARN)

First configure the framework for map reduce:

`mapred-site.xml`:

```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
```

Then configure the staging directory in:

`yarn-site.xml`


```xml
<property>
  <name>yarn.app.mapreduce.am.staging-dir</name>
  <value>/user</value>
</property>
```

A staging directory is simply a directory, where temporary files were created by running jobs.


## Starting HDFS

There is a simple script for this purpose:

```bash
for x in `cd /etc/init.d ; ls hadoop-hdfs-*` ; do sudo service $x start ; done
```

which simply starts everything in `/etc/init.d` starting with `hadoop-hdfs-`


## Creating HDFS directories for Hadoop

```bash
sudo -u hdfs hadoop fs -mkdir /tmp
sudo -u hdfs hadoop fs -chmod -R 1777 /tmp
sudo -u hdfs hadoop fs -mkdir /user/history
sudo -u hdfs hadoop fs -chmod -R 1777 /user/history
sudo -u hdfs hadoop fs -chown yarn /user/history
sudo -u hdfs hadoop fs -mkdir /var/log/hadoop-yarn
sudo -u hdfs hadoop fs -chown yarn:mapred /var/log/hadoop-yarn
```

Verify the success of the operation by running:

```bash
sudo -u hdfs hadoop fs -ls -R /
```

You should see something like this:


```bash
drwxrwxrwt   - hdfs hadoop          0 2012-04-19 14:31 /tmp
drwxr-xr-x   - hdfs hadoop          0 2012-05-31 10:26 /user
drwxrwxrwt   - yarn hadoop          0 2012-04-19 14:31 /user/history
drwxr-xr-x   - hdfs   hadoop        0 2012-05-31 15:31 /var
drwxr-xr-x   - hdfs   hadoop        0 2012-05-31 15:31 /var/log
drwxr-xr-x   - yarn   mapred            0 2012-05-31 15:31 /var/log/hadoop-yarn
```

## Creating HDFS directories for each MapReduce user

```bash
sudo -u hdfs hadoop fs -mkdir /user/$USER
sudo -u hdfs hadoop fs -chown $USER /user/$USER
```


## Starting YARN and the MapReduce JobHistory Server

```bash
sudo service hadoop-yarn-resourcemanager start
sudo service hadoop-yarn-nodemanager start
sudo service hadoop-mapreduce-historyserver start
```

## Setting HADOOP_MAPRED_HOME
For each user who will be submitting MapReduce jobs using MapReduce v2 (YARN), or running Pig, Hive, or Sqoop in a YARN installation, set the HADOOP_MAPRED_HOME environment variable as follows:

```bash
export HADOOP_MAPRED_HOME=/usr/lib/hadoop-mapreduce
```


## Verifying the Installation


Hadoop ships with some examples:

```
fkasper@hadoop:~$ hadoop jar /usr/lib/hadoop-0.20-mapreduce/hadoop-examples.jar
An example program must be given as the first argument.
Valid program names are:
  aggregatewordcount: An Aggregate based map/reduce program that counts the words in the input files.
  aggregatewordhist: An Aggregate based map/reduce program that computes the histogram of the words in the input files.
  dbcount: An example job that count the pageview counts from a database.
  grep: A map/reduce program that counts the matches of a regex in the input.
  join: A job that effects a join over sorted, equally partitioned datasets
  multifilewc: A job that counts words from several files.
  pentomino: A map/reduce tile laying program to find solutions to pentomino problems.
  pi: A map/reduce program that estimates Pi using monte-carlo method.
  randomtextwriter: A map/reduce program that writes 10GB of random textual data per node.
  randomwriter: A map/reduce program that writes 10GB of random data per node.
  secondarysort: An example defining a secondary sort to the reduce.
  sleep: A job that sleeps at each map and reduce task.
  sort: A map/reduce program that sorts the data written by the random writer.
  sudoku: A sudoku solver.
  teragen: Generate data for the terasort
  terasort: Run the terasort
  teravalidate: Checking results of terasort
  wordcount: A map/reduce program that counts the words in the input files.
```

Here I will focus on `PI` and `wordcount`:

#### PI

```bash
fkasper@hadoop:~$ hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar  pi 1 10
```

```bash
fkasper@hadoop:~$ hadoop jar /usr/lib/hadoop-0.20-mapreduce/hadoop-examples.jar  pi 10 1000000000
Number of Maps  = 10
Samples per Map = 1000000000
Wrote input for Map #0
Wrote input for Map #1
Wrote input for Map #2
Wrote input for Map #3
Wrote input for Map #4
Wrote input for Map #5
Wrote input for Map #6
Wrote input for Map #7
Wrote input for Map #8
Wrote input for Map #9
Starting Job
14/03/12 14:46:51 WARN conf.Configuration: session.id is deprecated. Instead, use dfs.metrics.session-id
14/03/12 14:46:51 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
14/03/12 14:46:51 WARN mapred.JobClient: Use GenericOptionsParser for parsing the arguments. Applications should implement Tool for the same.
14/03/12 14:46:51 INFO mapred.FileInputFormat: Total input paths to process : 10
14/03/12 14:46:52 INFO mapred.LocalJobRunner: OutputCommitter set in config null
14/03/12 14:46:52 INFO mapred.JobClient: Running job: job_local1391904775_0001
14/03/12 14:46:52 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapred.FileOutputCommitter
14/03/12 14:46:52 INFO mapred.LocalJobRunner: Waiting for map tasks
14/03/12 14:46:52 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000000_0
14/03/12 14:46:52 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:46:52 INFO util.ProcessTree: setsid exited with exit code 0
14/03/12 14:46:52 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@e946c0
14/03/12 14:46:52 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part0:0+118
14/03/12 14:46:52 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:46:52 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:46:52 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:46:52 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:46:52 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:46:52 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:46:53 INFO mapred.JobClient:  map 0% reduce 0%
14/03/12 14:46:58 INFO mapred.LocalJobRunner: Generated 347959000 samples.
14/03/12 14:46:59 INFO mapred.JobClient:  map 10% reduce 0%
14/03/12 14:47:01 INFO mapred.LocalJobRunner: Generated 494651000 samples.
14/03/12 14:47:04 INFO mapred.LocalJobRunner: Generated 636609000 samples.
14/03/12 14:47:07 INFO mapred.LocalJobRunner: Generated 778043000 samples.
14/03/12 14:47:10 INFO mapred.LocalJobRunner: Generated 930874000 samples.
14/03/12 14:47:11 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:47:11 INFO mapred.MapTask: Finished spill 0
14/03/12 14:47:11 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000000_0 is done. And is in the process of commiting
14/03/12 14:47:11 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:47:11 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000000_0' done.
14/03/12 14:47:11 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000000_0
14/03/12 14:47:11 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000001_0
14/03/12 14:47:11 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:47:11 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@4148b3b8
14/03/12 14:47:11 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part1:0+118
14/03/12 14:47:11 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:47:11 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:47:11 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:47:11 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:47:11 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:47:11 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:47:17 INFO mapred.LocalJobRunner: Generated 345511000 samples.
14/03/12 14:47:18 INFO mapred.JobClient:  map 20% reduce 0%
14/03/12 14:47:20 INFO mapred.LocalJobRunner: Generated 520921000 samples.
14/03/12 14:47:23 INFO mapred.LocalJobRunner: Generated 696292000 samples.
14/03/12 14:47:26 INFO mapred.LocalJobRunner: Generated 865117000 samples.
14/03/12 14:47:28 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:47:28 INFO mapred.MapTask: Finished spill 0
14/03/12 14:47:28 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000001_0 is done. And is in the process of commiting
14/03/12 14:47:28 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:47:28 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000001_0' done.
14/03/12 14:47:28 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000001_0
14/03/12 14:47:28 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000002_0
14/03/12 14:47:28 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:47:28 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@4f5237c8
14/03/12 14:47:28 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part2:0+118
14/03/12 14:47:28 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:47:28 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:47:28 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:47:28 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:47:28 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:47:28 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:47:34 INFO mapred.LocalJobRunner: Generated 333142000 samples.
14/03/12 14:47:35 INFO mapred.JobClient:  map 30% reduce 0%
14/03/12 14:47:37 INFO mapred.LocalJobRunner: Generated 472473000 samples.
14/03/12 14:47:40 INFO mapred.LocalJobRunner: Generated 609763000 samples.
14/03/12 14:47:43 INFO mapred.LocalJobRunner: Generated 809886000 samples.
14/03/12 14:47:46 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:47:46 INFO mapred.MapTask: Finished spill 0
14/03/12 14:47:46 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000002_0 is done. And is in the process of commiting
14/03/12 14:47:46 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:47:46 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000002_0' done.
14/03/12 14:47:46 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000002_0
14/03/12 14:47:46 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000003_0
14/03/12 14:47:46 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:47:46 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@5f18d1f9
14/03/12 14:47:46 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part3:0+118
14/03/12 14:47:46 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:47:46 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:47:46 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:47:46 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:47:46 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:47:46 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:47:52 INFO mapred.LocalJobRunner: Generated 396664000 samples.
14/03/12 14:47:53 INFO mapred.JobClient:  map 40% reduce 0%
14/03/12 14:47:55 INFO mapred.LocalJobRunner: Generated 596877000 samples.
14/03/12 14:47:58 INFO mapred.LocalJobRunner: Generated 796676000 samples.
14/03/12 14:48:01 INFO mapred.LocalJobRunner: Generated 997069000 samples.
14/03/12 14:48:01 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:48:01 INFO mapred.MapTask: Finished spill 0
14/03/12 14:48:01 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000003_0 is done. And is in the process of commiting
14/03/12 14:48:01 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:48:01 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000003_0' done.
14/03/12 14:48:01 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000003_0
14/03/12 14:48:01 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000004_0
14/03/12 14:48:01 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:48:01 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@7c4b258b
14/03/12 14:48:01 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part4:0+118
14/03/12 14:48:01 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:48:01 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:48:01 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:48:01 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:48:01 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:48:01 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:48:07 INFO mapred.LocalJobRunner: Generated 390446000 samples.
14/03/12 14:48:08 INFO mapred.JobClient:  map 50% reduce 0%
14/03/12 14:48:10 INFO mapred.LocalJobRunner: Generated 589229000 samples.
14/03/12 14:48:13 INFO mapred.LocalJobRunner: Generated 788903000 samples.
14/03/12 14:48:16 INFO mapred.LocalJobRunner: Generated 989134000 samples.
14/03/12 14:48:17 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:48:17 INFO mapred.MapTask: Finished spill 0
14/03/12 14:48:17 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000004_0 is done. And is in the process of commiting
14/03/12 14:48:17 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:48:17 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000004_0' done.
14/03/12 14:48:17 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000004_0
14/03/12 14:48:17 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000005_0
14/03/12 14:48:17 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:48:17 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@10e631f
14/03/12 14:48:17 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part5:0+118
14/03/12 14:48:17 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:48:17 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:48:17 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:48:17 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:48:17 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:48:17 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:48:23 INFO mapred.LocalJobRunner: Generated 397961000 samples.
14/03/12 14:48:23 INFO mapred.JobClient:  map 60% reduce 0%
14/03/12 14:48:26 INFO mapred.LocalJobRunner: Generated 597254000 samples.
14/03/12 14:48:29 INFO mapred.LocalJobRunner: Generated 796758000 samples.
14/03/12 14:48:32 INFO mapred.LocalJobRunner: Generated 996941000 samples.
14/03/12 14:48:32 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:48:32 INFO mapred.MapTask: Finished spill 0
14/03/12 14:48:32 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000005_0 is done. And is in the process of commiting
14/03/12 14:48:32 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:48:32 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000005_0' done.
14/03/12 14:48:32 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000005_0
14/03/12 14:48:32 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000006_0
14/03/12 14:48:32 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:48:32 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@83df40b
14/03/12 14:48:32 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part6:0+118
14/03/12 14:48:32 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:48:32 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:48:32 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:48:32 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:48:32 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:48:32 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:48:38 INFO mapred.LocalJobRunner: Generated 398104000 samples.
14/03/12 14:48:38 INFO mapred.JobClient:  map 70% reduce 0%
14/03/12 14:48:41 INFO mapred.LocalJobRunner: Generated 593376000 samples.
14/03/12 14:48:44 INFO mapred.LocalJobRunner: Generated 789114000 samples.
14/03/12 14:48:47 INFO mapred.LocalJobRunner: Generated 988222000 samples.
14/03/12 14:48:47 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:48:47 INFO mapred.MapTask: Finished spill 0
14/03/12 14:48:47 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000006_0 is done. And is in the process of commiting
14/03/12 14:48:47 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:48:47 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000006_0' done.
14/03/12 14:48:47 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000006_0
14/03/12 14:48:47 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000007_0
14/03/12 14:48:47 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:48:47 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@1ea92bad
14/03/12 14:48:47 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part7:0+118
14/03/12 14:48:47 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:48:47 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:48:47 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:48:47 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:48:47 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:48:47 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:48:53 INFO mapred.LocalJobRunner: Generated 327837000 samples.
14/03/12 14:48:54 INFO mapred.JobClient:  map 80% reduce 0%
14/03/12 14:48:56 INFO mapred.LocalJobRunner: Generated 498775000 samples.
14/03/12 14:48:59 INFO mapred.LocalJobRunner: Generated 698546000 samples.
14/03/12 14:49:02 INFO mapred.LocalJobRunner: Generated 898367000 samples.
14/03/12 14:49:03 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:49:03 INFO mapred.MapTask: Finished spill 0
14/03/12 14:49:03 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000007_0 is done. And is in the process of commiting
14/03/12 14:49:03 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:49:03 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000007_0' done.
14/03/12 14:49:03 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000007_0
14/03/12 14:49:03 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000008_0
14/03/12 14:49:03 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:49:03 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@13e49516
14/03/12 14:49:03 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part8:0+118
14/03/12 14:49:03 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:49:03 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:49:03 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:49:03 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:49:03 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:49:03 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:49:09 INFO mapred.LocalJobRunner: Generated 394737000 samples.
14/03/12 14:49:10 INFO mapred.JobClient:  map 90% reduce 0%
14/03/12 14:49:12 INFO mapred.LocalJobRunner: Generated 594852000 samples.
14/03/12 14:49:15 INFO mapred.LocalJobRunner: Generated 794850000 samples.
14/03/12 14:49:18 INFO mapred.LocalJobRunner: Generated 994484000 samples.
14/03/12 14:49:18 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:49:18 INFO mapred.MapTask: Finished spill 0
14/03/12 14:49:18 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000008_0 is done. And is in the process of commiting
14/03/12 14:49:18 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:49:18 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000008_0' done.
14/03/12 14:49:18 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000008_0
14/03/12 14:49:18 INFO mapred.LocalJobRunner: Starting task: attempt_local1391904775_0001_m_000009_0
14/03/12 14:49:18 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:49:18 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@51067c73
14/03/12 14:49:18 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/in/part9:0+118
14/03/12 14:49:18 WARN mapreduce.Counters: Counter name MAP_INPUT_BYTES is deprecated. Use FileInputFormatCounters as group name and  BYTES_READ as counter name instead
14/03/12 14:49:18 INFO mapred.MapTask: numReduceTasks: 1
14/03/12 14:49:18 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:49:18 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:49:18 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:49:18 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:49:24 INFO mapred.LocalJobRunner: Generated 398864000 samples.
14/03/12 14:49:25 INFO mapred.JobClient:  map 100% reduce 0%
14/03/12 14:49:27 INFO mapred.LocalJobRunner: Generated 599451000 samples.
14/03/12 14:49:30 INFO mapred.LocalJobRunner: Generated 799370000 samples.
14/03/12 14:49:33 INFO mapred.LocalJobRunner: Generated 999733000 samples.
14/03/12 14:49:33 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:49:33 INFO mapred.MapTask: Finished spill 0
14/03/12 14:49:33 INFO mapred.Task: Task:attempt_local1391904775_0001_m_000009_0 is done. And is in the process of commiting
14/03/12 14:49:33 INFO mapred.LocalJobRunner: Generated 1000000000 samples.
14/03/12 14:49:33 INFO mapred.Task: Task 'attempt_local1391904775_0001_m_000009_0' done.
14/03/12 14:49:33 INFO mapred.LocalJobRunner: Finishing task: attempt_local1391904775_0001_m_000009_0
14/03/12 14:49:33 INFO mapred.LocalJobRunner: Map task executor complete.
14/03/12 14:49:33 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:49:33 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@2154c4ea
14/03/12 14:49:33 INFO mapred.LocalJobRunner:
14/03/12 14:49:33 INFO mapred.Merger: Merging 10 sorted segments
14/03/12 14:49:33 INFO mapred.Merger: Down to the last merge-pass, with 10 segments left of total size: 240 bytes
14/03/12 14:49:33 INFO mapred.LocalJobRunner:
14/03/12 14:49:34 INFO mapred.Task: Task:attempt_local1391904775_0001_r_000000_0 is done. And is in the process of commiting
14/03/12 14:49:34 INFO mapred.LocalJobRunner:
14/03/12 14:49:34 INFO mapred.Task: Task attempt_local1391904775_0001_r_000000_0 is allowed to commit now
14/03/12 14:49:34 INFO mapred.FileOutputCommitter: Saved output of task 'attempt_local1391904775_0001_r_000000_0' to hdfs://localhost/user/fkasper/PiEstimator_TMP_3_141592654/out
14/03/12 14:49:34 INFO mapred.LocalJobRunner: reduce > reduce
14/03/12 14:49:34 INFO mapred.Task: Task 'attempt_local1391904775_0001_r_000000_0' done.
14/03/12 14:49:35 INFO mapred.JobClient:  map 100% reduce 100%
14/03/12 14:49:35 INFO mapred.JobClient: Job complete: job_local1391904775_0001
14/03/12 14:49:35 INFO mapred.JobClient: Counters: 26
14/03/12 14:49:35 INFO mapred.JobClient:   File System Counters
14/03/12 14:49:35 INFO mapred.JobClient:     FILE: Number of bytes read=1642617
14/03/12 14:49:35 INFO mapred.JobClient:     FILE: Number of bytes written=2652843
14/03/12 14:49:35 INFO mapred.JobClient:     FILE: Number of read operations=0
14/03/12 14:49:35 INFO mapred.JobClient:     FILE: Number of large read operations=0
14/03/12 14:49:35 INFO mapred.JobClient:     FILE: Number of write operations=0
14/03/12 14:49:35 INFO mapred.JobClient:     HDFS: Number of bytes read=7670
14/03/12 14:49:35 INFO mapred.JobClient:     HDFS: Number of bytes written=13195
14/03/12 14:49:35 INFO mapred.JobClient:     HDFS: Number of read operations=340
14/03/12 14:49:35 INFO mapred.JobClient:     HDFS: Number of large read operations=0
14/03/12 14:49:35 INFO mapred.JobClient:     HDFS: Number of write operations=135
14/03/12 14:49:35 INFO mapred.JobClient:   Map-Reduce Framework
14/03/12 14:49:35 INFO mapred.JobClient:     Map input records=10
14/03/12 14:49:35 INFO mapred.JobClient:     Map output records=20
14/03/12 14:49:35 INFO mapred.JobClient:     Map output bytes=180
14/03/12 14:49:35 INFO mapred.JobClient:     Input split bytes=1190
14/03/12 14:49:35 INFO mapred.JobClient:     Combine input records=0
14/03/12 14:49:35 INFO mapred.JobClient:     Combine output records=0
14/03/12 14:49:35 INFO mapred.JobClient:     Reduce input groups=2
14/03/12 14:49:35 INFO mapred.JobClient:     Reduce shuffle bytes=0
14/03/12 14:49:35 INFO mapred.JobClient:     Reduce input records=20
14/03/12 14:49:35 INFO mapred.JobClient:     Reduce output records=0
14/03/12 14:49:35 INFO mapred.JobClient:     Spilled Records=40
14/03/12 14:49:35 INFO mapred.JobClient:     CPU time spent (ms)=0
14/03/12 14:49:35 INFO mapred.JobClient:     Physical memory (bytes) snapshot=0
14/03/12 14:49:35 INFO mapred.JobClient:     Virtual memory (bytes) snapshot=0
14/03/12 14:49:35 INFO mapred.JobClient:     Total committed heap usage (bytes)=4433379328
14/03/12 14:49:35 INFO mapred.JobClient:   org.apache.hadoop.mapreduce.lib.input.FileInputFormatCounter
14/03/12 14:49:35 INFO mapred.JobClient:     BYTES_READ=240
Job Finished in 163.45 seconds
Estimated value of Pi is 3.14159266440000000000
```

#### Wordcount

```bash
echo "word count should be 5" > hdfs_testfile
hdfs dfs -put hdfs_testfile /user/$USER/hdfs_testfile
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar wordcount hdfs_testfile hdfs_testresult 
```

```bash
fkasper@hadoop:~$ hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar wordcount hdfs_testfile hdfs_testresult
14/03/12 14:42:13 WARN conf.Configuration: session.id is deprecated. Instead, use dfs.metrics.session-id
14/03/12 14:42:13 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
14/03/12 14:42:13 WARN mapred.JobClient: Use GenericOptionsParser for parsing the arguments. Applications should implement Tool for the same.
14/03/12 14:42:13 INFO input.FileInputFormat: Total input paths to process : 1
14/03/12 14:42:13 INFO mapred.LocalJobRunner: OutputCommitter set in config null
14/03/12 14:42:13 INFO mapred.JobClient: Running job: job_local730913191_0001
14/03/12 14:42:13 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
14/03/12 14:42:13 INFO mapred.LocalJobRunner: Waiting for map tasks
14/03/12 14:42:13 INFO mapred.LocalJobRunner: Starting task: attempt_local730913191_0001_m_000000_0
14/03/12 14:42:13 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:42:13 INFO util.ProcessTree: setsid exited with exit code 0
14/03/12 14:42:13 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@7d2394ac
14/03/12 14:42:13 INFO mapred.MapTask: Processing split: hdfs://localhost/user/fkasper/hdfs_testfile:0+23
14/03/12 14:42:13 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
14/03/12 14:42:13 INFO mapred.MapTask: io.sort.mb = 100
14/03/12 14:42:13 INFO mapred.MapTask: data buffer = 79691776/99614720
14/03/12 14:42:13 INFO mapred.MapTask: record buffer = 262144/327680
14/03/12 14:42:13 INFO mapred.LocalJobRunner:
14/03/12 14:42:13 INFO mapred.MapTask: Starting flush of map output
14/03/12 14:42:13 INFO mapred.MapTask: Finished spill 0
14/03/12 14:42:13 INFO mapred.Task: Task:attempt_local730913191_0001_m_000000_0 is done. And is in the process of commiting
14/03/12 14:42:13 INFO mapred.LocalJobRunner:
14/03/12 14:42:13 INFO mapred.Task: Task 'attempt_local730913191_0001_m_000000_0' done.
14/03/12 14:42:13 INFO mapred.LocalJobRunner: Finishing task: attempt_local730913191_0001_m_000000_0
14/03/12 14:42:13 INFO mapred.LocalJobRunner: Map task executor complete.
14/03/12 14:42:13 WARN mapreduce.Counters: Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
14/03/12 14:42:13 INFO mapred.Task:  Using ResourceCalculatorPlugin : org.apache.hadoop.util.LinuxResourceCalculatorPlugin@169fd6
14/03/12 14:42:13 INFO mapred.LocalJobRunner:
14/03/12 14:42:13 INFO mapred.Merger: Merging 1 sorted segments
14/03/12 14:42:13 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 55 bytes
14/03/12 14:42:13 INFO mapred.LocalJobRunner:
14/03/12 14:42:13 INFO mapred.Task: Task:attempt_local730913191_0001_r_000000_0 is done. And is in the process of commiting
14/03/12 14:42:13 INFO mapred.LocalJobRunner:
14/03/12 14:42:13 INFO mapred.Task: Task attempt_local730913191_0001_r_000000_0 is allowed to commit now
14/03/12 14:42:13 INFO output.FileOutputCommitter: Saved output of task 'attempt_local730913191_0001_r_000000_0' to hdfs_testresult
14/03/12 14:42:13 INFO mapred.LocalJobRunner: reduce > reduce
14/03/12 14:42:13 INFO mapred.Task: Task 'attempt_local730913191_0001_r_000000_0' done.
14/03/12 14:42:14 INFO mapred.JobClient:  map 100% reduce 100%
14/03/12 14:42:14 INFO mapred.JobClient: Job complete: job_local730913191_0001
14/03/12 14:42:14 INFO mapred.JobClient: Counters: 25
14/03/12 14:42:14 INFO mapred.JobClient:   File System Counters
14/03/12 14:42:14 INFO mapred.JobClient:     FILE: Number of bytes read=286069
14/03/12 14:42:14 INFO mapred.JobClient:     FILE: Number of bytes written=479080
14/03/12 14:42:14 INFO mapred.JobClient:     FILE: Number of read operations=0
14/03/12 14:42:14 INFO mapred.JobClient:     FILE: Number of large read operations=0
14/03/12 14:42:14 INFO mapred.JobClient:     FILE: Number of write operations=0
14/03/12 14:42:14 INFO mapred.JobClient:     HDFS: Number of bytes read=46
14/03/12 14:42:14 INFO mapred.JobClient:     HDFS: Number of bytes written=33
14/03/12 14:42:14 INFO mapred.JobClient:     HDFS: Number of read operations=9
14/03/12 14:42:14 INFO mapred.JobClient:     HDFS: Number of large read operations=0
14/03/12 14:42:14 INFO mapred.JobClient:     HDFS: Number of write operations=3
14/03/12 14:42:14 INFO mapred.JobClient:   Map-Reduce Framework
14/03/12 14:42:14 INFO mapred.JobClient:     Map input records=1
14/03/12 14:42:14 INFO mapred.JobClient:     Map output records=5
14/03/12 14:42:14 INFO mapred.JobClient:     Map output bytes=43
14/03/12 14:42:14 INFO mapred.JobClient:     Input split bytes=108
14/03/12 14:42:14 INFO mapred.JobClient:     Combine input records=5
14/03/12 14:42:14 INFO mapred.JobClient:     Combine output records=5
14/03/12 14:42:14 INFO mapred.JobClient:     Reduce input groups=5
14/03/12 14:42:14 INFO mapred.JobClient:     Reduce shuffle bytes=0
14/03/12 14:42:14 INFO mapred.JobClient:     Reduce input records=5
14/03/12 14:42:14 INFO mapred.JobClient:     Reduce output records=5
14/03/12 14:42:14 INFO mapred.JobClient:     Spilled Records=10
14/03/12 14:42:14 INFO mapred.JobClient:     CPU time spent (ms)=0
14/03/12 14:42:14 INFO mapred.JobClient:     Physical memory (bytes) snapshot=0
14/03/12 14:42:14 INFO mapred.JobClient:     Virtual memory (bytes) snapshot=0
14/03/12 14:42:14 INFO mapred.JobClient:     Total committed heap usage (bytes)=424673280
```



