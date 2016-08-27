---
title: "Elastic 2.x integration testing."
tags:
  - java
  - elastic
  - testing
---

Moving from elastic search 1 to elastic 2 comes with some breaking changes. 
One of those changes is the setup of an in-memory node. 

In elastic search 1 it was possible to create a node that didn't store any data on disk (except for some log files). 
This was useful for creating a fast in memory node that could be used for testing. 
Unfortunately this feature was removed with the release of elastic 2. 
This change can be found in the documentation [here](https://www.elastic.co/guide/en/elasticsearch/reference/2.0/breaking_20_setting_changes.html#_in_memory_indices). 

Elastic has a separate testing jar that comes with a 'ESIntegTestCase' class that you can extend. 
Documentation [here](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/integration-tests.html). 
Using this class was not as easy as hoped and it seemd to create a rather heavy cluster with multiple nodes.

A better approach seemed to alter the in-memory setup for elastic search 1 as little as possible until it worked again. 
The biggest change is altering the 'index.store.type' to 'default' (from 'memory') and including a 'path.home' directory to store data on disk. 
Below is the complete code of the InMemoryMongo class (that is now not so in memory more).  
{% gist codingtim/9f53ff83172ad23f87055db463db1871 %}

This class can be used in a test as demonstrated below.

{% gist codingtim/c89dd5a9a77976b987ad55c0212c9c20 %}
