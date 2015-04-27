---
title: "Configuring a Cluster for Different Workloads"
parent: "Configuration Options"
---
In this release of Drill, to configure a Drill cluster for different workloads, you re-allocate memory resources only. To re-allocate memory for services, establish baselines for performance testing, make changes in small increments, test and compare the effects of the change. 

Warden allocates resources for MapR Hadoop services, such as Zookeeper or NFS, associated with roles installed on the node. You modify `warden.conf` to manage the memory allocation. For example, you might modify these default settings:

    service.command.nfs.heapsize.percent=3
    service.command.nfs.heapsize.min=64
    service.command.nfs.heapsize.max=64
    . . .
    service.command.zk.heapsize.percent=1
    service.command.zk.heapsize.max=1500
    service.command.zk.heapsize.min=256

MapR shares memory and disk among services, such as Drill and Impala that are not associated with roles on the cluster. You manage the memory for these services in multiple files:

* The os heap settings in `warden.conf`
* Configuration files of the particular service, such as [Drill](#drill-memory-configuration), [Impala](#impala-memory-configuation), and [#jobtracker-memory-configuration] configuration files

The names of os heap settings are:

    service.command.os.heapsize.percent
    service.command.os.heapsize.max
    service.command.os.heapsize.min

## Drill Memory Configuration
You can configure the amount of direct memory allocated to a Drillbit for
query processing. The default limit is 8G, but Drill prefers 16G or more
depending on the workload. The total amount of direct memory that a Drillbit
allocates to query operations cannot exceed the limit set.

Drill mainly uses Java direct memory and performs well when executing
operations in memory instead of storing the operations on disk. Drill does not
write to disk unless absolutely necessary, unlike MapReduce where everything
is written to disk during each phase of a job.

The JVM heap memory does not limit the amount of direct memory available in
a Drillbit. The on-heap memory for Drill is only about 4-8G, which should
suffice because Drill avoids having data sit in heap memory.

You can modify memory for each Drillbit node in your cluster. To modify the
memory for a Drillbit, edit the `XX:MaxDirectMemorySize` parameter in the
Drillbit startup script located in `<drill_installation_directory>/conf/drill-
env.sh`.

{% include startnote.html %}If this parameter is not set, the limit depends on the amount of available system memory.{% include endnote.html %}

After you edit `<drill_installation_directory>/conf/drill-env.sh`, [restart
the Drillbit
]({{ site.baseurl }}/docs/starting-stopping-drill#starting-a-drillbit)on
the node.

## Impala Memory Configuration

The configuration service for the Impala service is in the Impala env.sh file.

## JobTracker Memory Configuration

Memory allocated for JobTracker in `warden.conf` is used only to calculate total memory required for services to run. The -Xmx JobTracker itself is not set, allowing memory on JobTracker to grow as needed. If you want to set an upper limit on memory, set the HADOOP_HEAPSIZE env. variable in `/opt/mapr/hadoop/hadoop-0.20.2/conf/hadoop-env.sh`.