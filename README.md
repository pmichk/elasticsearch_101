# Doctor Who episode database


This repository contains a Docker compose file to run Elasticsearch and Kibana in version 7.9.2 and a dataset. This dataset is compiled from Wikipedia articles with summary of Doctor Who episodes up until December 2017. It's meant to serve as an example to [an article about Elasticsearch basics](http://pawelmichalik.net/articles/elasticsearch_101/index.html). You're welcome to follow along and experiment with the queries, or just use it to test out Elasticsearch.

## Setup

### Docker

Before starting docker compse, you need to set `vm.max_map_count` to at least `262144` [as described in the documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_set_vm_max_map_count_to_at_least_262144). It is mentioned as required only in production, but this limit is important in general to ensure Elastic has enough memory to run.

Then start the docker containers for Elasticsearch and Kibana by using the attached docker-compose file:

```
docker-compose up -d
```

This is by no means a production setup, most importantly it has a single node cluster, which should never be used outside of development environment!

Keep in mind that Kibana may take a while to start up at first. You can check the logs to make sure there was no errors and the services are starting up:

```
docker-compose logs -f
```

After Kibana has started, it will be available at <http://localhost:5601>. After starting for the first time select "Default space". After that you can load example data or start fresh - you can choose both, you'll still be able to load the data from this dataset.

### Import data

Now you can import the data using the provided file called `episodes.json`. Elasticsearch uses HTTP API for management and data manipulation, which means that you don't need a specialised client to use it. You can use any HTTP client, `curl` will be used in the commands below.

The commands below will create an index called `doctorwho`, create a custom mapping as defined in `mapping.json` and then import the data.

```
# Response should be {"acknowledged":true,"shards_acknowledged":true}
curl -s -H "Content-Type: application/x-ndjson" -XPUT localhost:9200/doctorwho

# Response should be {"acknowledged":true}
curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/doctorwho/_mapping --data-binary "@mapping.json"

# Response should be {"took":however much time the import took,"errors":false,"items":[all the data you sent]}
curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/doctorwho/_bulk --data-binary "@episodes.json"
```
