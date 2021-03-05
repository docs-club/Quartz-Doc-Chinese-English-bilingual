# Lesson 10: Configuration, Resource Usage and SchedulerFactory

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-10.html

The architecture of Quartz is modular, and therefore to get it running several components need to be `snapped` together. Fortunately, some helpers exist for making this happen.

Quartz 的架构是模块化的，因此要让它运行几个组件需要 `snapped`。幸运的是，有一些辅助工具可以实现这一点。

The major components that need to be configured before Quartz can do its work are:

在 Quartz 开始工作之前，需要配置的主要组件有：

- ThreadPool
- JobStore
- DataSources (if necessary)
- The Scheduler itself

The ThreadPool provides a set of Threads for Quartz to use when executing Jobs. The more threads in the pool, the greater number of Jobs that can run concurrently. However, too many threads may bog-down your system. Most Quartz users find that 5 or so threads are plenty- because they have fewer than 100 jobs at any given time, the jobs are not generally scheduled to run at the same time, and the jobs are short-lived (complete quickly). Other users find that they need 10, 15, 50 or even 100 threads - because they have tens-of-thousands of triggers with various schedules - which end up having an average of between 10 and 100 jobs trying to execute at any given moment. Finding the right size for your scheduler’s pool is completely dependent on what you’re using the scheduler for. There are no real rules, other than to keep the number of threads as small as possible (for the sake of your machine’s resources) - but make sure you have enough for your Jobs to fire on time. Note that if a trigger’s time to fire arrives, and there isn’t an available thread, Quartz will block (pause) until a thread comes available, then the Job will execute - some number of milliseconds later than it should have. This may even cause the thread to misfire - if there is no available thread for the duration of the scheduler’s configured `misfire threshold`.

线程池提供了一组线程供 Quartz 在执行作业时使用。池中的线程越多，可以并发运行的作业数量就越多。然而，过多的线程可能会使系统陷入困境。大多数 Quartz 用户发现，5 个左右的线程就足够了，因为它们在任何给定时间都少于 100 个作业，这些作业通常不会同时运行，而且这些作业的寿命很短（很快就会完成）。其他用户发现他们需要 10 个、15 个、50 个甚至 100 个线程（因为他们有成千上万个具有各种调度的触发器），这些线程最终在任何给定时刻平均有 10 到 100 个作业要执行。确定调度程序池的正确大小完全取决于使用调度程序的目的。除了让线程的数量尽可能少（为了节约机器的资源）之外，没有什么真正的规则，但要确保有足够的线程来按时启动作业。注意，如果触发器的触发时间到了，但没有可用的线程，Quartz 将阻塞（暂停），直到有线程可用，然后作业将执行（比应该执行的时间晚了一些毫秒）。这甚至可能导致线程失败，如果在调度程序配置的 `失效阈值` 期间没有可用的线程。

A ThreadPool interface is defined in the org.quartz.spi package, and you can create a ThreadPool implementation in any way you like. Quartz ships with a simple (but very satisfactory) thread pool named org.quartz.simpl.SimpleThreadPool. This ThreadPool simply maintains a fixed set of threads in its pool - never grows, never shrinks. But it is otherwise quite robust and is very well tested - as nearly everyone using Quartz uses this pool.

在 org.quartz.spi 包中定义了 ThreadPool 接口，可以以任何喜欢的方式创建线程池实现。Quartz 提供了一个简单（但非常令人满意）的线程池，名为 org.quartz.simpl.SimpleThreadPool。这个线程池只是在它的线程池中维护一组固定的线程（永不增长，永不收缩）。但它在其他方面相当健壮，而且经过了很好的测试，因为几乎每个使用 Quartz 的人都使用这个池。

JobStores and DataSources were discussed in Lesson 9 of this tutorial. Worth noting here, is the fact that all JobStores implement the org.quartz.spi.JobStore interface - and that if one of the bundled JobStores does not fit your needs, then you can make your own.

在本教程的第 9 课中讨论了 JobStores 和数据源。这里值得注意的是，所有的 JobStores 都实现了 org.quartz.spi.JobStore 接口，如果绑定的一个 JobStore 不适合需要，那么可以自己创建一个。

Finally, you need to create your Scheduler instance. The Scheduler itself needs to be given a name, told its RMI settings, and handed instances of a JobStore and ThreadPool. The RMI settings include whether the Scheduler should create itself as a server object for RMI (make itself available to remote connections), what host and port to use, etc.. StdSchedulerFactory (discussed below) can also produce Scheduler instances that are actually proxies (RMI stubs) to Schedulers created in remote processes.

最后，需要创建调度程序实例。需要给调度程序本身一个名称，告诉它的 RMI 设置，并传递 JobStore 和 ThreadPool 的实例。RMI 设置包括调度程序是否应该将自己创建为 RMI 的服务器对象（使自己对远程连接可用），使用什么主机和端口，等等。StdSchedulerFactory（下面将讨论）还可以生成调度程序实例，这些实例实际上是远程进程中创建的调度程序的代理（RMI 存根）。

## StdSchedulerFactory

StdSchedulerFactory is an implementation of the org.quartz.SchedulerFactory interface. It uses a set of properties (java.util.Properties) to create and initialize a Quartz Scheduler. The properties are generally stored in and loaded from a file, but can also be created by your program and handed directly to the factory. Simply calling getScheduler() on the factory will produce the scheduler, initialize it (and its ThreadPool, JobStore and DataSources), and return a handle to its public interface.

StdSchedulerFactory 是 org.quartz.SchedulerFactory 接口的一个实现。它使用一组属性（java.util.Properties）来创建和初始化 Quartz 调度程序。属性通常存储在文件中并从文件中加载，但也可以由程序创建并直接交给工厂。只需在工厂上调用 getScheduler() 就会生成调度程序，初始化它（及其 ThreadPool、JobStore 和 DataSources），并向其公共接口返回一个句柄。

There are some sample configurations (including descriptions of the properties) in the `docs/config` directory of the Quartz distribution. You can find complete documentation in the `Configuration` manual under the `Reference` section of the Quartz documentation.

Quartz 发行版的 `docs/config` 目录中有一些配置示例（包括属性描述）。可以在 Quartz 文档的 `参考` 部分的 `配置` 手册中找到完整的文档。

## DirectSchedulerFactory

DirectSchedulerFactory is another SchedulerFactory implementation. It is useful to those wishing to create their Scheduler instance in a more programmatic way. Its use is generally discouraged for the following reasons: (1) it requires the user to have a greater understanding of what they’re doing, and (2) it does not allow for declarative configuration - or in other words, you end up hard-coding all of the scheduler’s settings.

DirectSchedulerFactory 是另一个 SchedulerFactory 实现。对于那些希望以更程序化的方式创建调度程序实例的人来说，它很有用。通常不鼓励使用它，原因如下：1、它要求用户对他们正在做的事情有很好的理解；2、它不允许声明式配置。换句话说，最终要硬编码所有调度程序的设置。

## Logging

Quartz uses the SLF4J framework for all of its logging needs. In order to `tune` the logging settings (such as the amount of output, and where the output goes), you need to understand the SLF4J framework, which is beyond the scope of this document.

Quartz 使用 SLF4J 框架满足其所有日志记录需求。为了 `调优` 日志记录设置（比如输出的数量和输出的位置），需要理解 SLF4J 框架，这超出了本文的范围。

If you want to capture extra information about trigger firings and job executions, you may be interested in enabling the org.quartz.plugins.history.LoggingJobHistoryPlugin and/or org.quartz.plugins.history.LoggingTriggerHistoryPlugin.

如果想获取关于触发器触发和作业执行的额外信息，可能会对启用 org.quartz.plugins.history.LoggingJobHistoryPlugin 和/或 org.quartz.plugins.history.LoggingTriggerHistoryPlugin 感兴趣。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 9: JobStores](/Tutorials/Lesson-9)**
- **Next Lesson（下一课）：[Lesson 11: Advanced (Enterprise) Features](/Tutorials/Lesson-11)**