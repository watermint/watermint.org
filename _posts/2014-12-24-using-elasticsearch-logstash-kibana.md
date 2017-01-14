---
layout: post
title: 'Using ELK(ElasticSearch 1.4.2 + Logstash 1.4.2 + Kibana 4 Beta3)'
tags:
 - elasticsearch
 - logstash
 - kibana
---

# Demonstration on ELK on Mac OS X (Yosemite)

## Download

* http://www.elasticsearch.org/overview/elkdownloads/

## Setup - ElasticSearch

```bash
$ unzip elasticsearch-1.4.2.zip
```

### Configure

Configure elasticsearch to allow connection from Kibana4. Add following two lines into $ELASTICSEARCH_HOME/conf/elasticsearch.yml 

```yaml
http.cors.enabled: true
http.cors.allow-origin: /.*/
```

### Startup

```bash
$ ./bin/elasticsearch
```

### Test

```bash
$ curl localhost:9200
```

## Logstash 1.4.2

```bash
$ unzip logstash-1.4.2.zip
```

You have to replace elasticsearch libraries inside logstash. 

* https://github.com/elasticsearch/kibana/issues/1629#issuecomment-64172892
* logstash contains old version of elasticsearch (1.1.1).
* Kibana4 requires elasticsearch 1.4.0 or higher.

```bash
$ cd logstash-1.4.2/vendor/jar
$ rm -fr elasticsearch-1.1.1
$ unzip $YOUR_DOWNLOAD_DIR/elasticsearch-1.4.2.zip
```

### Test configuration

Save file like below as test-logstash.conf

```
input {
  file {
    type => "apache"
    path => "/var/log/apache2/access_log"
  }
}

output {
  elasticsearch {
    host => localhost
  }
}
```

### Startup

```bash
$ ./bin/logstash -f test-logstash.conf
```

### Test

```bash
$ sudo apachectl start
$ curl localhost
```

## Kibana 4 Beta 3

```bash
$ unzip kibana-4.0.0-beta3.zip
```

### Startup

```bash
$ ./bin/kibana
```

### Test

Open http://localhost:5601 with your browser.
Configure index pattern with time-based events.

* Check "Index contains time-based events"
* Index name or pattern: logstash-*
* Choose Time-field name: `@timestamp`

![Kibana4.png](/images/2014-12-24-kibana.jpg)