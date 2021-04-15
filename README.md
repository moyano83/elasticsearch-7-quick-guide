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

Elasticsearch is an open source search engine based on Lucene, the Elastic Stack consists of four components: Elasticsearch, Kibana, Logstash, and
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

Some of the Elasticsearch use cases includes:

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

On Mac, brew is utilised to install Elasticsearch, first execute `brew tap elastic/tap`, and then you can execute
`brew install elastic/tap/elasticsearch-full`.

### Checking whether Elasticsearch is running

You can check the version installed by doing `curl -X GET "localhost:9200/"`.

## Chapter 3: Many as One - the Distributed Model<a name="Chapter3"></a>

Every single server of Elasticsearch is considered a node, and multiple nodes form a cluster. Elasticsearch uses clusters to provide scalability and
redundancy. If a search is performed on a cluster, all nodes within it will be included in the search. Cluster APIs are cluster-level APIs that
operate on a set of nodes in a specific cluster.

### Handling multiple indices

APIs can be applied to multiple indices using a list of indices, a wildcard index, or all of the indices, by using the _\_all_ notation. Elasticsearch
supports the following URL query string parameters for all multi-index APIs:

    * ignore_unavailable: boolean to include or not include nonexistent indices or closed indices
    * allow_no_indices: boolean to control whether the process will fail if a wildcard index expression leads to no indices
    * expand_wildcards: boolean to include open or closed indices

### Common options for the API response

We can pass url parameters to the query we are performing to modify the format of the response given by Elasticsearch. Some options include

    * pretty=true: to beautify the JSON response given
    * format=yaml: To return a Yaml file instead of a JSON
    * human=true: To transform the result into a more readable response (i.e. transform 3,600,000 milliseconds to 1 hour)
    * filter_path: To reduce the response by selecting only the fields we are interested in
    * flat_settings=true: To return the results in flat format

#### Cluster state and statistics

€### Cluster health status

To retrieve cluster health status information, we need to run `GET _cluster/health`. The results of this query outputs details of the cluster, such as
the name of the cluster, status, whether it has timed out, the number of nodes, the number of data nodes, the number of active primary shards, the
number of active shards, the number of initializing shards, the number of pending tasks...

#### Cluster state

To view information about the comprehensive state of the cluster, run `GET _cluster/state`. The cluster state query can be filtered using response
filters, like in `GET /_cluster/state/{metrics}/{indices}`. Some of the available metrics include _version_, _master_node_, _nodes_,
_routing_table_, _metadata_ or _blocks_.

#### Cluster stats

To retrieve information from the overall cluster run `GET _cluster/stats`, which can also be restricted like in
`GET /_cluster/stats/nodes/node_name_1,master:false`.

#### Cluster administration

A cluster can reroute the locations of shards to nodes. Get this information running `POST /_cluster/reroute` with the data
(This will reroute the 0 shard with the reroute_me index from node 1 to node 2 and create a replica in node 3):

```json
{
  "commands": [
    {
      "move": {
        "index": "reroute_me",
        "shard": 0,
        "from_node": "node1",
        "to_node": "node2"
      }
    },
    {
      "allocate_replica": {
        "index": "reroute_me",
        "shard": 1,
        "node": "node3"
      }
    }
  ]
}
``` 

The cluster settings can be reviewed with `GET /_cluster/settings`. The order of cluster precedence is as follows:

    1. Transient cluster settings
    2. Persistent cluster settings
    3. Settings in the elasticsearch.yml configuration file

### Node state and statistics

To retrieve information about a node, run `GET /_nodes`. Some flags can be put to narrow the results like in `GET /_nodes/{FLAG}`. FLAG can be one of:

    * _all: Retrieve information of all nodes
    * _local: Retrieve the local node information
    * _master: Retrieve the master node information
    * _name_*:  Retrieve information of the nodes with that name, or that include a wildcard
    * master:true or _master:false: Retrieve information of the nodes according to their role

#### Operating system information

To retrieve information about the operating system, add the os flag `GET _nodes/os`. This call returns information about the os name, version, number
of processors available for the virtual machine and number of processors that are actually being used in the thread pool size.

#### Process Information

To retrieve the process information about the running processes, use `GET _nodes/process`, which displays info about the PID or refresh interval.

#### Plugin information

To retrieve information about plugins and modules installed in the nodes, the following command is used `GET /_nodes/plugins`.

### Index APIs

An index is a place where data is stored, and it points to one or more physical shards, which commute between nodes to ensure balance in the cluster.
When creating an index, a set number of primary shards is fixed. We can create an index using the command `PUT index_name` and the following data:

```json
{
  "Settings": {
    "number_of_shards": 4,
    "number_of_replicas": 1
  },
  "mappings": {
    "_doc": {
      "properties": {
        "field_number_1": {
          "type": "text"
        }
      }
    }
  }
}
```

In the example above, we define the mappings of the document, capturing the field with the name "field_number_1". To delete an index,
run `DELETE /{index_name}[,{index_name2},{index_name3}...]`.

### Single-document APIs

#### Creating a document

The following will create a new document with id 1 `PUT test_index/_doc/1` with data `{"user": "user1","created_on": "2001-01-01"}`. The command
returns a JSON with information about the number of shard copies (field total), replication-related errors (field failed), and number of shards
successfully copied (field successful). The automatic indexing of documents is created with the _action.auto_create_index_ setting. This can be
updated with the command `PUT _cluster/settings` and the data:

```json
{
  "persistent": {
    "action.auto_create_index": "true"
  }
}
```

Unless specified otherwise, when an index operation is executed, an ID is automatically generated.

#### Viewing a document

To view a document, run `GET {INDEX_NAME}/_doc/{DOCUMENT_ID}`. Using the _source parameter, we can return the selected fields instead of all fields of
the document. The previous command will return something like this:

```json
{
  "_index": "test_index",
  "_type": "_doc",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "user": "user_name",
    "Post_date": "2001-01-01T10-10-10"
  }
}
```

#### Deleting a document

To delete a document, run `DELETE {INDEX_NAME}/_doc/{DOCUMENT_ID}`, which will return a JSON containing information about the number of successful
shard deletions, the result of the operations, and others.

#### Delete by query

We can also delete documents using the _\_delete\_by\_query_ API. The command is `POST {INDEX_NAME}/_delete_by_query` and an example of the data
passed is `{"query": {"match": {}}}`.

#### Updating a document

To use the update API, use the _update parameter containing the required script changes. The parameters that can be used are _index, _type, _id, _
version, _routing, and _now. An update using a partial document will add the partial document fields to an existing document:
`POST test_index/_doc/1/_update {"doc": {"user": "jorge"}}`.

### Multi-document APIs

Similar to single document APIs but affecting more than one document, for example the command `GET /test_index/_doc/_mget {"ids" : ["1", "2"]}`
will return documents 1 and 2.

## Chapter 4: Prepping Your Data - Text Analysis and Mapping<a name="Chapter4"></a>

## Chapter 5: Let's Do a Search!<a name="Chapter5"></a>

## Chapter 6: Performance Tuning<a name="Chapter6"></a>

## Chapter 7: Aggregating Datasets<a name="Chapter7"></a>

## Chapter 8: Best Practices<a name="Chapter8"></a>

