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

## Chapter 6: Performance Tuning<a name="Chapter6"></a>

## Chapter 7: Aggregating Datasets<a name="Chapter7"></a>

## Chapter 8: Best Practices<a name="Chapter8"></a>

