# Elasticsearch 7 Quick Start Guide

## Table of contents:

1. [Chapter 1: Introduction to Elastic Stack](#Chapter1)
2. [Chapter 2: Installing Elasticsearch](#Chapter2)
3. [Chapter 3: Many as One - the Distributed Model](#Chapter3)
4. [Chapter 4: Prepping Your Data - Text Analysis and Mapping](#Chapter4)
5. [Chapter 5: Let's Do a Search!](#Chapter5)
6. [Chapter 6: Performance Tuning](#Chapter6)
7. [Chapter 7: Aggregating Datasets](#Chapter7)
8. [Chapter 8: Best Practices](#Chapter8)

## Chapter 1: Introduction to Elastic Stack<a name="Chapter1"></a>

### Introduction

Elasticsearch is an open source search engine based on Lucene, the Elastic Stack consists of four components— Elasticsearch, Kibana, Logstash, and
Beats (Beats and Logstash send data to Elasticsearch, where the data is stored and Kibana is the UI of Elastic Stack).

### Elasticsearch

Elasticsearch provides the searching and management functionality of a document-oriented database. Documents are stored in JSON form, and queried with
a special DSL. It uses an HTTP interface, and REST APIs are used to index, search, retrieve, delete, or update the database. A relational database
system is a cluster of databases in which each database is called an index. The tables in the index are named type, each row is a document, and each
column is a field. The process of defining how a document and its fields are stored and indexed is called mapping. A query DSL is a SQL query that
requests information from a database. A cluster is a collection of servers that contain the entirety of the data. The default name for the cluster is
Elasticsearch.

### Kibana

Kibana is an open source interactive visualization and analytics plugin used by Elastic.

### Logstash

Logstash is an open source pipeline that collects data from multiple sources, processes them, and forwards events and log messages along with the data
to a stash, in this case to Elasticsearch. The operating procedure starts with an input plugin that collects the data, which is then processed using
filters that modify and annotate the event data. An input plugin is a component that allows a specified source of events to be accessed by Logstash. A
filter plugin then processes the event data, and this is often dependent on the characteristics of the event. An output plugin then sends the data to
the destination specified.

### Beats

Beats are basically lightweight data shippers that are designed for a very specific purpose. Some of the beat types are listed below

    * Filebeat: Filebeat is a lightweight data shipper that can be installed on different servers to read file data. Filebeat monitors the log 
    files that we specify in the configuration, collects the data from there in an incremental way, and then forwards them to Logstash or directly 
    into Elasticsearch for indexing
    * Metricbeat: Metricbeat is again a lightweight data shipper that can be installed on any server to fetch system metrics. Metricbeat ships the 
    collected system metrics data to Elasticsearch or Logstash for analysis
    * Auditbeat: We can install and configure Auditbeat on any server to audit the activities of users and processes. Auditbeat is helpful because 
    it detects changes to critical files, such as different configuration files and binaries
    * Winlogbeat: To ship Windows event logs from multiple machines (Hardware, security, system or application events)
    * Heartbeat: To monitor a server's uptime and check the status of different services and tells us whether they are available (similar to ping)

### Use cases:

Some of the elasticsearch use cases includes:

    * System monitoring: Elastic Stack can help us to avoid such situations where the system can impact application performance
    * Log management: Log analysis and visualization
    * Application Performance monitoring: To monitor applications for performance and availability
    * Data Visualization: Main feature of Kibana, we can create visualizations by creating index patterns in Kibana 

## Chapter 2: Installing Elasticsearch<a name="Chapter2"></a>

### Installing Elasticsearch on Linux

Follow these steps to install Elasticsearch on Linux:

The first step is to download the Elasticsearch7.1.1 Linux TAR file,which we can do using the following command:
`curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz`
Then extract it using the following command ` tar -xvf elasticsearch-7.1.1-linux-x86_64.tar.gz`. To start it go to _elasticsearch-7.1.1/bin_ and
execute `./elasticsearch`. There are other options of installation like the ones for debian, rpm package, SysV, systemd and others, check the
documentation for more information.

### Installing Elasticsearch on macOS

On Mac, brew is utilised to install elasticsearch, first execute `brew tap elastic/tap`, and then you can execute
`brew install elastic/tap/elasticsearch-full`.

### Checking whether Elasticsearch is running

You can check the version installed by doing `curl -X GET "localhost:9200/"`.

## Chapter 3: Many as One - the Distributed Model<a name="Chapter3"></a>

Every single server of Elasticsearch is considered a node, and multiple nodes form a cluster. Elasticsearch uses clusters to provide scalability and
redundancy. If a search is performed on a cluster, all nodes within it will be included in the search. Cluster APIs are cluster-level APIs that
operate on a set of nodes in a specific cluster.

### Handling multiple indices

APIs can be applied to multiple indices using a list of indices, a wildcard index, or all of the indices, by using the _\_all_ notation.


## Chapter 4: Prepping Your Data - Text Analysis and Mapping<a name="Chapter4"></a>

## Chapter 5: Let's Do a Search!<a name="Chapter5"></a>

## Chapter 6: Performance Tuning<a name="Chapter6"></a>

## Chapter 7: Aggregating Datasets<a name="Chapter7"></a>

## Chapter 8: Best Practices<a name="Chapter8"></a>

