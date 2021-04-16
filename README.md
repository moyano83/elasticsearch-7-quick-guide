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

Elasticsearch is very flexible for data analysis as it provides many built-in analyzers that we can pick, and we can even create our own analyzer.
Mapping can be dynamic or explicit; in dynamic mapping, Elasticsearch identifies the datatype for each field, which can be incorrect sometimes, while
in explicit mapping, we create the mapping before indexing the actual data.

### What is an analyzer?

Analysis as the process of converting text into tokens, which are then added to the inverted index and used for searching. An analyzer can be
specified in the mapping in the text field, as shown in the following query `PUT my_index` with the data

```json
{
  "mappings": {
    "_doc": {
      "properties": {
        "title": {
          "type": "text",
          "analyzer": "standard"
        }
      }
    }
  }
}
```

If there is no index analyzer specified, it will default to standard, which would remove the most frequent words and reduce words to the word stem
(i.e. "lines" becomes "line"). A similar process is performed on full-text queries. The text in the query string is changed into tokens similar to
those stored in the inverted index, then it performs a search. i.e. "My First program" will be transformed into \[first, program\] to ensure that the
query will match the terms exactly. The analyzer can be used to look for a specific field, such as the following:

    * An analyzer that is specified in the query
    * The search_analyzer mapping parameters or analyzer
    * An analyzer in the default_search or default index setting The standard analyzer

### Anatomy of an analyzer

An analyzer is a package that contains three building blocks. A user can create a custom analyzer by using these or other building blocks to create
the functionality needed. These blocks are:

    * Character filters: convert text into a stream of characters. They can transform the stream by adding, removing, or changing the format of 
      the characters. (i.e. convert '&' to 'and)
    * Tokenizers receive the stream of characters and break it down into tokens. The output will then be a stream of tokens as seen before
    * Token filters are converters of token streams. They can add, remove, or even change the tokens. An analyzer can contain anywhere from zero 
      to many token filters—and they are applied in order. (i.e. lowercasing letters)

#### How to use an analyzer

After applying the analyzer, we can find out how the analyzer is working by hitting the analyze API. Run `POST _analyze` with data

```json
{
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "asciifolding"
  ],
  "text": "Is this déja vu?"
}
```

##### The custom analyzer

By using a custom analyzer, we can tweak the tokenizer and filter and more as per our requirement, this depends on the type of data analyzed. We can
change the settings of an index with a `PUT <INDEX_NAME>`, an example of the data to pass is below:

```json

{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "_doc": {
      "properties": {
        "my_sentence": {
          "type": "text",
          "analyzer": "custom_analyzer"
        }
      }
    }
  }
}
```

##### The standard analyzer

Is the default analyzer if none is selected. Tokenizes the grammar based on the _Unicode Text Segmentation_. The standard analyzer accepts the
following parameters:

    * max_token_length: Takes the token length and breaks it into intervals if it exceeds 255
    * stopwords: Pre-defined stop list. Default to _none_. We can pass _english_ to handle English stop words or pass a list of stop words
    * stopwords_path: Shows the path to the stop words file

The standard analyzer consists of the standard tokenizer, a lowercase token filter and a stop token filter (disabled by default). To customize the
standard analyzer, we must recreate it as a custom analyzer.

##### The simple analyzer

A simple analyzer breaks the text into intervals whenever it encounters a non-letter term. Not configurable, only contains a lowercase tokenizer. We
can add the filters that we want to use with `PUT <INDEX_NAME>` and

```json
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_simple": {
          "tokenizer": "lowercase",
          "filter": [
            "add filter here"
          ]
        }
      }
    }
  }
}
```

##### The whitespace analyzer

A whitespace analyzer breaks the text into terms when it encounters whitespace characters. This analyzer cannot be configured. To alter it, users must
create a custom analyzer that mimics the whitespace analyzer, and then add filters to it.

##### The stop analyzer

The stop analyzer is similar to the simple analyzer and adds support to remove stop words. By default, it will use _english_ stop words. the stop
analyzer can be configured using the _stopwords_ and _stopwords_path_ parameters seen before. The stop analyzer consists of a lowercase tokenizer as
well as the stop token filter and it can be configured to a custom analyzer as well.

##### The keyword analyzer

A keyword analyzer returns the entire string as a token. It cannot be configured, and consists of the keyword tokenizer. To customize the analyzer, it
must be recreated and modified by adding token filters. i.e `PUT <INDEX_NAME>` with data:

```json

{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_keyword": {
          "tokenizer": "keyword",
          "filter": [
            "add filter here"
          ]
        }
      }
    }
  }
}
```

##### The pattern analyzer

A pattern analyzer splits the text into terms using a regular expression. The pattern analyzer can be configured using the following:

    * pattern: defaults to \W+.
    * flags: represents Java flags
    * lowercase: defaults all words to lowercase (the default is true)
    * stopwords: is a pre-defined stop words list such as _english_ or an array of lists
    * stopwords_path: points to the location of the file containing the stop words

The pattern analyzer consists of a pattern tokenizer, the lowercase token filter, and a stop token filter. A custom analyzer can be built by adding
token filters. i.e `PUT <INDEX_NAME>` with data:

```json
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "split_on_non_word": {
          "type": "pattern",
          "pattern": "\\W+"
        }
      },
      "analyzer": {
        "rebuilt_pattern": {
          "tokenizer": "split_on_non_word",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  }
}
```

##### The language analyzer

Language analyzers can be set to analyze the text of a specific language. They support custom stopwords, and the stem_exclusion parameter allows the
user to specify a certain array of lowercase words that are not to be stemmed.

##### The fingerprint analyzer

The fingerprint analyzers implement a fingerprinting algorithm that allows clustering. The fingerprint analyzer uses the following parameters:

    * separator: concatenates the terms using a character (defaults to space)
    * max_output_size: declares the maximum token size (defaults to 255)
    * stopwords: is a pre-defined stop words list such as _english_ or an array of lists
    * stopwords_path: points to the location of the file containing the stop words

To customize the fingerprint analyzer, the user needs to re-create it and add token filters to it.

#### Normalizers

Normalizers are a type of analyzer, but instead of producing multiple tokens, they only produce one. They do not contain tokenizers and accept only
some character filters and token filters. The filters that can be used are _asciifolding_, _cjk_width_, _decimal_digit_, _elision_, _lowercase_, and _
uppercase_, as well as some language filters. As with other analyzers, a custom normalization can be created.

#### Tokenizers

A tokenizer receives a stream of characters from a string and, whenever it encounters whitespace, it outputs the characters as individual words known
as tokens. A tokenizer is also in charge of keeping track of the order of each term and the start and end of character offsets.

##### The standard tokenizer

The standard tokenizer uses Unicode Text Segmentation to divide the text. It accepts the _\max\_token\_length_ parameter, which will split the token
if it exceeds the specified length (defaults to 255).

##### The letter tokenizer

The letter tokenizer breaks the text into individual words whenever it meets a character that is not a letter.

##### The lowercase tokenizer

The lowercase tokenizer, similar to the letter tokenizer, breaks the text into individual words whenever it finds a non-letter character, but it also
turns all terms into lowercase.

##### The whitespace tokenizer

The whitespace tokenizer breaks the text into individual words whenever whitespace is encountered. It accepts the _\max\_token\_length_ parameter,
which will split the token if it exceeds the specified length (defaults to 255).

##### The keyword tokenizer

The keyword tokenizer outputs the text as a single term. This tokenizer accepts the buffer_size parameter, which represents the number of characters
read in a single pass (defaults to 255).

##### The pattern tokenizer

The pattern tokenizer uses a regular expression to divide the text or capture the matching text as terms. The pattern tokenizer can be configured
using the following parameters:

    * pattern: defaults to \W+
    * flags: represents Java flags
    * group: captures the group as a token (by default, it splits at -1)

##### The simple pattern tokenizer

The simple_pattern tokenizer uses a regular expression to divide the text or capture the matching text as terms, similar to the pattern tokenizer, but
here, it only uses a pattern. It does not accept split patterns and is therefore generally faster than the pattern tokenizer. It accepts the pattern
parameter, which defaults the text to an empty string.

#### Token filters

Token filters receive a stream of tokens from a tokenizer and have the ability to add, modify, or delete tokens. Important token filters include:

    * lowercase: lowercases tokens 
    * uppercase: uppercases tokens 
    * stop: Removes the stop words from token streams
    * reverse: Reverses each token
    * elision: Removes elisions. i.e. 'l'avion' will be tokenized as 'avion'
    * truncate: Cuts the tokens into specific lengths (defaults to 10)
    * unique: Indexes the unique tokens during analysis
    * remove_duplicates: Removes the tokens that are identical in the same position

These can be configured with the following parameters:

    * stopwords: is a pre-defined stop words list such as _english_ or an array of lists
    * stopwords_path: points to the location of the file containing the stop words
    * ignore_case: set to true, it will change all words into lowercase. By default, this is set to false
    * remove_trailing: set to false, it will ignore the last term of a search if it is a stop word

#### Character filters

Character filters process the stream of characters by adding, modifying, or removing characters before they are passed to the tokenizer.

##### The HTML strip character filter

The html_strip character filter removes the HTML elements and decodes the HTML entities. For example:

```json
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer_name": {
          "tokenizer": "keyword",
          "char_filter": [
            "my_char_filter_name"
          ]
        }
      },
      "char_filter": {
        "my_char_filter_name": {
          "type": "html_strip",
          "escaped_tags": [
            "b"
          ]
        }
      }
    }
  }
}
```

With the above configuration, the text "\<p>I'm so \<b>happy\</b>!\</p>" will result in the term \[ \n I'm so happy! \n \].

##### The mapping character filter

The mapping character filter accepts a map of keys and values that are then used to replace certain characters in a string with their associated keys.
For example:

```json
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer_name": {
          "tokenizer": "keyword",
          "char_filter": [
            "my_char_filter_name"
          ]
        }
      },
      "char_filter": {
        "my_char_filter_name": {
          "type": "mapping",
          "mappings": [
            "٢ => 0",
            "٥ => 1"
          ]
        }
      }
    }
  }
}
```

With the above, a text such as "The binary text is ٢٥٢٥" will be converted in \[ The binary text is 0101 \]

##### The pattern replace character filter

The pattern_replace character filter searches with a regular expression and replaces the matched characters with the specified replacement string.
i.e:

```json
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer_name": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": {
            "my_char_filter": {
              "type": "pattern_replace",
              "pattern": ",(\d+)-(?=\d)",
              "replacement": "$1_"
            },
            "filter": [
              "lowercase",
              "asciifolding"
            ]
          }
        }
      }
    }
  }
}
```

With the above, if we pass "My phone number is 102-456-789", `[ my, phone, number, is, 102_456_789 `] will be returned.

### Mapping

Elasticsearch mapping is used to define the document structure with available fields and their datatypes. In Elasticsearch mapping, we have two types
of fields, user-defined fields or properties and meta fields. User-defined fields can be any field that we provide for mapping or indexing, while meta
fields are those fields that help us to identify associated document metadata, for example, _\_index_, _\_id_, and _\_source_.

#### Datatypes

Each field in an Elasticsearch document can have a datatype, which can be categorized into:

    * Simple datatype: Represent text, numbers, Boolean values, or dates. We use text, long, double, boolean, date, and so on to represent these types
    * Complex datatype: Represent nested data, which is hierarchical. We use object or nested to represent these types of values
    * Specialized datatype: Represent geo datatypes and such as geo_shape and geo_point

#### Multi-field mapping

Type of field mapping where we can map a field with different types. In the same way, we can index a string field with different types of analyzers
for different use cases. i.e. mapping the _ip\_address_ field with text as well as the keyword type:

```json
{
  "index_name": {
    "mappings": {
      "_doc": {
        "ip_address": {
          "full_name": "ip_address",
          "mapping": {
            "ip_address": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

#### Dynamic mapping

In dynamic mapping, we need not define the datatypes before using it. As soon as we index the document, the mapping is created automatically.

#### Explicit mapping

We can apply the explicit mapping using the create index API. i.e. `PUT <INDEX_NAME>` with data:

```json
   {
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "city": {
        "type": "text"
      }
    }
  }
}

```

## Chapter 5: Let's Do a Search!<a name="Chapter5"></a>

### Introduction to data search

Elasticsearch provides a search option for queries, strings, and even bodies of text. Most search APIs are multi-index. When performing a search, the
routing parameter is used to point to the location of the shards in the indices that are to be searched. This search is known as a round-robin between
replicas, where each replica is picked in the same rational order. As an alternative to the round-robin search replicas, an adaptive replica selection
can be made. This will look for the best copy that the node can send the request to. There are a few criteria based on this method:

    * The response time of the previous interactions between the coordinating node and the node that contains the copy of the data
    * The extra time that the previous requests took to execute on the node containing the data
    * The queue size of the requested data on the coordinating node

To activate this adaptive replica selection, the following setting needs to be changed to true _cluster.routing.use_adaptive_replica_selection_. In
Elasticsearch, individual searches have timeout criteria. Elasticsearch will apply a global timeout to all the clusters that do not mention a timeout
in their settings. To change these timeout settings, simply modify the _search.default_search_timeout_ parameter and set a specific value. A value of
-1 will set the global search timeout to no timeout at all. To cancel a search, the _\_cancel_ parameter is needed, along with the ID of the search
task. Since searches on shards can be canceled, a large segment may lead to a delay in the cancellation. To minimize the cancel responsiveness, set
_search.low_level_cancellation_ to true. The searches that are performed on Elasticsearch may need access to various shards simultaneously, and this
will affect the CPU and memory. Users can limit the number of shards that can be accessed at once through the _max_concurrent_shard_requests_ setting.
This is a hard limit that can be set to any value. A soft limit can also be created using _action.search.shard_count.limit_.

### Search API

The search API performs a search using a query string, a parameter, or a request body as the search criteria, and then it returns exact matches. i.e

    * `GET facebook/_search?q=user:flower` search all of the documents in a specific index using the cactus_flower user
    * `GET _all/_search?q=tag:wow` search for a tag wow in all indices

#### URI search

A URI search can be executed using a URI in the request parameters `GET <INDEX_NAME>>/_search?q=user:flower`. Parameters available in this search:

    * q: The query string.
    * df: The default field to be used when nothing is defined.
    * analyzer: The analyzer name that was used to execute the search
    * analyze_wildcard: Checks whether a wildcard and prefix queries are to be analyzed. Defaults to false
    * batched_reduced_size: The number of shards that will be reduced on the coordinating node. This is used to void memory overhead per request
    * default_operator: The default operator that is to be used. The default is OR and it can be set to AND
    * lenient: Ignores format-based failures. Defaults to false
    * explain: Contains information on how the scoring of the hit was computed. It does this for every hit
    * stored_fields: Retrieves the selective store fields of the documents that were retrieved with each hit. When no value is indicated, it will 
      show no fields in the returned hits.
    * sort: Performs sorting and is written as fieldName or fieldName:asc/ or fieldName:dec. It can be an actual field name or a special _score name. 
    * track_scores: Defaults to true and tracks the scores when sorting. It will return this with the hits.
    * track_total_hits: Defaults to false and tracks the total number of hits that match the query.
    * timeout: A search timeout that's used to execute the search within a specified time. Defaults to no timeout
    * terminate_after: Will retrieve whether the query execution was terminated early. Defaults to no terminate_after
    * from: The number of the index that will start the return values. Defaults to 0
    * size: The number of hits to be returned. Defaults to 10
    * search_type: Determines which type of search operation is to be executed. It can be dfs_query_then_fetch or query_then_fetch
    * allow_partial_search_results: If false, any partial results will mean an overall failure. Controlled with search.default_allow_partial_results

#### Request body search

Search requests are executed with a search DSL. i.e run `GET <INDEX_NAME>/_search` with data:

```json
{
  "query": {
    "term": {
      "user": "flower"
    }
  }
}
```

The parameters for the body search are as follows:

    * timeout: If set, this means that searches must be executed within a specified time. Defaults to no timeout.
    * from: The number of the index from which to start the return values. Defaults to 0.
    * size: The number of hits to be returned. Defaults to 10.
    * search_type: Determines which type of search operation is to be executed. It can be dfs_query_then_fetch or query_then_fetch.
    * request_cache: If set to true, it enables the caching of search requests with aggregations and suggestions (known as size = 0). 
    * allow_partial_search_results: If the request returns partial results, this will result in an overall failure if this parameter is set to false. 
      This can be controlled with the search.default_allow_partial_results setting. 
    * terminate_after: This will specify whether the query execution was terminated early. Defaults to no terminate_after.
    * batched_reduced_size: The number of shards that will be reduced on the coordinating node. This is used to void memory overhead request

The search_type, allow_partial_search_results, and request_cache parameters are set as query string parameters. The others are passed within the body.
Searches within the body are allowed with HTTP, GET, and POST.

##### Query

The _query_ element defines a query in the request body using Query DSL

##### From/size

The search query can be modified to return only a specific set of indices or a number of hits using the _from_ and _size parameters, the _from_
_size_ and the overall size cannot be greater than the _index.max_result_window_.

##### Sort

The _sort_ element allows us to sort the results on specific fields. Each sort is reversible. To sort by score, the _\_score_ field name is used,
whereas to sort by an index, the _\_doc_ field name is used. The endpoint is `GET /<INDEX_NAME>/_search` and syntax is:

```json
{
  "sort": [
    {
      "field1": {
        "order": "asc",
        "mode": "avg"
      },
      "field2": {
        "name": "desc"
      }
    },
    "_score"
  ],
  "query": {
    "term": {
      "user": "flower"
    }
  }
}
```

The _\_order_ term can be _asc_ or _desc_ (_\_score_ defaults to desc, whereas everything else defaults to asc). The mode option controls the array
value to be chosen for the sorting document:

     * min: Chooses the lowest value
     * max: Chooses the highest value
     * sum: Chooses the sum of all the values as a sort value. This is only for number-based array fields
     * avg: Chooses the average of all the values as a sort value. This is only for number-based array fields
     * median: Chooses the median of all the values as a sort value. This is only for number-based array fields

Elasticsearch supports sorting in nested objects as well, with the following properties:

    * path: Defines the path to the nested object
    * filter: A filter that the inner objects need to pass through before being considered
    * max_children: The maximum number of children per root document (unlimited by default)
    * nested: Applies to another nested path within the nested object

##### Source filtering

This controls how the _source field is returned with each hit, defaults to true (It also accepts wildcards):

```json
{
  "_source": "obj.*",
  "query": {
    "term": {
      "user": "cactus_flower"
    }
  }
}
```

To have more specific search control, the pattern's _includes_ and _excludes_ (`{"source":{"includes":[...], "excludes":[...]}`).

##### Fields

The stored_fields parameter is off by default and is not recommended. Source filtering is recommended instead.

##### Script fields

Script fields return a script evaluation for each hit. They are applied to _fields_ that are not stored and allow the evaluated value of the script to
be returned. They can also access the document's source and extract specific elements with params['_source']. The following example shows _source
pointing to the JSON-like model:

```json
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "test1": {
      "script": "params['_source']['message']"
    }
  }
}
```

##### Doc value fields

This field returns the doc value representation of a field for each hit. They are applied to fields that have doc values, regardless of whether they
are stored:

```json
{
  "query": {
    "match_all": {}
  },
  "docvalue_fields": [
    {
      "field": "*_date_field",
      "format": "epoch_millis"
    }
  ]
}
```

In the above example "*_date_field"  matches all fields that end with field, and "format" is the format to be applied to the fields. Date fields and
numeric fields are some of the only fields that accept custom formats.

##### Post filter

The post filter is applied to the hits that are returned after a search request.

```json
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "brand": "milka"
          }
        }
      ]
    }
  },
  "aggs": {
    "tastes": {
      "terms": {
        "field": "salty"
      }
    }
  }
}
```

The above query, retrieves all chocolates from milka that are marked as salty.

##### Highlighting

Highlighters permit the highlighting of snippets from one or more fields. The highlighters are specified in the highlighter type and they can be:

    * unified highlighter: uses the Lucene Unified highlighter and breaks the text into sentences by using the BM25 algorithm
    * plain highlighter: uses the standard Lucene highlighter and returns the reflect query logic
    * fvh highlighter: uses the Lucene fvh. It can be customized using boundary_scanner and requires setting term_vector to with_position_offsets 
      in order to increase the size of the index. It can also combine multiple fields and can be used to assign weights to different positions to 
      modify the order of importance of a query.

The parameters that are used by highlight fields are:

    * boundary_chars: A string that contains the boundary characters (defaults to .,!?\t\n)
    * boundary_max_scan: The range of the boundary characters (defaults to 20) 
    * boundary_scanner_locale: This controls which locale is used for searches 
    * encoder: Specifies whether the snippet should be default or html (HTML encoded)
    * fields: Specifies the fields that are to be retrieved for highlighting
    * force_source: Highlights the source of the field (defaults to false)
    * fragmenter: Indicates how the text is to be broken in snippets. It can be simple or span.
    * fragment_offset: This controls which end of the range to start highlighting. Only available in the fvh highlighter.
    * fragment_size: Specifies the size of the highlighted fragment in characters (defaults to 100)
    * highlight_query: Highlights the matches for a different query. 
    * matched_fields: Combines multiple fields and highlights a single field. 
    * no_match_size: Amount of text to be returned from the beginning of the field in the case where there are no matching fragments (defaults to 0) 
    * number_of_fragments: Specifies the maximum number of fragments to be returned (defaults to 5)
    * order: Sorts the highlighted hits. The score value will sort according to the score of the hit (defaults to 5)
    * phrase_limit: The number of matching phrases within a document to be considered. To prevent fvh from analyzing too many phrases affecting memory
    * pre_tags: Defines the HTML tags. Used in conjunction with post_tags. By default, the text is wrapped with <em> and </em> tags.
    * post_tags: Defines the HTML tags. Used in conjunction with pre_tags. By default, the text is wrapped with <em> and </em> tags.
    * require_field_match: Highlights only the fields that match the query. If set to false, it will highlight all fields (defaults to true)
    * tags_schema: Sets to the styled schema to define pre_tags and post_tags.
    * type: Indicates the type of highlighter to be used, that is, unified, plain, or fvh.

##### Rescoring

Rescoring helps with the precision of requests by reordering the search hits. It is used in the query and post_filter phases:

```json
{
  "query": {
    "match": {
      "message": {
        "operator": "or",
        "query": "the quick brown"
      }
    }
  },
  "rescore": {
    "window_size": 50,
    "query": {
      "rescore_query": {
        "match_phrase": {
          "message": {
            "query": "the quick brown",
            "slop": 2
          }
        }
      },
      "query_weight": 0.7,
      "rescore_query_weight": 1.2
    }
  }
}
```

The parameters are as follows:

    * total: Sum the original score and the rescore query score 
    * multiply: Multiply the original score and the rescore query score 
    * avg: The average of the original score and the rescore query score 
    * max: The maximum of the original score and the rescore query score 
    * min: The minimum of the original score and the rescore query score

##### Search type

Searches can be executed using different paths. The distribution search operation is scattered to all the relevant shards, and then the results are
retrieved. It will sort through the results and check for correctability. Since each shard is independent, the searches of term frequencies happen
over all the shards. To provide an accurate ranking, the term frequencies of the shards need to be used to calculate a global term frequency, which
will then be applied to the query in each shard search. Elasticsearch allows control over the type of search to be performed on a per search request
basis.

The settings are configured in the search_type parameter in the query string. The first setting that can be used is the _query_then_fetch_
value. This will send the query to all the shards that are relevant. Each shard performs the search and returns the necessary information to the
coordinating node, which will merge and resort the results from all the shards it contains. The second value that can be used is _
dfs_query_then_fetch_. This is very similar to _query_then_fetch_, except that it provides more accurate scoring.

##### Scroll

The scroll API retrieves a large number of results. The scroll field is used to scroll through large amounts of data.

##### Preference

This controls the preference of the shard copies that are used to execute the search. It can be set to any of the following:

    * _only_local: Only local shards that have been allocated to the local node are used to execute the operation
    * _local: Only shards that are from the local node are used to execute the operation, the process falls back to other shards when not possible
    * _prefer_nodes: abc, xyz: Executes the operation on those nodes if possible
    * _shards: 2,3: Restricts operation to the specified shards
    * _only_nodes: abc*, x*yz,...:Restricts operation to the specified nodes

##### Explanation

This will print an explanation of the query regarding how each hit was computed, accepts true/false.

##### Version

To return the version number for each result hit, along with the results, accepts true/false.

##### min_score

_\_min_score_ excludes documents that do not meet the minimum score specified.

##### Named queries

Using _\_name_, we can define a name for each filter and query.

##### Inner hits

Used for nested and parent-join documents that matches. In the case of parent/child, the parent documents are retrieved on matches in the child
documents, while child documents are returned in the match of parent documents. The inner hits feature is used to determine exactly which inner nested
object or parent/child document matches the search query. The query format is like `"<query>": { "inner_hits": { <inner_hits_options>}}`, and the
options are as follows:

    * from: The first hit to fetch inner_hits
    * size: The maximum number of hits per inner_hits
    * sort: What type of sorting should take place. Defaults to sorting by score
    * name: Defines the name for the inner hit definition

##### Field collapsing

This allows the search to collapse according to the field values

#### Search template

The search template API allows for the use of the _mustache language_ to prerender search actions before they are executed. The endpoint is `GET
_search/template` and data:

```json
{
  "source": {
    "query": {
      "match": {
        "{{my_field}}": "{{my_value}}"
      }
    },
    "size": "{{my_size}}"
  },
  "params": {
    "my_field": "message",
    "my_value": "some message",
    "my_size": 5
  }
}
```

In the preceding example, we are filling the existing templates with template parameters.

#### Multi search template

The multi search template allows for the execution of the search over multiple template requests in the same API. It does this by using the
_\_msearch/template_ endpoint. It is similar in format to the multi search API. The structure of the multi search template is:

header\n body\n header\n body\n

As seen in the preceding example, each newline character ends with \n. Each line may be preceded by \r. The header indicates which indices are to be
searched, that is, _search_type_, _preference_, and _routing_. i.e.

```text
{"index": "test"}
{"source": {"query": {"match": {"user" : "{{username}}" }}}, "params": {"username":"John"}}
{"source": {"query": {"{{query_type}}": {"name": "{{name}}" }}}, "params":{"query_type": "match_phrase_prefix", "name": "Smith"}}
{"index": "_all"}
{"id": "template_1", "params": {"query_string": "search for these words" }}
```

#### Search shards API

API that retrieves the indices and shards that the search request is executed on. Used to get feedback and determine issues, plan optimizations, or
determine shard preference. To specify a route, add the routing parameter with the path to the search shard API like in
`GET /<INDEX_NAME>/_search_shards?routing=foo,bar`. The search shard API uses the following parameters:

    * routing: A list of comma-separated values to determine the location of the shards to be searched during execution
    * preference: Gives preference to shard replicas (defaults to random) 
    * local: A Boolean value that determines whether to read the cluster locally

#### Suggesters

The suggest feature looks for similar terms based on the suggested text. _\_suggesters_ has been deprecated and _\_search_ is now used instead.

#### Multi search API

The multi search API allows for the execution of search requests within the same. The header indicates which indices are to be searched, that is,
_search_type_, _preference_, and _routing_.

#### Count API

The _\_count_ API executes a query and retrieves the number of matches within that query. The query can be defined using a query string as a parameter
or using Query DSL. It can be applied to multiple indices. The request parameters can be any of the following:

    * df: The default field to be used
    * analyzer: The name of the analyzer that's used to analyze the query string
    * default_operator: The default operator to be used can be OR (default) or AND
    * lenient: Ignores format-based failures (defaults to false)
    * analyze_wildcard: Indicates whether a wildcard and prefix queries are to be analyzed (defaults to false)
    * terminate_after: Will retrieve whether the query's execution was terminated early (defaults to no terminate_after)

#### Validate API

The validate API validates a potentially expensive query without executing it. It uses the parameters _df_, _analyzer_, _default\_operator_,
_lenient_ and _analyze_wildcard_ explained before. The endpoint is `GET <INDEX_NAME>/_validate/query?q=user:foo` and the resulting output is like:

```json
{
  "valid": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  }
}
```

#### Explain API

The explain API explains the score computation of a query and a document `GET /<INDEX_NAME>/_explain/<DOCUMENT_IT>` with data

```json
{
  "query": {
    "match": {
      "message": "elasticsearch"
    }
  }
}
```

Here, we have the following parameters

    * _source: Retrieves the source if set to true
    * stored_fields: Controls which stored fields are returned as part of the document being explained
    * routing: When routing is used during indexing, this will control the routing
    * parent: This is the same as routing
    * preference: Controls which shard the explain is executed on
    * source: The data of the request will be put in the query string
    * q: The query string

On top of those, we have the parameters _df_, _analyzer_, _default\_operator_, _lenient_ and _analyze_wildcard_ explained before.

#### Profile API

Profile is a debugging tool which adds detailed information about the execution of each component in a search request. It gives the user insight about
every step a search request performs and can assist in determining why certain requests are slow. Include it as a parameter within the body of the
_\_search_ request.

##### Profiling queries

The Profile API on queries exposes Lucene class names and concepts. This requires some knowledge of Lucene:

    * The query section contains detailed timing that's executed by Lucene on a shard
    * The breakdown component lists the timing statistics about the low-level execution
    * The collectors section shows the high-level execution details
    * The rewrite section performs optimizations. This is a very complex process

##### Profiling aggregations

The aggregations section contains detailed information about the timing of the aggregation tree that's executed by a shard.

##### Profiling considerations

When using the Profile API, a non-negligible overhead is added to the search execution, and this leads to a number of limitations:

    * Profile does not measure the search fetch phase
    * Profile does not measure the network overhead
    * It also doesn't account for the time spent in the queue, merging, or doing any additional work 
    * The statistics are not available for suggestions, highlighting, or dfs_query_then_fetch 
    * It is a highly experimental instrument and a part of Lucene that is used for diagnostics

#### Field capabilities API

This retrieves the capabilities of fields among multiple indices. To execute on all indices, use the query `GET _field_caps?fields=field_name` and to
execute on a specific index, use `GET <INDEX_NAME>/_field_caps?fields=field_name`. The parameters are as follows:

    * searchable: Whether the field is indexed for search on all indices 
    * aggregatable: Whether the field can be aggregated on all indices
    * indices: A list of indices where the field is of the same type 
    * non_searchable_indices: A list of indices where searches cannot be executed 
    * non_aggregatable_indices: A list of indices where the field is not aggregatable

## Chapter 6: Performance Tuning<a name="Chapter6"></a>

## Chapter 7: Aggregating Datasets<a name="Chapter7"></a>

## Chapter 8: Best Practices<a name="Chapter8"></a>

