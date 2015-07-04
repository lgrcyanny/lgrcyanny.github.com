title: Hadoop MapReduce 2 (YARN)
tags:
  - Hadoop
  - Learning
  - Research
id: 542
categories:
  - Hadoop
date: 2013-12-05 16:57:01
---

For large clusters with more than 4000 nodes, the classic MapReduce framework hit the scalability problems.
**Therefore, a group in Yahoo began to design the next generation MapReduce in 2010, and in 2013 Hadoop 2.x releases MapReduce 2, Yet Another Resource Negotiator (YARN) to remedy the sociability shortcoming.**
The fundamental idea of YARN is to split up the two major functionalities of the JobTracker: resource management (job scheduling) and job monitoring into separate daemons. YARN has a global Resource Manager (RM) and per-application Application Master(AM).[more...]

### YARN High-level Overview

[![hd7.1](http://cyanny/myblog/wp-content/uploads/2013/11/hd7.1.png)Figure1 YARN High-level Overview](http://cyanny/myblog/wp-content/uploads/2013/11/hd7.1.png)
As shown in Figure1, the YARN involves more entities than classic MapReduce 1 :
- Client, the same as classic MapReduce which submits the MapReduce job.
- Resource Manager, which has the ultimate authority that arbitrates resources among all the applications in the cluster, it coordinates the allocation of compute resources on the cluster.
- Node Manager, which is in charge of resource containers, monitoring resource usage (cpu, memory, disk , network) on the node , and reporting to the Resource Manager.
- Application Master, which is in charge of the life cycle an application, like a MapReduce Job. It will negotiates with the Resource Manager of cluster resources---in YARN called containers. The Application Master and the MapReduce task in the containers are scheduled by the Resource Manager. And both of them are managed by the Node Manager. Application Mater is also responsible for keeping track of task progress and status. 
- HDFS, the same as classic MapReduce, for files sharing between different entities.

Resource Manager consists of two components: Scheduler and ApplicationsManager.

Scheduler is in charge of allocating resources. The resource Container incorporates elements such as memory, cup, disk, network etc. Scheduler just has the resource allocation function, has no responsible for job status monitoring. And the scheduler is pluggable, can be replaced by other scheduler plugin-in.

The ApplicationsManager is responsible for accepting job-submissions, negotiating the first container for executing the application specific Application Master, and it provides restart service when the container fails. 
The MapReduce job is just one type of application in YARN. Different application can run on the same cluster with YARN framework. That’s the beauty of YARN.

### YARN MapReduce

[![hd7.2](http://cyanny/myblog/wp-content/uploads/2013/11/hd7.2.png)Figure2 MapReduce with YARN](http://cyanny/myblog/wp-content/uploads/2013/11/hd7.2.png)
As shown in Figure2, it is the MapReduce process with YARN, there are 11 steps, and we will explain it in 6 steps the same as the MapReduce 1 framework. They are Job Submission, Job Initialization, Task Assignment, Task Execution, Progress and Status Updates, and Job Completion.

#### Job Submission

Clients can submit jobs with the same API as MapReduce 1 in YARN. YARN implements its ClientProtocol, the submission process is similar to MapReduce 1.
-  The client calls the submit() method, which will initiate the JobSubmmitter object and call submitJobInternel().
-  Resource Manager will allocate a new application ID and response it to client.
-  The job client checks the output specification of the job
-  The job client computes the input splits
-  The job client copies resources, including the splits data, configuration information, the job JAR into HDFS
-  Finally, the job client notify Resource Manager it is ready by calling submitApplication() on the Resource Manager.

#### Job Initialization

 When the Resource Manager(RM) receives the call submitApplication(), RM will hands off the job to its scheduler. The job initialization is as follows:
-  The scheduler allocates a resource container for the job, 
-  The RM launches the Application Master under the Node Manager’s management. 
-  Application Master initialize the job. Application Master is a Java class named MRAppMaster, which initializes the job by creating a number of bookkeeping objects to keep track of the job progress. It will receive the progress and the completion reports from the tasks. 
-  Application Master retrieves the input splits from HDFS, and creates a map task object for each split. It will create a number of reduce task objects determined by the mapreduce.job.reduces configuration property.
-  Application Master then decides how to run the job. 

For small job, called uber job, which is the one has less than 10 mappers and only one reducer, or the input split size is smaller than a HDFS block, the Application Manager will run the job on its own JVM sequentially. This policy is different from MapReduce 1 which will ignore the small jobs on a single TaskTracker.

For large job, the Application Master will launches a new node with new NodeManager and new container, in which run the task. This can run job in parallel and gain more performance. 

Application Master calls the job setup method to create the job’s output directory. That’s different from MapReduce 1, where the setup task is called by each task’s TaskTracker.

#### Task Assignment

When the job is very large so that it can’t be run on the same node as the Application Master. The Application Master will make request to the Resource Manager to negotiate more resource container which is in piggybacked on heartbeat calls. The task assignment is as follows:
-  The Application Master make request to the Resource Manager in heartbeat call. The request includes the data locality information, like hosts and corresponding racks that the input splits resides on. 
-  The Recourse Manager hand over the request to the Scheduler. The Scheduler makes decisions based on these information. It attempts to place the task as close the data as possible. The data-local nodes is great, if this is not possible , the rack-local the preferred to nolocal node.
-  The request also specific the memory requirements, which is between the minimum allocation (1GB by default) and the maximum allocation (10GB). The Scheduler will schedule a container with multiples of 1GB memory to the task, based on the mapreduce.map.memory.mb and mapreduce.reduce.memory.mb property set by the task.

This way is more flexible than MapReduce 1\. In MapReduce 1, the TaskTrackers have a fixed number of slots and each task runs in a slot. Each slot has fixed memory allowance which results in two problems. For small task, it will waste of memory, and for large task which need more memeory, it will lack of memory.
In YARN, the memory allocation is more fine-grained, which is also the beauty of YARE resides in.

#### Task Execution

After the task has been assigned the container by the Resource Manger’s scheduler, the Application Master will contact the NodeManger which will launch the task JVM.
The task execution is as follows:
-  The Java Application whose class name is YarnChild localizes the resources that the task needs. YarnChild retrieves job resources including the job jar, configuration file, and any needed files from the HDFS and the distributed cache on the local disk.
-  YarnChild run the map or the reduce task
Each YarnChild runs on a dedicated JVM, which isolates user code from the long running system daemons like NodeManager and the Application Master. Different from MapReduce 1, **YARN doesn’t support JVM reuse**, hence each task must run on new JVM.
The streaming and the pipeline processs and communication in the same as MapReduce 1.

#### Progress and Status Updates

When the job is running under YARN, the mapper or reducer will report its status and progress to its Application Master every 3 seconds over the umbilical interface. The Application Master will aggregate these status reports into a view of the task status and progress. While in MapReduce 1, the TaskTracker reports status to JobTracker which is responsible for aggregating status into a global view.
Moreover, the Node Manger will send heartbeats to the Resource Manager every few seconds. The Node Manager will monitoring the Application Master and the recourse container usage like cpu, memeory and network, and make reports to the Resource Manager. When the Node Manager fails and stops heartbeat the Resource Manager, the Resource Manager will remove the node from its available resource nodes pool.
The client pulls the status by calling getStatus() every 1 second to receive the progress updates, which are printed on the user console. User can also check the status from the web UI. The Resource Manager web UI will display all the running applications with links to the web UI where displays task status and progress in detail.
[![hd7.3](http://cyanny/myblog/wp-content/uploads/2013/11/hd7.3.png)Figure 3  YARN Progress and Status Updates](http://cyanny/myblog/wp-content/uploads/2013/11/hd7.3.png)

#### Job Completion

Every 5 second the client will check the job completion over the HTTP ClientProtocol by calling waitForCompletion(). When the job is done, the Application Master and the task containers clean up their working state and the outputCommitter’s job cleanup method is called. And the job information is archived as history for later interrogation by user.

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")