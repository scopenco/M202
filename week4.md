# CHAPTER 4: SHARDED CLUSTER MANAGEMENT

# Videos

* [Starting and stopping the balancer](https://www.youtube.com/watch?v=exzSK-yjoqc)
* [When does the balancer kick in?](https://www.youtube.com/watch?v=fzSGRtNLWyY)
* [How does the balancer pick chunks?](https://www.youtube.com/watch?v=T1w6P5TPN6c)
* [Scaling out with shards: Practical considerations](https://www.youtube.com/watch?v=iE8BXU4RYnU)
* [Config servers](https://www.youtube.com/watch?v=RnLcmVcYrVU)
* [The config database](https://www.youtube.com/watch?v=DdctidkwpFY)
* [The Config Database: Vital Collections](https://www.youtube.com/watch?v=UFRgGFxA_bg)
* [Upgrades on sharded clusters](https://www.youtube.com/watch?v=3XrnQ9yFwfU)
* [Services a mongos performs](https://www.youtube.com/watch?v=NfbTNoD82rs)
* [Mongos processes do connection pooling](https://www.youtube.com/watch?v=YJ_TOKkIHv8)
* [Mongos Process: Refreshing its Cache](https://www.youtube.com/watch?v=Wo5TBZmfnHI)
* [Chunk Splitting Overview](https://www.youtube.com/watch?v=d692uIssk0k)
* [Chunk splitting algorithm](https://www.youtube.com/watch?v=b8dfn8eSlS8)
* [Manual chunk splitting](https://www.youtube.com/watch?v=YXfMxZbEBuI)
* [Jumbo chunks](https://www.youtube.com/watch?v=HwB9moUEBEI)
* [Manual pre-splitting overview](https://www.youtube.com/watch?v=9hDfnRdVoeg)
* [Manual pre-splitting example: Describing the data](https://www.youtube.com/watch?v=-d8XpvvVABU)
* [Manual pre-splitting example: Actually pre-splitting the data](https://www.youtube.com/watch?v=BcBTTJli8nc)
* [Tag-based sharding overview -- why use it?](https://www.youtube.com/watch?v=HMrWxcNR2Oc)
* [Simple tag-based sharding example](https://www.youtube.com/watch?v=m_djWGAaoic)
* [Tag-based sharding caveats](https://www.youtube.com/watch?v=pFdyaZoxqsI)
* [Hash-based sharding overview](https://www.youtube.com/watch?v=98dRSnHtIUc)
* [Hash-based sharding demo](https://www.youtube.com/watch?v=UnzGPMddPH0)
* [Hash-based sharding pros and cons](https://www.youtube.com/watch?v=MQlnJq7nzEA)
* [Empty chunks](https://www.youtube.com/watch?v=PAyFFj_n1yk)
* [Data imbalance](https://www.youtube.com/watch?v=-cdmEIgMUY4)
* [Removing a shard](https://www.youtube.com/watch?v=kSlGXXe215Q)

# Homeworks
## Homework: 4.1: Pre-splitting data
### Question
For this assignment, you will be pre-splitting chunks into ranges. We'll be working with the "m202.presplit" database/collection.

First, create 3 shards. You can use standalone servers or replica sets on each shard, whatever is most convenient.

Pre-split your collection into chunks with the following ranges, and put the chunks on the appropriate shard, and name the shards "shard0", "shard1", and "shard2". Let's make the shard key the "a" field. Don't worry if you have other ranges as well, we will only be checking for the following:
```
  Range  / Shard
 0 to 7  / shard0
 7 to 10 / shard0
10 to 14 / shard0
14 to 15 / shard1
15 to 20 / shard1
20 to 21 / shard1
21 to 22 / shard2
22 to 23 / shard2
23 to 24 / shard2
```
reminder: Chunk ranges are inclusive of the lower bound and exclusive of the upper bound (link)[http://docs.mongodb.org/manual/core/sharding-chunk-splitting/?&_ga=1.45562080.597017498.1456007075#limitations].

You may have other chunks with other ranges (you undoubtedly will if you are solving the problem correctly!) but we will only be checking for these. Keep in mind that if your balancer is on, it may move chunks around if it detects an imbalance. Also, remember that you can check the state of your sharded cluster by calling sh.status(true).

Please keep in mind that this problem has only been tested in the course VM.

### Answer
* Deploy sharding cluster with 3 shards
```bash
mkdir -p shard/s1
mkdir -p shard/s2
mkdir -p shard/s3
mkdir -p shard/cfg

mongod --configsvr --dbpath shard/cfg --port 27003 --fork --logpath log.cfg --logappend -bind_ip 127.0.0.1
mongod --shardsvr --dbpath shard/s1 --logpath log.s1 --port 27000 --fork --logappend --smallfiles --oplogSize 50 -bind_ip 127.0.0.1
mongod --shardsvr --dbpath shard/s2 --logpath log.s2 --port 27001 --fork --logappend --smallfiles --oplogSize 50 -bind_ip 127.0.0.1
mongod --shardsvr --dbpath shard/s3 --logpath log.s3 --port 27002 --fork --logappend --smallfiles --oplogSize 50 -bind_ip 127.0.0.1
mongos --configdb localhost:27003 --fork --logappend --logpath log.mongos -bind_ip 127.0.0.1

mongo --port 27017
> use admin
> db.runCommand({addShard: "localhost:27000", name: "shard0"})
> db.runCommand({addShard: "localhost:27001", name: "shard1"})
> db.runCommand({addShard: "localhost:27002", name: "shard2"})
```
* Configure sharding db and collection
```bash
mongo --port 27017
> use m202
> sh.enableSharding("m202")
> db.presplit.insert({"a":0})
> db.presplit.ensureIndex({a:1})
> sh.shardCollection("m202.presplit", {"a" : 1})
```
* Stop balancer and split collection to chunks
```bash
mongo --port 27017
> use admin
> sh.stopBalancer()
> db.runCommand( { split : "m202.presplit", middle : { a : 0 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 7 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 10 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 14 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 15 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 20 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 21 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 22 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 23 } } )
> db.runCommand( { split : "m202.presplit", middle : { a : 24 } } )
```
* Migrate chunks to selected shards
```bash
mongo --port 27017
> use admin
> sh.moveChunk("m202.presplit", { a: 14 }, "shard1")
> sh.moveChunk("m202.presplit", { a: 15 }, "shard1")
> sh.moveChunk("m202.presplit", { a: 20 }, "shard1")
> sh.moveChunk("m202.presplit", { a: 21 }, "shard2")
> sh.moveChunk("m202.presplit", { a: 22 }, "shard2")
> sh.moveChunk("m202.presplit", { a: 23 }, "shard2")
```

## Homework: 4.2: Advantages of pre-splitting
### Question
Which of the following are advantages of pre-splitting the data that is being loaded into a sharded cluster, rather than throwing all of the data in and waiting for it to migrate?

### Answer
```
- Data can get lost during chunk migration. Pre-splitting allows you to avoid that.
- You have more control over the shard key if you pre-split.
+ You can decide which shard has which data range initially if you pre-split the data
+ Migration takes time, especially when the system is under load. Pre-splitting allows you to avoid that.
```

## Homework: 4.3: Upgrading a sharded cluster to 3.0
### Question
Which of the following are required or recommended when upgrading a sharded cluster to MongoDB 3.0? Check all that apply.

### Answer
```bash
+ If your MongoDB deployment is not already running MongoDB 2.6, upgrade to 2.6 first.
+ Upgrade all mongos instances before upgrading mongod instances.
- Upgrade all mongod instances before upgrading mongos instances.
+ Disable the balancer.
- Upgrade the secondaries on all shards before upgrading any primary.
```

## Homework: 4.4: Using shard tags to manage data
### Question
In this problem we will emulate a data management strategy in which we periodically move data from short-term storage (STS) to long-term storage (LTS). We have implemented this strategy using tag-based sharding.

Start by spinning up a sharded cluster as follows:
```
$ mongo --nodb
> config = { d0 : { smallfiles : "", noprealloc : "", nopreallocj : ""}, d1 : { smallfiles : "", noprealloc : "", nopreallocj : "" }, d2 : { smallfiles : "", noprealloc : "", nopreallocj : ""}};
> cluster = new ShardingTest( { shards : config } );
```

This will create 3 standalone shards on ports 30000, 30001, and 30002, as well as a mongos on port 30999. The config portion of the above will eliminate the disk space issues some students have seen when using ShardingTest.

Next, initialize the data in this cluster using MongoProc. You can choose the host you're pointing to, but MongoProc will be looking for the mongos at port 30999.

Following initialization, your system will contain the collection testDB.testColl. Once initial balancing completes, all documents in this collection with a createdDate field value from October 1, 2013 until the end of 2013 are in LTS and all documents created in 2014 are in STS. There are two shards used for LTS and one shard for STS.

Your assignment is to move all data for the month of January 2014 into LTS as part of periodic maintenance. For this problem we are pretty sure you can "solve" it in a couple of ways that are not ideal. In an ideal solution you will make the balancer do the work for you. Please note that the balancer must be running when you turn in your solution.

### Answer
* Deploy sharding cluster with 3 shards
```bash
mkdir -p shard/s1
mkdir -p shard/s2
mkdir -p shard/s3
mkdir -p shard/cfg

mongod --configsvr --dbpath shard/cfg --port 27003 --fork --logpath log.cfg --logappend -bind_ip 127.0.0.1
mongod --shardsvr --dbpath shard/s1 --logpath log.s1 --port 27000 --fork --logappend --smallfiles --oplogSize 50 -bind_ip 127.0.0.1
mongod --shardsvr --dbpath shard/s2 --logpath log.s2 --port 27001 --fork --logappend --smallfiles --oplogSize 50 -bind_ip 127.0.0.1
mongod --shardsvr --dbpath shard/s3 --logpath log.s3 --port 27002 --fork --logappend --smallfiles --oplogSize 50 -bind_ip 127.0.0.1
mongos --configdb localhost:27003 --fork --logappend --port 30999 --logpath log.mongos -bind_ip 127.0.0.1

mongo --port 30999
> use admin
> db.runCommand({addShard: "localhost:27000", name: "shard0"})
> db.runCommand({addShard: "localhost:27001", name: "shard1"})
> db.runCommand({addShard: "localhost:27002", name: "shard2"})
```
* Stop Balancer
```bash
mongo --port 30999
> sh.stopBalancer();
```
* Create new tag ranges
```
mongo --port 30999
mongos> sh.addTagRange("testDB.testColl", {createdDate : ISODate("2013-10-01T00:00:00Z")}, {createdDate : ISODate("2014-02-01T00:00:00Z")}, "LTS")
mongos> sh.addTagRange('testDB.testColl', {createdDate : ISODate("2014-02-01")}, {createdDate : ISODate("2014-05-01")}, "STS")
```
* Start Balancer
```bash
mongo --port 30999
> sh.stopBalancer();
```
* Remove old tag range and balancer will start migration
```bash
mongo --port 30999
mongos> db.tags.remove({_id : { ns : "testDB.testColl", min : { createdDate : ISODate("2014-01-01")} }, tag: "STS" })
```
