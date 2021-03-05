# Lesson 2: The Quartz API, Jobs And Triggers

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-02.html

## The Quartz API

Quartz 的 API

The key interfaces of the Quartz API are:

Quartz API 的主要接口有如下这些：

- Scheduler - the main API for interacting with the scheduler.

用于与调度程序交互的主 API。

- Job - an interface to be implemented by components that you wish to have executed by the scheduler.

由组件实现的接口，组件定义了要执行的内容。

- JobDetail - used to define instances of Jobs.

用于定义作业实例。

- Trigger - a component that defines the schedule upon which a given Job will be executed.

定义给定作业执行时间表的组件。

- JobBuilder - used to define/build JobDetail instances, which define instances of Jobs.

用于定义/构建定义作业实例的 JobDetail 实例。

- TriggerBuilder - used to define/build Trigger instances.

用于定义/构建触发器实例

A Scheduler’s life-cycle is bounded by it’s creation, via a SchedulerFactory and a call to its shutdown() method. Once created the Scheduler interface can be used add, remove, and list Jobs and Triggers, and perform other scheduling-related operations (such as pausing a trigger). However, the Scheduler will not actually act on any triggers (execute jobs) until it has been started with the start() method, as shown in Lesson 1.

调度程序的生命周期从 SchedulerFactory 创建起直到 shutdown() 方法的调用而止。一旦创建了 Scheduler 接口，就可以使用添加、删除和列出作业和触发器，并执行其他与调度相关的操作（如暂停触发器）。但是，在使用 start() 方法启动调度程序之前，调度程序不会实际操作任何触发器（执行作业），如第一课中所示。

Quartz provides `builder` classes that define a Domain Specific Language (or DSL, also sometimes referred to as a `fluent interface`). In the previous lesson you saw an example of it, which we present a portion of here again:

Quartz 提供了定义 Domain Specific Language（或 DSL，有时也称为 `流畅接口`）的 `构建器` 类。在上一节课中，你看到了一个例子，我们在这里再次展示：

```java
// define the job and tie it to our HelloJob class
JobDetail job = newJob(HelloJob.class)
    .withIdentity("myJob", "group1") // name "myJob", group "group1"
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

The block of code that builds the job definition is using methods that were statically imported from the JobBuilder class. Likewise, the block of code that builds the trigger is using methods imported from the TriggerBuilder class - as well as from the SimpleScheduleBulder class.

作业定义的代码块使用从 JobBuilder 导入的静态方法构建。同样，触发器的代码块使用从 TriggerBuilder 类和 SimpleScheduleBulder 类导入的方法构建。

The static imports of the DSL can be achieved through import statements such as these:

DSL 的静态导入可以通过如下的 import 语句来实现：

```java
import static org.quartz.JobBuilder.*;
import static org.quartz.SimpleScheduleBuilder.*;
import static org.quartz.CronScheduleBuilder.*;
import static org.quartz.CalendarIntervalScheduleBuilder.*;
import static org.quartz.TriggerBuilder.*;
import static org.quartz.DateBuilder.*;
```

The various `ScheduleBuilder` classes have methods relating to defining different types of schedules.

`ScheduleBuilder` 类的不同表现形式具有定义不同类型的时间表相关的方法。

The DateBuilder class contains various methods for easily constructing java.util.Date instances for particular points in time (such as a date that represents the next even hour - or in other words 10:00:00 if it is currently 9:43:27).

DateBuilder 类包含用 java.util.Date 实例构造特定时间点的各种方法。例如表示下一个偶数小时的日期，换句话说，如果当前是 9:43:27，则转化为 10:00:00

## Jobs and Triggers

作业和触发器

A Job is a class that implements the Job interface, which has only one simple method:

Job 是实现 Job 接口的类，该接口只有一个简单的方法：

## The Job Interface

Job 接口

```java
package org.quartz;

public interface Job {

public void execute(JobExecutionContext context) throws JobExecutionException;
}
```

When the Job’s trigger fires (more on that in a moment), the execute(..) method is invoked by one of the scheduler’s worker threads. The JobExecutionContext object that is passed to this method provides the job instance with information about its `run-time` environment - a handle to the Scheduler that executed it, a handle to the Trigger that triggered the execution, the job’s JobDetail object, and a few other items.

当作业的触发器触发时（稍后将详细介绍），`execute(..)` 方法将由调度程序的一个工作线程调用。传递给该方法的 JobExecutionContext 对象为作业实例提供了有关其 `运行时` 环境的信息（执行它的调度程序的句柄、触发执行的触发器的句柄、作业的 JobDetail 对象和一些其他事项）。

The JobDetail object is created by the Quartz client (your program) at the time the Job is added to the scheduler. It contains various property settings for the Job, as well as a JobDataMap, which can be used to store state information for a given instance of your job class. It is essentially the definition of the job instance, and is discussed in further detail in the next lesson.

JobDetail 对象是在将作业添加到调度程序时由 Quartz 客户端（你的程序）创建的。它包含作业的各种属性设置，以及一个 JobDataMap，它可用于存储作业类的给定实例的状态信息。它本质上是作业实例的定义，并将在下一课中进行更详细的讨论。

Trigger objects are used to trigger the execution (or ‘firing’) of jobs. When you wish to schedule a job, you instantiate a trigger and ‘tune’ its properties to provide the scheduling you wish to have. Triggers may also have a JobDataMap associated with them - this is useful to passing parameters to a Job that are specific to the firings of the trigger. Quartz ships with a handful of different trigger types, but the most commonly used types are SimpleTrigger and CronTrigger.

触发器对象用于触发作业的执行（或 `触发`）。当你希望调度作业时，你可以实例化一个触发器并 `使能` 其属性，以提供你想要的调度。触发器还可能有一个与它们相关联的 JobDataMap（这对于将参数传递给特定于触发器的触发的作业非常有用）。Quartz 有几种不同的触发类型，但最常用的类型是 SimpleTrigger 和 CronTrigger

SimpleTrigger is handy if you need ‘one-shot’ execution (just single execution of a job at a given moment in time), or if you need to fire a job at a given time, and have it repeat N times, with a delay of T between executions. CronTrigger is useful if you wish to have triggering based on calendar-like schedules - such as `every Friday, at noon` or `at 10:15 on the 10th day of every month.`

如果你需要 `一次性` 执行（仅在给定时刻单个执行一个作业），或者你需要在给定时间触发一个作业，并让它重复 N 次，两次执行之间的延迟为 T，那么 SimpleTrigger 就非常方便。如果你希望根据类似日历的时间表进行触发，例如 `每周五中午` 或 `每月10日的10:15` ，CronTrigger 是非常有用的。

Why Jobs AND Triggers? Many job schedulers do not have separate notions of jobs and triggers. Some define a ‘job’ as simply an execution time (or schedule) along with some small job identifier. Others are much like the union of Quartz’s job and trigger objects. While developing Quartz, we decided that it made sense to create a separation between the schedule and the work to be performed on that schedule. This has (in our opinion) many benefits.

什么是工作和触发器？许多作业调度程序没有作业和触发器的单独概念。有些人将 `作业` 定义为简单的执行时间（或调度）以及一些小型作业标识符。其他的则很像 Quartz 的工作和触发器对象的结合。在开发 Quartz 时，我们决定在日程安排和在该日程安排上执行的工作之间创建一个分离是有意义的。（在我们看来）这有许多好处。

For example, Jobs can be created and stored in the job scheduler independent of a trigger, and many triggers can be associated with the same job. Another benefit of this loose-coupling is the ability to configure jobs that remain in the scheduler after their associated triggers have expired, so that that it can be rescheduled later, without having to re-define it. It also allows you to modify or replace a trigger without having to re-define its associated job.

例如，可以独立于触发器而在作业调度程序中创建和存储作业，而且许多触发器可以与同一作业关联。这种松耦合的另一个好处是能够配置在关联触发器过期后仍留在调度器中的作业，以便以后可以重新调度，而不必重新定义它。它还允许你修改或替换触发器，而不必重新定义其关联的作业。

## Identities

标识符

Jobs and Triggers are given identifying keys as they are registered with the Quartz scheduler. The keys of Jobs and Triggers (JobKey and TriggerKey) allow them to be placed into ‘groups’ which can be useful for organizing your jobs and triggers into categories such as `reporting jobs` and `maintenance jobs`. The name portion of the key of a job or trigger must be unique within the group - or in other words, the complete key (or identifier) of a job or trigger is the compound of the name and group.

当作业和触发器注册到 Quartz 调度器时，它们被赋予一个标识符 key。作业和触发器的 key（JobKey 和 TriggerKey）允许将它们放置到 `组` 中，这对于将作业和触发器组织到诸如 `报告作业` 和 `维护作业` 等类别中非常有用。作业或触发器的 key 名部分在组中必须是唯一的。换句话说，作业或触发器的完整 key（或标识符）是名称和组的复合。

You now have a general idea about what Jobs and Triggers are, you can learn more about them in Lesson 3: More About Jobs & JobDetails and Lesson 4: More About Triggers.

现在你对 Jobs 和 Triggers 有了一个大致的了解，你可以在第三课：more about Jobs & JobDetails 和第四课：more about Triggers 中更多地了解它们。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 1: Using Quartz](/Tutorials/Lesson-1)**
- **Next Lesson（下一课）：[Lesson 3: More About Jobs & JobDetails](/Tutorials/Lesson-3)**
