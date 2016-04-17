# CHAPTER 3: FAULT TOLERANCE AND AVAILABILITY

# Videos

* [Rolling maintenance](https://www.youtube.com/watch?v=OO3tpaC4L4g)
* [Rolling maintenance use cases](https://www.youtube.com/watch?v=LH22a2FATho)
* [Can I put a load balancer in front of a replica set?](https://www.youtube.com/watch?v=fqcDrfwvygU)
* [Using load balancers with sharded clusters](https://www.youtube.com/watch?v=LW_mlmvwWDM)
* [Driver options: connections](https://www.youtube.com/watch?v=-kzJ44CbCsM)
* [Driver options: socket timeout](https://www.youtube.com/watch?v=Vxmu2a60t0g)
* [Driver options: high availability](https://www.youtube.com/watch?v=K-HCSA83iqI)
* [Connection management in replica sets](https://www.youtube.com/watch?v=RFsNyf1mBhs)
* [Connection management in sharded clusters](https://www.youtube.com/watch?v=k3wEg0LQX-E)
* [Formula for maximum mongos connections](https://www.youtube.com/watch?v=CVFqJ80ZpIw)
* [Read preferences](https://www.youtube.com/watch?v=TuDMmq7oGHU)
* [Rollback](https://www.youtube.com/watch?v=XEzwehA46bQ)
* [Other MongoDB states](https://www.youtube.com/watch?v=l09AtjdZaew)

# Homeworks
## Homework: 3.1: Limiting connections to a primary
### Question
Suppose you have a sharded cluster for which each shard is a 3-node replica set running MongoDB 3.0. You are concerned about limiting the number of connections in order to help ensure predictable behavior. Given the constraints of your system, you have decided to use 10GB as the amount of memory allocated to determine the maximum connections on the primary.

Using the formula discussed earlier in this chapter, determine the value to which you should set maxConns for each mongos in this cluster if you want to limit primary connections to a ~90% utilization level. Your cluster is using 64 mongos routers, and the number of other possible connections to a primary is 6.

Please calculate the maxConns value to achieve 90% utilization and round down to the nearest integer. Enter only the integer in the box below or your answer will be considered incorrect.

Note: For consistency with the lesson video, please assume that 10,000 connections consume 10GB of RAM.

### Answer
We have:
* 90% of 10000 conns is 9000
* 2 secondaries
* 6 other connections
* 64 mongos
```
(9000 - (2 * 3) - (6 * 3))/64 = 140
```

## Homework: 3.2: Optimizing a secondary for special case reads
### Question
Suppose you have an application on which you want to run analytics monthly against a MongoDB 3.0 server. The analytics require an index and for performance reasons you will create the index on a secondary. Initiate a replica set with a primary, a secondary, and an arbiter. Create an index on the secondary only. The index should be on the "a" field of the "testDB.testColl" collection.

When you have created the index on the secondary, test with MongoProc to be sure you've completed the problem correctly and then submit.

### Answer
* Deploy replica set
```bash
mongo --nodb
> var rst = new ReplSetTest ({name: 'testSet', nodes: 3});
> rst.startSet();
mongo --port 2000
> rs.initiate();
> rs.add("centos6.dev:20001")
> rs.add("centos6.dev:20002")
```
* Insert some data to testDB.testColl collection
```bash
mongo --port 20000
use testDB
for (var i = 1; i <= 100; i++) {
   db.testColl.insert( { a : i } )
}
```
* Stop slave
```bash
[root@centos6 ~]# ps ax | grep mongod
14772 pts/2    Sl+    0:17 mongod --oplogSize 40 --port 20000 --noprealloc --smallfiles --replSet testSet --dbpath /data/db/testSet-0 --setParameter enableTestCommands=1
14797 pts/2    Sl+    0:17 mongod --oplogSize 40 --port 20001 --noprealloc --smallfiles --replSet testSet --dbpath /data/db/testSet-1 --setParameter enableTestCommands=1
14822 pts/2    Sl+    0:14 mongod --oplogSize 40 --port 20002 --noprealloc --smallfiles --replSet testSet --dbpath /data/db/testSet-2 --setParameter enableTestCommands=1
15213 pts/0    S+     0:00 grep mongod
[root@centos6 ~]# kill 14797
```
* Restart slave without option relset and other port
```bash
mongod --oplogSize 40 --port 20101 --noprealloc --smallfiles --dbpath /data/db/testSet-1
```
* Create index for testDB.testColl
```bash
mongo --port 20101
> use testDB
switched to db testDB
> db.testColl.createIndex({a:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```
* Restart slave with old replSet/port
```bash
mongod --oplogSize 40 --port 20001 --noprealloc --smallfiles --replSet testSet --dbpath /data/db/testSet-1 --setParameter enableTestCommands=1
```

## Homework: 3.3: Triggering Rollback
### Question
In this problem, you will be causing a rollback scenario on a MongoDB 3.0 server. Set up a replica set with one primary, one secondary, and an arbiter. Write some data to the primary in such a way that it does not replicate on the secondary (perhaps by taking down the secondary) then remove the primary from the replica set. Let the secondary come back up and become primary, write to it, and then bring up the original primary (it will now be a secondary).

Once you've triggered the rollback scenario, please submit your work using MongoProc. You do not need to reload the data lost during rollback. For your solution to be graded correct, you must have one secondary in the replica set on which rollback has occurred.

You must run mongoProc from the machine that experiences rollback. MongoProc will look for the server at host='localhost' regardless of your settings, and you will need to direct it to the port that your (new) primary is on.

Adam wrote a tutorial blog post on simulating rollback. It might be a little outdated, but provides a detailed discussion of this topic and is worth the read.

### Answer
* Deploy replica set with secondary and arbiter
```
mkdir -p pri sec arb
mongod --port 30001 --dbpath pri --replSet test --smallfiles --oplogSize 128 --nojournal
mongod --port 30002 --dbpath sec --replSet test --smallfiles --oplogSize 128 --nojournal
mongod --port 30003 --dbpath arb --replSet test --nojournal
mongo --port 30001
rsconf = { _id: "test", members: [{_id: 0, host: "localhost:30001"}]}
rs.initiate(rsconf)
rs.add("localhost:30002")
rs.addArb("localhost:30003")

```
* Insert data to primary
```bash
mongo --port 30001
use testDB
for (var i = 1; i <= 10000; i++) { db.testColl.insert({a:i}); sleep (10)}
```
* Stop slave
* Stop master and start slave
* Insert data to new elected master
```
mongo --port 30002
use testDB
for (var i = 1; i <= 10000; i++) { db.testColl.insert({a:i}); sleep(10); }
```
* Start master again and see in logs
```
2016-04-10T07:43:26.422-0700 I REPL     [ReplicationExecutor] transition to ROLLBACK
2016-04-10T07:43:26.422-0700 I REPL     [rsBackgroundSync] rollback 1
2016-04-10T07:43:26.423-0700 I REPL     [rsBackgroundSync] rollback 2 FindCommonPoint
2016-04-10T07:43:26.423-0700 I REPL     [rsBackgroundSync] rollback our last optime:   Apr 10 07:39:55:8
2016-04-10T07:43:26.423-0700 I REPL     [rsBackgroundSync] rollback their last optime: Apr 10 07:43:16:10c9
2016-04-10T07:43:26.423-0700 I REPL     [rsBackgroundSync] rollback diff in end of log times: -201 seconds
2016-04-10T07:43:26.433-0700 I REPL     [rsBackgroundSync] rollback 3 fixup
2016-04-10T07:43:26.446-0700 I REPL     [rsBackgroundSync] rollback 3.5
2016-04-10T07:43:26.446-0700 I REPL     [rsBackgroundSync] rollback 4 n:1
2016-04-10T07:43:26.446-0700 I REPL     [rsBackgroundSync] minvalid=(term: 2, timestamp: Apr 10 07:43:16:10c9)
2016-04-10T07:43:26.456-0700 I REPL     [rsBackgroundSync] rollback 4.6
2016-04-10T07:43:26.456-0700 I REPL     [rsBackgroundSync] rollback 4.7
2016-04-10T07:43:26.462-0700 I REPL     [rsBackgroundSync] rollback 5 d:346 u:0
2016-04-10T07:43:26.462-0700 I REPL     [rsBackgroundSync] rollback 6
2016-04-10T07:43:26.463-0700 I REPL     [rsBackgroundSync] rollback done
```

Mongodb will create rollback file in primary directory pri/rollback/testDB.testColl.2016-04-10T14-43-26.0.bson
