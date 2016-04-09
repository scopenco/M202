# CHAPTER 1: SYSTEM SIZING AND TUNING

# Videos

* [Welcome to the class](https://www.youtube.com/watch?v=MnZrqOfei4g)
* [Intro to M202](https://www.youtube.com/watch?v=QES1Wrkldss)
* [Introduction to System Sizing and Tuning](https://www.youtube.com/watch?v=vTkip8vukGw)
* [Spinning Up a VM with Vagrant (Unix)](https://www.youtube.com/watch?v=gz9JgQk3HNM)
* [Spinning up a VM with Vagrant (Windows)](https://www.youtube.com/watch?v=8wIvr4ssXXY)
* [Memory model in MongoDB](https://www.youtube.com/watch?v=7cmY6XqiXmg)
* [Resident memory](https://www.youtube.com/watch?v=9gmkG0KQWLs)
* [Journaling's impact on resident memory](https://www.youtube.com/watch?v=8TmmEzm50cw)
* [Storage Engine: WiredTiger](https://www.youtube.com/watch?v=O9TGqK3FBX8)
* [Process restarts](https://www.youtube.com/watch?v=ivBcMSYA3zs)
* [Spinning disks](https://www.youtube.com/watch?v=hmU949bd8Gk)
* [Network storage](https://www.youtube.com/watch?v=k1MprP_rSu8)
* [SSDs](https://www.youtube.com/watch?v=CjGowwDdfMs)
* [RAID](https://www.youtube.com/watch?v=NlIA3PjZV6s)
* [MongoDB and NUMA hardware](https://www.youtube.com/watch?v=GGCYm34DM_k)
* [Filesystems and options](https://www.youtube.com/watch?v=rVkwokQDFzk)
* [NFS](https://www.youtube.com/watch?v=6HmCFOBuiCg)
* [Swap](https://www.youtube.com/watch?v=nwEfEp0jrI0)
* [Readahead](https://www.youtube.com/watch?v=fY3ffrvzkNQ)
* [A final word on production notes](https://www.youtube.com/watch?v=kFUC9vSO788)
* [MongoDB CPU Usage](https://www.youtube.com/watch?v=agVy719jNq4)
* [How MongoDB uses disk](https://www.youtube.com/watch?v=6gNFIvp6pDU)
* [Reclaiming disk space](https://www.youtube.com/watch?v=jX8X6FW2hOg)
* [Monitoring disk usage](https://www.youtube.com/watch?v=gCaRAIAHmQQ)
* [Segregation of resources](https://www.youtube.com/watch?v=icGs8UL1pSw)
* [Virtualization](https://www.youtube.com/watch?v=6P6SX5WnHzA)
* [Containers (zones/jails)](https://www.youtube.com/watch?v=lrvhAZ7t9qs)
* [Intro to replica set sizing](https://www.youtube.com/watch?v=opBw2UWFr6g)
* [Going beyond 3 nodes](https://www.youtube.com/watch?v=t5EUHQgIwro)
* [Geographically distributed replica sets](https://www.youtube.com/watch?v=gtZLeXYWq4w)
* [Replica set syncing demo](https://www.youtube.com/watch?v=NbvqYvCsofA)

# Homeworks
## Homework 1.1: Readahead scenarios 
### Question
Initially, your system has the following properties:
* You're working with a system where your indexes and part of your working set fit in memory
* You're not constrained by write locks
* You are using an SSD

Documents are typically sorted on one of several fields and order does not correspond to natural order, though often adjacent docs will be requested in the same query
You then change your system in the following ways, one at a time, before returning to your original state.

During which of the following changes should a higher readahead result in a larger performance increase than it would have for the initial system state?

Assume that any property of the system not mentioned in a particular choice is still in the state listed above.
### Answer
```
+ You are now using a spinning disk, rather than an SSD.
- You are now writing frequently, so that write locks become a constraint while reads have to wait.
- Your working set outgrows the available memory, so you are having to go to storage much more often.
+ You begin frequently accessing your data from capped collections, in the order in which it was written.
```

## Homework 1.2: Replica set chaining 
### Question
You are operating a geographically dispersed replica set as outlined below:

![replica set](https://raw.githubusercontent.com/wiki/scopenco/M202/images/replica_set_chaining.png)

Your network operations team has asked if you can limit the amount of outbound traffic from Site A because of some capacity issues with the traffic in and out of that site. For example, consider that this is where the majority of your users stream/download. However, due to how the application is deployed, you wish to keep your write traffic in Site A for performance, and so you have chosen not to change which server is primary.

Using replica set chaining, which of the following scenarios will minimize traffic due to MongoDB replication in and out of Site A?

### Answer
```
- P to S1; P to S2; P to S3; P to S4
- P to S1; S1 to S2; S2 to S3; S1 to S4
+ P to S1; S1 to S2; S2 to S3; S3 to S4
- P to S4; S4 to S1; S4 to S2; S4 to S3
- P to S2; S2 to S1; S2 to S3; S3 to S4
```

## Homework 1.3: Memory usage 
### Question
You are performing an aggregation query with $sort, and are hitting the maximum size limit for in-memory sort. Which of the following might resolve this problem?
### Answer
```
+ Set the "allowDiskUse" parameter to true
- Switch out your HDD for an SSD so that data can be accessed more quickly
- Move your system to another machine with a faster CPU
+ Add an index for the variable(s) you are using to sort the documents
+ If you are not already doing so, include a $match earlier in the pipeline that will reduce the
  number of documents you are sorting
```
