# Timeseries databases comparison

## InfluxDb
Sharding feature is only for enterprise version.

## KairosDB
KairosDB is a fast distributed scalable time-series database. It was initially a rewrite of the original OpenTSDB project, but evolved to a different system for which data management, data processing, and visualization are fully separated. 

In [this article](http://www.scylladb.com/2017/05/24/9-steps-building-highly-available-time-series-solution-scylla-kairosdb/) an alternative solution is proposed which combine KairosDB and Scylla. Scylla is here used as backend database (rather than Cassandra).

With this solution we can have a scylla table per metric. 

Pro.
- Efficient and scalable

Cons. 
- External Kerberos authentication is an enterprise feature ( upcoming security feature: 2018.2).

## Chronix
Chronix is built to store time series highly compressed and for fast access times.  It is based on Solr (which guarantees  scalability, fault tolerance, distributed indexing and replication) and has an embedded spark connector for analysis scope (see [here](https://github.com/ChronixDB/chronix.spark) for more details.).

Pro.
- Chronix provides a grafana plugin for timeseries visualization (see [here](https://github.com/ChronixDB/chronix.grafana) for more details). 

Cons:
- Chronix is based on Apache Solr version 6.4.2. However Cloudera 5.12.0 supports an older version (i.e. 4.10.3)


## Elasticsearch
In my knowledge, does not exist any mappings for modeling datapoints in an optimized way (see [here](https://stackoverflow.com/questions/44544529/elasticsearch-mapping-for-timeseries) for a possible and  general purpose solution).

Pro.
- Grafana provides the Elasticsearch plugin (see [here](http://docs.grafana.org/features/datasources/elasticsearch/) for more details).


## Kudu
Kudu is a much lower level datastore than timeseries databases. Its more like a distributed file system that provides a few database like features than a full fledged database (this is repeated multiple times in  the [faq section](https://kudu.apache.org/faq.html#does-kudu-support-dynamic-partitioning)). It currently relies on a query engine such as Impala for finding data stored in Kudu. 

Kudu stores data in tablets and offers two ways of partitioning data: Range Partitions, Hash based Partitioning or both. See [here](https://kudu.apache.org/docs/schema_design.html#partitioning) and [here](https://www.linkedin.com/pulse/storing-data-range-hash-partitions-kudu-arkanil-dutta) for more details. This means that to storage timeseries it is required a partition by time range and series hash (i.e. all tags attribute of a timeseries). 

Cons.
- Kudu is a is a storage engine not a SQL engine. This means that it is not optimized for timeseries analytic queries as traditional timeseries databases. However, impala or spark could be used for querying and analyzing data stored in kudu.
- Grafana does not provide any Kudu plugins.
