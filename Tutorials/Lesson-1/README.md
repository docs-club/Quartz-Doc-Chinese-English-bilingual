# Lesson 1: Using Quartz

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-01.html

Before you can use the scheduler, it needs to be instantiated (who’d have guessed?). To do this, you use a SchedulerFactory. Some users of Quartz may keep an instance of a factory in a JNDI store, others may find it just as easy (or easier) to instantiate and use a factory instance directly (such as in the example below).

在使用调度程序之前，你需要使用 SchedulerFactory 对它进行实例化（谁能猜到呢？）。一些 Quartz 用户可能在 JNDI 存储中保存工厂实例，其他用户可能会发现直接实例化与使用工厂实例同样容易（或更容易），如下面的示例。

Once a scheduler is instantiated, it can be started, placed in stand-by mode, and shutdown. Note that once a scheduler is shutdown, it cannot be restarted without being re-instantiated. Triggers do not fire (jobs do not execute) until the scheduler has been started, nor while it is in the paused state.

一旦调度程序被实例化，就可以启动它、将其置于待机模式或关闭它。注意，一旦调度程序关闭，就只能重新实例化来启动。在调度程序启动之前，或者处于暂停状态时，触发器不会触发（作业不会执行）。

Here’s a quick snippet of code, that instantiates and starts a scheduler, and schedules a job for execution:

下面是一个代码片段，它实例化并启动一个调度程序，并调度一个作业来执行：

```java
SchedulerFactory schedFact = new org.quartz.impl.StdSchedulerFactory();

Scheduler sched = schedFact.getScheduler();

sched.start();

// define the job and tie it to our HelloJob class
JobDetail job = newJob(HelloJob.class)
    .withIdentity("myJob", "group1")
    .build();

// Trigger the job to run now, and then every 40 seconds
Trigger trigger = newTrigger()
    .withIdentity("myTrigger", "group1")
    .startNow()
    .withSchedule(simpleSchedule()
        .withIntervalInSeconds(40)
        .repeatForever())
    .build();

// Tell quartz to schedule the job using our trigger
sched.scheduleJob(job, trigger);
```

As you can see, working with quartz is rather simple. In Lesson 2 we’ll give a quick overview of Jobs and Triggers, and Quartz’s API so that you can more fully understand this example.

如你所见，使用 quartz 相当简单。在第二课中，我们将快速概述作业和触发器，以及 Quartz 的 API，以便让你可以更充分地理解这个示例。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Next Lesson（下一课）：[Lesson 2: The Quartz API, and Introduction to Jobs And Triggers](/Tutorials/Lesson-2)**
