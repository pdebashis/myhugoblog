---
title: "Quick Notes : Hadoop"
date: 2021-06-12T17:34:37+05:30
---
# Introduction

Collection of opensource software utilities that facilitate using a network of many computers to solve problems involving massive amounts of data and computation. Majorly consisting of a filesystem, atop the file systems comes the MapReduce Engine, an Hbase database and other applications.

**Namenode** maintains file system namespace. It knows the list of directories and files. It also manages file to block mapping. 

**Journal Node** is a lightweight solution to store namenode edit logs 

**YARN** is a layer that came in between mapreduce and hadoop for the better management of resources and jobs. Yarn was introduced to Hadoop ecosystem on 2.x 

# Startup scripts

**start-all.sh** & **stop-all.sh** : Used to start and stop hadoop daemons all at once. Issuing it on the master machine will start/stop the daemons on all the nodes of a cluster. Deprecated as you have already noticed. 

**start-dfs.sh**, **stop-dfs.sh** and **start-yarn.sh**, **stop-yarn.sh** : Same as above but start/stop HDFS and YARN daemons separately on all the nodes from the master machine. It is advisable to use these commands now over start-all.sh & stop-all.sh 

**hadoop-daemon.sh start datanode** : Start the data node daemon only on a machine

**Start-mapred.sh** : on the machine you plan to run the Jobtracker on. This will bring up the Map/Reduce cluster with Jobtracker running on the machine you ran the command on and Tasktrackers running on machines listed in the slaves file. 

**hadoop-daemon.sh start zkfc** : Used to bring up the HA manager service in namenode and secondary namenode servers.

# Major Ports 
:50070/dfshealth.html#tab-overview

:16888

# Major configs 
Core-site.xml
hdfs-site.xml
mapred-site.xml
yarn-site.xml
slaves 

# Major Metrics

HDFS disk utilisations (hdfs dfs -df -h)
Live data nodes
Namenode status (hdfs haadmin -getAllServiceState)
Resouce Manager status  (yarn rmadmin -getAllServiceState)
Memory Usage
Network Usage
CPU Usage
Cluster Load
Namenode Heap utilization
Yarn Heap utilization
Yarn memory usage

# Fault Tolerance
Replication of files are kept on a separate data node. So if one data node fails, all files are still accessible. Hadoop ensures that the replication is stored on different machines. 

Re-replication is done by Namenode whenever it discovers a failed data node 

Rack Awareness is available to ensure at least one copy is available on a separate rack.  

Storage Policies introduced in 2.x 

Erasure coding introduced in 3.x 

# Namenode Failure
Backup of HDFS Namespace information is done to QJM setup. Namespace information consists of 
In memory FS Image : Entire file system stored in memory by Namenode (dfs.name.dir/hadoop.tmp.dir) 
Edit Log : Every change in filesystem, used to reconstruct the in-memory FSImage (dfs.name.edits.dir) 

20/08/12 03:29:24 WARN namenode.FSNamesystem: Only one namespace edits storage directory (dfs.namenode.edits.dir) configured. Beware of data loss due to lack of redundant storage directories! 

In a QJM setup (Qourum Journal Manager), a Journal Node is installed on atleast 3 machines on the cluster. Namenode write the edit log to QJM instead of local filesystem. 

Stand by name node is a namenode which continously reads from the journal nodes and is ready to take over in seconds. All data nodes, send the block report heartbeat to both Namenodes 


An active Zookeeper and 2 failover controllers help switch between the namenodes. The failover controller on the active namenode maintains a lock on the zookeper, Standby namenode keeps trying to get the lock.  

Secondary Name node is deployed to perform a check point activity every hour on the edit logs and save a copy of in memory FSImage and truncate the edit logs. (dfs.namenode.checkpoint.dir) 

Secondary namenode is not required when there is a standby name node, because stand by name node also performs this checkpoint activity 

# Inconsistencies and Recovery

To understand the scope of the problem 

    hdfs fsck / | egrep -v '^\.+$' | grep -v eplica 

Look through the output for missing or corrupt blocks (ignore under-replicated blocks for now). This command is really verbose especially on a large HDFS filesystem so I normally get down to the meaningful output with 

Use that output to determine where blocks might live. If the file is larger than your block size it might have multiple blocks. 

You can use the reported block numbers to go around to the datanodes and the namenode logs searching for the machine or machines on which the blocks lived. Try looking for filesystem errors on those machines. Missing mount points, datanode not running, file system reformatted/reprovisioned. If you can find a problem in that way and bring the block back online that file will be healthy again. 

Lather rinse and repeat until all files are healthy or you exhaust all alternatives looking for the blocks. 

    hdfs fsck / -delete #remove corrupted files only 
    hdfs fsck / #to confirm healthy status 