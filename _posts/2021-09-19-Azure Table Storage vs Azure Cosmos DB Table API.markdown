# Azure Table Storage vs Azure Cosmos DB Table API

## Overview

Azure provides two options for storing massive semi-structured datasets in NoSQL key-value store. This article lists basic feture comparison and performance measurement to help decide which one is the best to use in different scenarios.

### Azure Table Storage

Azure Table Storage is a data service part of an Azure Storage Account offering key-value store for semi structured data. Each entity in a Table Storage has 3 system properties - partition key, row key, timestamp - and up to 252 custom properties that can be added in an ad-hoc manner (schemaless design) allowing data structure changes over time. The partition key - row key pair acts as composite key of the table, using them for in queries (point query) returns quick results.

**Highlighted features**

-   Priced based on data size and transaction amount - relatively cheap
-   No secondary indexes - only point queries provide good performance

### Azure Cosmos DB - Table API

Azure Cosmos DB is a feature packed NoSQL database that expose several APIs (Table API, SQL API, Apache Cassandra, MongoDB, Gremlin). The main feature is the global distribution (geo-redundancy) that provides worldwide guaranteed low latency access for local applications in any number of regions. The various APIs allow easy migration between databases - as an example, the cosmos DB Table API uses the same C# SDK as for Table Storage making the switch between databases as easy as replacing the connection string in the configuration.
The billing is calculated based on throughput using allocated RU/s (reading unit). As a recent addition, it's now available to set throughput auto-scaling with max RU/s configuration in which case the billing is calculated hourly using the highest scaled RU/s for that given hour.

**Highlighted Features**

-   Turnkey global distribution
-   Dedicated throughput worldwide
-   Single-digit millisecond latencies at the 99th percentile
-   Guaranteed high availability
-   Automatic secondary indexing
-   Priced based on throughput and multiplies by regions used - relatively pricey

## Comparison

### Features

|                   |                      Azure Cosmos DB                      |           Azure Table Storage           |
| :---------------: | :-------------------------------------------------------: | :-------------------------------------: |
|    Throughput     |    No upper limit, supports >10M operations per second    |     Up to 20K operations per second     |
|    Redundancy     |          Multiple configurable worldwide regions          | One optional secondary read-only region |
|      Latency      |                 Single-digit milliseconds                 |             No upper bounds             |
|    Consistency    |              Five defined consistency models              | Strong and Eventual consistency models  |
|       Query       | Improved query performance because all fields are indexed |   Optimized for query on primary key    |
|     Failover      |              Automatic and manual failovers               |         Can't initiate failover         |
|      Billing      |                     Throughput-based                      |              Storage-based              |
| Entity size (max) |                           2 MB                            |                  1 MB                   |

### Performance

The performance test scenario is inserting 80K small (<1KB) records in Cosmos DB and Table storage (the Cosmos DB throughput is set to 400 RU/s).

**Cosmos DB results**

-   operation duration: 27:33 (mm:ss)
-   insert rate: ~48 records/second
-   estimated query rate: expect ~250record/second
-   monthly cost: ~£24/month

**Table storage results**

-   operation duration: 1:45 (m:ss)
-   insert rate: ~750 records/second
-   estimated Cosmos DB throughput equivalent: 6,000 RU/s
-   monthly cost: ~£3/month

## Conclusion

Microsoft is using heavy marketing to promote Cosmos DB and its advanced features but it comes with high costs. The test scenario above shows that Cosmos DB costs over 100x more then Storage Table calculating with the same throughput. Using Cosmos DB is justified if the advanced features are required like multiple region redundancy, control of failover scenarios and guaranteed minimal latency but the Table Storage performance is also very good and the cost considerations make it a better choice for less complex single region scenarios.

## Author
Gergely Vajda

Senior Integration Engineer
