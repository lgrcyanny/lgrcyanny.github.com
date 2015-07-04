title: Hadoop MapReduce 1 Framework
tags:
  - Hadoop
  - Learning
  - Research
id: 538
categories:
  - Hadoop
date: 2013-12-05 16:11:12
---

For MapReduce programming, a developer can run a MapReduce job by simply calling submit() or waitForCompletion() on a job object. This method abstracts the job processing details away from developer. But there is a great of job processing behind the scene that we will consider in this section.
Hadoop 2.x has released new MapReduce framework implementation called YARN or MapReduce 2, for traditional MapReduce is the classic framework which is also called MapReduce 1\. YARN is compatible with MapReduce 1\. [more...]

### MapReduce 1 high level overview

[![hd6.2](http://cyanny/myblog/wp-content/uploads/2013/11/hd6.2.png)Figure1 Classic MapReduce Framework](http://cyanny/myblog/wp-content/uploads/2013/11/hd6.2.png)
As shown in Figure 1, there are four independent entities in the framework:
-  Client, which submits the MapReduce Job
-  JobTracker, which coordinates and controls the job run. It is a Java class called JobTracker.
-  TaskerTrackers, which run the task that is split job, control the specific map or reduce task, and make reports to JobTracker. They are Java class as well.
-  HDFS, which provides distributed data storage and is used to share job files between other entities.

As the Figure 1 show, a MapReduce processing including 10 steps, and in short, that is:
-  The clients submit MapReduce jobs to the JobTracker. 
-  The JobTracker assigns Map and Reduce tasks to other nodes in the cluser
-  These nodes each run a software daemon TaskTracker on separate JVM.
-  Each TaskTracker actually initiates the Map or Reduce tasks and reports progress back to the JobTracker

### Job Submission

When the client call submit() on job object. An internal JobSubmmitter Java Object is initiated and submitJobInternal() is called. If the clients calls the waiForCompletion(), the job progresss will begin and it will response to the client with process results to clients until the job completion.
JobSubmmiter do the following work:
-  Ask the JobTracker for a new job ID.
-  Checks the output specification of the job.
-  Computes the input splits for the job.
-  Copy the resources needed to run the job. Resources include the job jar file, the configuration file and the computed input splits. These resources will be copied to HDFS in a directory named after the job id. The job jar will be copied more than 3 times across the cluster so that TaskTrackers can access it quickly.
-  Tell the JobTracker that the job is ready for execution by calling submitJob() on JobTracker.

### Job Initialization

When the JobTracker receives the call submitJob(), it will put the call into an internal queue from where the job scheduler will pick it up and initialize it. The initialization is done as follow:
-  An job object is created to represent the job being run. It encapsulates its tasks and bookkeeping information so as to keep track the task progress and status.
-  Retrieves the input splits from HDFS and create the list of tasks, each of which has task ID. JobTracker creates one map task for each split, and the number of reduce tasks according to configuration.
-  JobTracker will create the setup task and cleanup task. Setup task is to create the final output directory for the job and the temporary working space for the task output. Cleanup task is to delete the temporary working space for the task ouput.
-  JobTracker will assign tasks to free TaskTrackers

### Task Assignment

TaskTrackers send heartbeat periodically to JobTracker Node to tell it if it is alive or ready to get a new task. The JobTracker will allocate a new task to the ready TaskTracker. Task assignment is as follows:
-  The JobTracker will choose a job to select the task from according to scheduling algorithm, a simple way is chosen on a priority list of job. After chose the job, the JobTracker will choose a task from the job.
-  TaskTrackers has a fixed number of slots for map tasks and for reduces tasks which are set independently, the scheduler will fits the empty map task slots before reduce task slots.
-  To choose a reduce task, the JobTracker simply takes next in its list of yet-to-be-run reduce task, because there is no data locality consideration. But map task chosen depends on the data locality and TaskTracker’s network location.

### Task Execution

When the TaskTracker has been assigned a task. The task execution will be run as follows:
-  Copy jar file from HDFS, copy needed files from the distributed cache on the local disk.
-  Creates a local working directory for the task and ‘un-jars’ the jar file contents to the direcoty
-  Creates a TaskRunner to run the task. The TaskRunner will lauch a new JVM to run each task.. TaskRunner fails by bugs will not affect TaskTracker. And multiple tasks on the node can reuse the JVM created by TaskRunner.
-  Each task on the same JVM created by TaskRunner will run setup task and cleanup task.
-  The child process created by TaskRunner will informs the parent process of the task’s progress every few seconds until the task is complete.

### Progress and Status Updates

[![hd6.3](http://cyanny/myblog/wp-content/uploads/2013/11/hd6.3.png)Figure 2 Classic MapReduce Framework Progress and Status Updates](http://cyanny/myblog/wp-content/uploads/2013/11/hd6.3.png)
After clients submit a job. The MapReduce job is a long time batching job. Hence the job progress report is important. What consists of the Hadoop task progress is as follows:
-  Reading an input record in a mapper or reducer
-  Writing an output record in a mapper or a reducer
-  Setting the status description on a reporter, using the Reporter’s setStatus() method
-  Incrementing a counter
-  Calling Reporter’s progress()

**As shown in Figure 2, when a task is running, the TaskTracker will notify the JobTracker its task progress by heartbeat every 5 seconds.**

And mapper and reducer on the child JVM will report to TaskTracker with it’s progress status every few seconds. The mapper or reducers will set a flag to indicate the status change that should be sent to the TaskTracker. The flag is checked in a separated thread every 3 seconds. If the flag sets, it will notify the TaskTracker of current task status.
The JobTracker combines all of the updates to produce a global view, and the Client can use getStatus() to get the job progress status.

### Job Completion

When the JobTracker receives a report that the last task for a job is complete, it will change its status to successful. Then the JobTracker will send a HTTP notification to the client which calls the waitForCompletion(). The job statistics and the counter information will be printed to the client console. Finally the JobTracker and the TaskTracker will do clean up action for the job.

Resources:
[Part 0 Hadoop Overview](http://cyanny/myblog/2013/12/05/hadoop-overview/ "Hadoop Overview")
[Part 1 Hadoop HDFS Review](http://cyanny/myblog/2013/12/05/hadoop-hdfs-review/ "Hadoop HDFS Review")
[Part 2 Hadoop HDFS Federation](http://cyanny/myblog/2013/12/05/hadoop-hdfs-federation/ "Hadoop HDFS Federation")
[Part 3 Hadoop HDFS High Availability(HA)](http://cyanny/myblog/2013/12/05/hadoop-hdfs-high-availability/ "Hadoop HDFS High Availability(HA)")
[Part 4 Hadoop MapReduce Overview](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-overview/ "Hadoop MapReduce Overview")
[Part 5 Hadoop MapReduce 1 Framework](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-1-framework/ "Hadoop MapReduce 1 Framework")
[Part 6 Hadoop MapReduce 2 (YARN)](http://cyanny/myblog/2013/12/05/hadoop-mapreduce-2-yarn/ "Hadoop MapReduce 2 (YARN)")
[Part 7 Hadoop isn’t Silver Bullet](http://cyanny/myblog/2013/12/05/hadoop-isnt-silver-bullet/ "Hadoop isn’t Silver Bullet")