---
layout: post
category : database
tags : [nosql,database]
title: MongoDB 3.0 Major Release Note
---

## MongoDB 3.0 Major Changes
-----------------------------------------------------------

### Pluggable Storage Engine API

MongoDB 3.0 introduces a pluggable storage engine API that allows third parties to develop storage engines for MongoDB.

### WiredTiger

MongoDB 3.0 introduces support for the WiredTiger storage engine. With the support for WiredTiger, MongoDB now supports two storage engines:

* MMAPv1, the storage engine available in previous versions of MongoDB and the default storage engine for MongoDB 3.0, and
* WiredTiger, available only in the 64-bit versions of MongoDB 3.0.

#### WiredTiger Usage

WiredTiger is an alternate to the default MMAPv1 storage engine. WiredTiger supports all MongoDB features, including operations that report on server, database, and collection statistics. Switching to WiredTiger, however, requires a change to the on-disk storage format. For instructions on changing the storage engine to WiredTiger, see the appropriate sections in the Upgrade MongoDB to 3.0 documentation.

MongoDB 3.0 replica sets and sharded clusters can have members with different storage engines; however, performance can vary according to workload. For details, see the appropriate sections in the Upgrade MongoDB to 3.0 documentation.

The WiredTiger storage engine requires the latest official MongoDB drivers. For more information, see WiredTiger and Driver Version Compatibility.

> SEE ALSO  
> Support for touch Command, WiredTiger Storage Engine section in the Storage documentation

#### WiredTiger Configuration

To configure the behavior and properties of the WiredTiger storage engine, see storage.wiredTiger configuration options. You can set WiredTiger options on the command line.

> SEE ALSO  
> WiredTiger Storage Engine section in the Storage documentation

WiredTiger Concurrency and Compression <br />
The 3.0 WiredTiger storage engine provides document-level locking and compression.

By default, WiredTiger compresses collection data using the snappy compression library. WiredTiger uses prefix compression on all indexes by default.

> SEE ALSO  
> WiredTiger section in the Production Notes, the blog post New Compression Options in MongoDB 3.0

_WiredTiger特性_  

    - 支持文档级别并发控制  
    - 支持对所有集合和索引进行Block压缩和前缀压缩(包括journal)  
    - 通过storage.wiredTiger.engineConfig.cacheSizeGB可控制MongoDB所能使用的最大内存(该参数默认值为物理内存大小的一半)  

### MMAPv1 Improvements

#### MMAPv1 Concurrency Improvement

In version 3.0, the MMAPv1 storage engine adds support for collection-level locking.

#### MMAPv1 Configuration Changes
To support multiple storage engines, some configuration settings for MMAPv1 have changed. See Configuration File Options Changes.

#### MMAPv1 Record Allocation Behavior Changes

MongoDB 3.0 no longer implements dynamic record allocation and deprecates paddingFactor. The default allocation strategy for collections in instances that use MMAPv1 is power of 2 allocation, which has been improved to better handle large document sizes. In 3.0, the usePowerOf2Sizes flag is ignored, so the power of 2 strategy is used for all collections that do not have noPadding flag set.

For collections with workloads that consist only of inserts or in-place updates (such as incrementing counters), you can disable the power of 2 strategy. To disable the power of 2 strategy for a collection, use the collMod command with the noPadding flag or the db.createCollection() method with the noPadding option.

> WARNING  
> Do not set noPadding if the workload includes removes or any updates that may cause documents to grow. For more information, see No Padding Allocation Strategy.

When low on disk space, MongoDB 3.0 no longer errors on all writes but only when the required disk allocation fails. As such, MongoDB now allows in-place updates and removes when low on disk space.

> SEE ALSO  
> Dynamic Record Allocation


_MMAPv1存储引擎优化_

    - 并发锁粒度由数据库级别锁提升为集合级别锁
    - 抛弃了基于paddingFactor的自适应分配方式,基于usePowerOf2Sizes的预分配方式成为默认的文档空间分配方式

## Replica Sets

### Increased Number of Replica Set Members

In MongoDB 3.0, replica sets can have up to 50 members. [1] The following drivers support the larger replica sets:

* C# (.NET) Driver 1.10
* Java Driver 2.13
* Python Driver (PyMongo) 3.0
* Ruby Driver 2.0
* Node.JS Driver 2.0

The C, C++, Perl, and PHP drivers, as well as the earlier versions of the Ruby, Python, and Node.JS drivers, discover and monitor replica set members serially, and thus are not suitable for use with large replica sets.
[1]	The maximum number of voting members remains at 7.

### Replica Set Step Down Behavior Changes

The process that a primary member of a replica set uses to step down has the following changes:

* Before stepping down, replSetStepDown will attempt to terminate long running user operations that would block the primary from stepping down, such as an index build, a write operation or a map-reduce job.
* To help prevent rollbacks, the replSetStepDown will wait for an electable secondary to catch up to the state of the primary before stepping down. Previously, a primary would wait for a secondary to catch up to within 10 seconds of the primary (i.e. a secondary with a replication lag of 10 seconds or less) before stepping down.
* replSetStepDown now allows users to specify a secondaryCatchUpPeriodSecs parameter to specify how long the primary should wait for a secondary to catch up before stepping down.

### Other Replica Set Operational Changes

* Initial sync builds indexes more efficiently for each collection and applies oplog entries in batches using threads.
* Definition of w: “majority” write concern changed to mean majority of voting nodes.
* Stronger restrictions on Replica Set Configuration. For details, see Replica Set Configuration Validation.
* For pre-existing collections on secondary members, MongoDB 3.0 no longer automatically builds missing _id indexes.

> SEE ALSO  
> Replication Changes in Compatibility Changes in MongoDB 3.0


_Replica Set 优化_

    - 复制集成员增长到50个。但能够投票的最大成员个数依然为7个
    - Primary节点StepDown处理方式变化

## Sharded Clusters

MongoDB 3.0 provides the following enhancements to sharded clusters:

* Adds a new sh.removeTagRange() helper to improve management of sharded collections with tags. The new sh.removeTagRange() method acts as a complement to sh.addTagRange().
* Provides a more predictable read preference behavior. mongos instances no longer pin connections to members of replica sets when performing read operations. Instead, mongos reevaluates read preferences for every operation to provide a more predictable read preference behavior when read preferences change.
* Provides a new writeConcern setting to configure the write concern of chunk migration operations. You can configure the writeConcern setting for the balancer as well as for moveChunk and cleanupOrphaned commands.
* Improves visibility of balancer operations. sh.status() includes information about the state of the balancer. See sh.status() for details.

> SEE ALSO  
> Sharded Cluster Setting in Compatibility Changes in MongoDB 3.0


_Sharded Clusters 优化_

    - 新增工具函数 sh.removeTagRange()
    - 提供更可预测的Read Preference处理
    - 为chunk迁移提供writeConcern设置
    - 增加均衡器状态显示


## Security Improvements

MongoDB 3.0 includes the following security enhancements:

* MongoDB 3.0 adds a new SCRAM-SHA-1 challenge-response user authentication mechanism. SCRAM-SHA-1 requires a driver upgrade if your current driver version does not support SCRAM-SHA-1. For the driver versions that support SCRAM-SHA-1, see Upgrade Drivers.
* Increases restrictions when using the Localhost Exception to access MongoDB. For details, see Localhost Exception Changed.

> SEE ALSO  
> Security Changes  


## Improvements

### New Query Introspection System

MongoDB 3.0 includes a new query introspection system that provides an improved output format and a finer-grained introspection into both query plan and query execution.

For details, see the new db.collection.explain() method and the new explain command as well as the updated cursor.explain() method.

For information on the format of the new output, see Explain Results.

### Enhanced Logging
To improve usability of the log messages for diagnosis, MongoDB categorizes some log messages under specific components, or operations, and provides the ability to set the verbosity level for these components. For information, see Log Messages.

### MongoDB Tools Enhancements

All MongoDB tools except for mongosniff and mongoperf are now written in Go and maintained as a separate project.

* New options for parallelized mongodump and mongorestore. You can control the number of collections that mongorestore will restore at a time with the --numParallelCollections option.
* New options -excludeCollection and --excludeCollectionsWithPrefix for mongodump to exclude collections.
* mongorestore can now accept BSON data input from standard input in addition to reading BSON data from file.
* mongostat and mongotop can now return output in JSON format with the --json option.
* Added configurable write concern to mongoimport, mongorestore, and mongofiles. Use the --writeConcern option.
* mongofiles now allows you to configure the GridFS prefix with the --prefix option so that you can use custom namespaces and store multiple GridFS namespaces in a single database.

> SEE ALSO  
> MongoDB Tools Changes  

### Indexes

* Background index builds will no longer automatically interrupt if dropDatabase, drop, dropIndexes operations occur for the database or collection affected by the index builds. The dropDatabase, drop, and dropIndexes commands will still fail with the error message a background operation is currently running, as in 2.6.
* If you specify multiple indexes to the createIndexes command,
the command only scans the collection once, and
if at least one index is to be built in the foreground, the operation will build all the specified indexes in the foreground.
* For sharded collections, indexes can now cover queries that execute against the mongos if the index includes the shard key.

> SEE ALSO  
> Indexes in Compatibility Changes in MongoDB 3.0  


### Query Enhancements
MongoDB 3.0 includes the following query enhancements:

* For geospatial queries, adds support for “big” polygons for $geoIntersects and $geoWithin queries. “Big” polygons are single-ringed GeoJSON polygons with areas greater than that of a single hemisphere. See $geometry, $geoIntersects, and $geoWithin for details.
* For aggregate(), adds a new $dateToString operator to facilitate converting a date to a formatted string.
* Adds the $eq query operator to query for equality conditions.

> SEE ALSO  
> 2d Indexes and Geospatial Near Queries

### Distributions and Supported Versions

Most non-Enterprise MongoDB distributions now include support for TLS/SSL. Previously, only MongoDB Enterprise distributions came with TLS/SSL support included; for non-Enterprise distributions, you had to build MongoDB locally with the --ssl flag (i.e. scons --ssl).

32-bit MongoDB builds are available for testing, but are not for production use. 32-bit MongoDB builds do not include the WiredTiger storage engine.

MongoDB builds for Solaris do not support the WiredTiger storage engine.

MongoDB builds are available for Windows Server 2003 and Windows Vista (as “64-bit Legacy”), but the minimum officially supported Windows version is Windows Server 2008.

> SEE ALSO  
> Platform Support, What are the limitations of 32-bit versions of MongoDB?


### Package Repositories

Non-Enterprise MongoDB Linux packages for 3.0 are in a new repository. Follow the appropriate Linux installation instructions to install the 3.0 packages from the new location.

