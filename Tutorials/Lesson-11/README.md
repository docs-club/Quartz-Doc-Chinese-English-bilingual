# Lesson 11: Advanced (Enterprise) Features

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-11.html

## Clustering

Clustering currently works with the JDBC-Jobstore (JobStoreTX or JobStoreCMT) and the TerracottaJobStore. Features include load-balancing and job fail-over (if the JobDetail’s “request recovery” flag is set to true).

集群当前使用 JDBC-Jobstore（JobStoreTX 或 JobStoreCMT）和 TerracottaJobStore。特性包括负载平衡和作业故障转移（如果 JobDetail 的“请求恢复”标志设置为 true）。

####Clustering With JobStoreTX or JobStoreCMT Enable clustering by setting the “org.quartz.jobStore.isClustered” property to “true”. Each instance in the cluster should use the same copy of the quartz.properties file. Exceptions of this would be to use properties files that are identical, with the following allowable exceptions: Different thread pool size, and different value for the “org.quartz.scheduler.instanceId” property. Each node in the cluster MUST have a unique instanceId, which is easily done (without needing different properties files) by placing “AUTO” as the value of this property.

通过设置 org.quartz.jobStore.isClustered 来启用集群。属性为 true。集群中的每个实例都应该使用相同的 quartz 副本。属性文件。例外情况是使用相同的属性文件，但允许有以下例外情况:不同的线程池大小，以及 org.quartz.scheduler.instanceId 属性的不同值。集群中的每个节点都必须有一个唯一的 instanceId，这很容易做到（不需要不同的属性文件），只需将 AUTO 作为该属性的值。

> Never run clustering on separate machines, unless their clocks are synchronized using some form of time-sync service (daemon) that runs very regularly (the clocks must be within a second of each other). See http://www.boulder.nist.gov/timefreq/service/its.htm if you are unfamiliar with how to do this.

永远不要在单独的机器上运行集群，除非它们的时钟是使用某种形式的定时运行的时间同步服务（守护进程）来同步的（时钟之间的间隔必须在一秒之内）。如果不熟悉如何做到这一点，请参阅 http://www.boulder.nist.gov/timefreq/service/its.htm。

> Never fire-up a non-clustered instance against the same set of tables that any other instance is running against. You may get serious data corruption, and will definitely experience erratic behavior.

永远不要针对正在运行的其他实例所针对的同一组表启动非集群实例。这么做可能会遇到严重的数据损坏，并且肯定会出现不稳定的行为。

Only one node will fire the job for each firing. What I mean by that is, if the job has a repeating trigger that tells it to fire every 10 seconds, then at 12:00:00 exactly one node will run the job, and at 12:00:10 exactly one node will run the job, etc. It won’t necessarily be the same node each time - it will more or less be random which node runs it. The load balancing mechanism is near-random for busy schedulers (lots of triggers) but favors the same node that just was just active for non-busy (e.g. one or two triggers) schedulers.

每次触发作业时，只有一个节点触发作业。我的意思是，如果作业有一个告诉它每 10 秒触发一次的重复触发器，那么在 12:00:00 正好有一个节点运行作业，在 12:00:10 正好有一个节点运行作业，以此类推。不一定每次都是同一个节点，运行它的节点或多或少都是随机的。对于繁忙的调度器（有很多触发器），负载平衡机制几乎是随机的，但是对于不繁忙的调度器（比如一个或两个触发器），负载平衡机制更倾向于激活同一个节点。

####Clustering With TerracottaJobStore Simply configure the scheduler to use TerracottaJobStore (covered in Lesson 9: JobStores), and your scheduler will be all set for clustering.

只需配置调度程序来使用 TerracottaJobStore（在第 9 课：JobStores 中涉及），调度程序就会被设置为集群。

You may also want to consider implications of how you setup your Terracotta server, particularly configuration options that turn on features such as persistence, and running an array of Terracotta servers for HA.

可能还需要考虑如何设置 Terracotta 服务器，特别是启用持久性等特性的配置选项，以及运行一系列 Terracotta 服务器以实现 HA。

The Enterprise Edition of TerracottaJobStore provides advanced Quartz Where features, that allow for intelligent targeting of jobs to appropriate cluster nodes.

TerracottaJobStore 的企业版提供了高级的 Quartz Where 特性，允许将作业智能定位到适当的集群节点。

More information about this JobStore and Terracotta can be found at http://www.terracotta.org/quartz

关于这个 JobStore 和 Terracotta 的更多信息详见 http://www.terracotta.org/quartz

## JTA Transactions

As explained in Lesson 9: JobStores, JobStoreCMT allows Quartz scheduling operations to be performed within larger JTA transactions.

正如第 9 课：JobStores 中所解释的，JobStoreCMT 允许在较大的 JTA 事务中执行 Quartz 调度操作。

Jobs can also execute within a JTA transaction (UserTransaction) by setting the “org.quartz.scheduler.wrapJobExecutionInUserTransaction” property to “true”. With this option set, a a JTA transaction will begin() just before the Job’s execute method is called, and commit() just after the call to execute terminates. This applies to all jobs.

通过将 org.quartz.scheduler.wrapJobExecutionInUserTransaction 属性设置为 true，作业也可以在 JTA 事务（UserTransaction）中执行。通过设置这个选项，JTA 事务将在调用作业的 execute 方法之前执行 begin()，而在终止调用 execute 之后执行 commit()。这适用于所有的工作。

If you would like to indicate per job whether a JTA transaction should wrap its execution, then you should use the @ExecuteInJTATransaction annotation on the job class.

如果想要指示每个作业是否应该包装 JTA 事务的执行，那么应该在作业类上使用 @ExecuteInJTATransaction 注解。

Aside from Quartz automatically wrapping Job executions in JTA transactions, calls you make on the Scheduler interface also participate in transactions when using JobStoreCMT. Just make sure you’ve started a transaction before calling a method on the scheduler. You can do this either directly, through the use of UserTransaction, or by putting your code that uses the scheduler within a SessionBean that uses container managed transactions.

除了在 JTA 事务中自动包装作业执行之外，在使用 JobStoreCMT 时，调度程序接口上的调用也会参与事务。只需确保在调用调度程序上的方法之前已经启动了事务。可以通过使用 UserTransaction 直接做到这一点，也可以将使用调度器的代码放在使用容器管理事务的 SessionBean 中。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 10: Configuration, Resource Usage and SchedulerFactory](/Tutorials/Lesson-10)**
- **Next Lesson（下一课）：[Lesson 12: Miscellaneous Features](/Tutorials/Lesson-12)**