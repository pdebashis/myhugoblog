---
title: "Monitoring Hadoop"
date: 2021-06-12T17:34:37+05:30
---

A Hadoop ecosystem requires vigorous monitoring to identify any abnormality and prevent major downtimes. Major Hadoop service providers, provide their own tools for monitoring. But for a standalone hadoop, the monitoring aspect is an open field, and you can use your own tools and scripts to setup a monitoring framework.

Here, lets try to setup a basic script based monitoring capability with essential indicators

## Uptime Monitoring

Identify each process type running on each cluster. For a more detailed output, read the uptime from ps output.

    PID1=`pgrep -f "proc_datanode"`
    PID2=`pgrep -f "\<namenode\>"`
    PID3=`pgrep -f "\<JobHistoryServer\>"`
    PID4=`pgrep -f "\<DFSZKFailoverController\>"`
    PID5=`pgrep -f "\<NodeManager\>"`
    PID6=`pgrep -f "\<ResourceManager\>"`
    PID7=`pgrep -f "\<JournalNode\>"`

For each PID, the check will look something like this - 

    if [ -z $PID ]
    then
        IS_ALL_OK=1
        printf "%15s [ %5s ]\n" "Data Node" "ALERT"
    else
        printf "%15s [ %5s ]\n" "Data Node" "OK"
    fi

## Cluster Status

Namenode status

    hdfs haadmin -getAllServiceState
Resouce Manager status

    yarn rmadmin -getAllServiceState

## Resource Utilization

HDFS disk utilisations 

    hdfs dfs -df -h
Live data nodes

    grab_report=`$HADOOP_PREFIX/bin/hdfs dfsadmin -report 2>/dev/null`
    hosts=$(echo -n "$grab_report" | grep Name)
    while IFS= read -r i;do
      host=$(echo $i | awk -F'[()]' '{print $2}')
      totaldisk=`echo -n "$grab_report" | grep "$i" -A 7 | grep "Configured Capacity:" | awk -F'[()]' '{print $2}'`
      useddisk=`echo -n "$grab_report" | grep "$i" -A 7 | grep "^DFS Used:" | awk -F'[()]' '{print $2}'`
      availdisk=`echo -n "$grab_report" | grep "$i" -A 7 | grep "DFS Remaining:" | awk -F'[()]' '{print $2}'`
      perc=`echo -n "$grab_report" | grep "$i" -A 7 | grep "DFS Used%:" | awk -F':' '{print $2}'`
    echo "DATANODE,$host,$totaldisk,$useddisk,$availdisk,$percdisk"
    done <<< "$hosts"
    
Yarn memory usage

    http:/hostname:8088
    
Network Usage

    tcpdump,nmap,wireshark amongst others
    
CPU & Memory Usage

    top


## Hbase Monitoring

Regions Count

    hmaster=$(curl -s http://$hbase_IP:16010/jmx)
    liveserver=$(echo -n "$hmaster" | grep "tag.liveRegionServers" | awk -F':' '{print $2}' | tr '"' ' ' )
    IFS=';' read -r -a array <<< "$liveserver"

    for i in ${array[@]}; do
      rserver=$(echo $i | cut -d ',' -f1)
      if [ ! -z $rserver ]; then
      region_count=$(curl -s http://$rserver:16030/jmx | grep regionCount | cut -d':' -f2)
      echo "REGIONS,$rserver,$region_count" >> $outputfile
      fi
    done

Regions Skew

    grab_hbase_tables=$($HADOOP_PREFIX/bin/hdfs dfs -du -h  /hbase2/data/default)
    for table in ${tables[@]}; do
      size=$(echo -n "${grab_hbase_tables}" | grep /${table}$ | awk '{print $1$2}')
      regions_table=$(curl -s -N "http://${hbase_IP}:16010/table.jsp?name=$table" | readRegions | grep "^<td>" | grep -v href | grep -o -E '[0-9]+')
      regionscount=$(echo "${regions_table}" | awk '{sum+=$0}END{print sum}')
      regionsall=$(echo "${regions_table}" | paste -s -d/ -)
      echo "HBASE,$table,$size,$regionscount,$regionsall" >> $outputfile
    done

## Inconsistencies

HDFS block inconsistencies

    hdfs fsck / | egrep -v '^\.+$' | grep -v eplica 

Hbase block inconsistencies

    hbase hbck -details

## Conclusion

All the above and similar indicators can be run on the cluster in a script to get scheduled reports in mail. You could also build grafana dashboards using most of the indicators.

