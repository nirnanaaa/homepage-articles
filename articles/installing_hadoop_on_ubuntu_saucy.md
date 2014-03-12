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

[[http://www.ividata.com/wp-content/uploads/2013/09/01_Hadoop_full.jpg]]

In this tutorial I will describe the required steps for setting up a pseudo-distributed, single-node YARN Hadoop cluster.backed by the Hadoop Distributed File System (HDFS), running on Ubuntu Linux 13.10 codename Saucy.

This tutorial is based on [this](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/) tutorial which is a bit out of date.

Hadoop is a framework written in Java for running applications on large clusters of commodity hardware and incorporates features similar to those of MapReduce and GFS.


The main goal here is to get a simple Hadoop installation so you can play with `MapReduce` and other cool stuff.

This tutorial has been tested with the following versions:

* Ubuntu Linux 13.10 Saucy Salamander
* Cloudera Hadoop 4 (CDH4)



