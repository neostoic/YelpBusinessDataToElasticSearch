# Yelp Business Data to ElasticSearch
=========
Mapping of Yelp's Business Data to Elastic Search with Geo-sharding. 

[![Stories in Progress](https://badge.waffle.io/waffleio/waffle.io.svg?label=waffle%3Ain%20progress&title=In%20Progress)](http://waffle.io/ashwintumma23/YelpBusinessDataToElasticSearch)

### Motivation
This entire project is inspired from this Tech talk: [Evolution of ElasticSearch at Yelp](https://speakerdeck.com/elastic/the-evolution-of-elastic-search-at-yelp). A few slides towards the end, the talk highlights some of the future activities that will be done at Yelp using ElasticSearch. One of which is moving of Business data to ElasticSearch, and support better Geo-sharding. This project is my attempt to move business data meticulously on ElasticSearch cluster, and visualize it. 

### Dataset
Yelp's Business Dataset is obtained from Yelp's Dataset Challenge at [this](http://www.yelp.com/dataset_challenge) link. Due to the mammothness of the dataset, this repository contains only a miniscule version of the original file under the `data` directory.

### Sharding
One of the requirement of the problem statement states that the data be efficiently geosharded. Having a close look at the data, we see that the `state` field can be used in the `routing` parameter. Documents with the same `state` code fall under the same shard. For this example, I have taken the state as the factor which will separate the documents geographically. But, in the bigger picture, when data from all the supported cities around the globe is indexed, the geosharding can be influenced by the following factors: 
* Physical proximity to the cluster geolocation
* Concentration of businesses in specific locations
* `country` field, in case the distribution of data is fairly distributed across the globe

### Verification of data distribution in different shards
The infrastructure used for the project are three ElasticSearch nodes running in the same cluster (the system is also tried and tested on Amazon EC2 instance. I wanted to test cluster deployment with ElasticSearch nodes running in two different data centers, say, Singapore and US West, but couldn't test that because of subnet issues). In addition to ElasticSearch nodes, running with a `green` cluster health status, Kibana and Logstash nodes are also present.

I trimmed the input business data JSON file to a lesser number of records (100), having variegated state codes, and pushed the documents to ElasticSearch. REST API call of `_status` returns the data distribution across the entire cluster, and all the information about the index. For this small data it is clearly evident, that the document distribution works perfect! Documents with different state codes land up in different shards. It would have been amazing to see this feature working in live servers distributed geographically.

### Sharding Caveats
While working on sharding and routing, I also had the chance of interacting with the Elastic community on [Discuss](https://discuss.elastic.co), and there I learnt that, though routing states that documents with same routing IDs land up in same shards, but this feature is not guaranteed. We always run at the risk of non-uniform data distribution because of the nature of our data. For instance, if the number of documents containing the routing ID, `California`, is more than that of `North Dakota`, then the data is not distributed evenly. Also, in such scenarios, ElasticSearch takes care of distributing the data efficiently. 

One suggestion that the community presents in this scenario is that, to guarantee that the documents land up in desirable shards, use separate indexes.
