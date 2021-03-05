# Lesson 7: TriggerListeners and JobListeners

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-07.html

Listeners are objects that you create to perform actions based on events occurring within the scheduler. As you can probably guess, TriggerListeners receive events related to triggers, and JobListeners receive events related to jobs.

监听器是用于根据调度程序中发生的事件执行操作的对象。正如你可能猜到的，TriggerListener 接收与触发器相关的事件，而 JobListener 接收与作业相关的事件。

创建以执行基于调度程序中发生的事件的操作

Trigger-related events include: trigger firings, trigger mis-firings (discussed in the “Triggers” section of this document), and trigger completions (the jobs fired off by the trigger is finished).

与触发器相关的事件包括：触发器触发、触发器失效触发（在本文档的 Triggers 一节中讨论）和触发器完成（由触发器触发的作业执行完成）。

**The org.quartz.TriggerListener Interface**

```java
public interface TriggerListener {

    public String getName();

    public void triggerFired(Trigger trigger, JobExecutionContext context);

    public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);

    public void triggerMisfired(Trigger trigger);

    public void triggerComplete(Trigger trigger, JobExecutionContext context,
            int triggerInstructionCode);
}
```

Job-related events include: a notification that the job is about to be executed, and a notification when the job has completed execution.

与作业相关的事件包括：作业即将执行的通知，以及作业完成执行时的通知。

**The org.quartz.JobListener Interface**

```java
public interface JobListener {

    public String getName();

    public void jobToBeExecuted(JobExecutionContext context);

    public void jobExecutionVetoed(JobExecutionContext context);

    public void jobWasExecuted(JobExecutionContext context,
            JobExecutionException jobException);

}
```

## Using Your Own Listeners

To create a listener, simply create an object that implements the org.quartz.TriggerListener and/or org.quartz.JobListener interface. Listeners are then registered with the scheduler during run time, and must be given a name (or rather, they must advertise their own name via their getName() method).

要创建监听器，只需创建一个实现 org.quartz.TriggerListener 接口或 org.quartz.JobListener 接口的对象。然后，监听器在运行时向调度程序注册，并且必须提供一个名称（或者更确切地说，它们必须通过 getName() 方法声明它们自己的名称）。

For your convenience, tather than implementing those interfaces, your class could also extend the class JobListenerSupport or TriggerListenerSupport and simply override the events you’re interested in.

为了方便，除了实现这些接口，你还可以继承 JobListenerSupport 或 TriggerListenerSupport，并简单地覆盖感兴趣的事件。

Listeners are registered with the scheduler’s ListenerManager along with a Matcher that describes which Jobs/Triggers the listener wants to receive events for.

监听器与一个 Matcher 一起注册到调度程序的 ListenerManager 中，Matcher 描述监听器希望为哪些作业/触发器接收事件。

> Listeners are registered with the scheduler during run time, and are NOT stored in the JobStore along with the jobs and triggers. This is because listeners are typically an integration point with your application. Hence, each time your application runs, the listeners need to be re-registered with the scheduler.

监听器在运行时向调度程序注册，不会与作业和触发器一起存储在 JobStore 中。这是因为监听器通常是与应用程序的集成点。因此，每次应用程序运行时，都需要向调度程序重新注册监听器。

**Adding a JobListener that is interested in a particular job:**

对特定作业添加 JobListener

```java
scheduler.getListenerManager().addJobListener(myJobListener, KeyMatcher.jobKeyEquals(new JobKey("myJobName", "myJobGroup")));
```

You may want to use static imports for the matcher and key classes, which will make your defining the matchers cleaner:

你可能想要为 matcher 和 key 类使用静态导入，这将使你的 matcher 定义更清晰：

```java
import static org.quartz.JobKey.*;
import static org.quartz.impl.matchers.KeyMatcher.*;
import static org.quartz.impl.matchers.GroupMatcher.*;
import static org.quartz.impl.matchers.AndMatcher.*;
import static org.quartz.impl.matchers.OrMatcher.*;
import static org.quartz.impl.matchers.EverythingMatcher.*;
...etc.
```

Which turns the above example into this:

上面的例子变成:

```java
scheduler.getListenerManager().addJobListener(myJobListener, jobKeyEquals(jobKey("myJobName", "myJobGroup")));
```

**Adding a JobListener that is interested in all jobs of a particular group:**

对特定组的所有作业添加 JobListener

```java
scheduler.getListenerManager().addJobListener(myJobListener, jobGroupEquals("myJobGroup"));
```

**Adding a JobListener that is interested in all jobs of two particular groups:**

对两个特定组的所有作业添加 JobListener

```java
scheduler.getListenerManager().addJobListener(myJobListener, or(jobGroupEquals("myJobGroup"), jobGroupEquals("yourGroup")));
```

**Adding a JobListener that is interested in all jobs:**

对所有作业添加 JobListener

```java
scheduler.getListenerManager().addJobListener(myJobListener, allJobs());
```

…Registering TriggerListeners works in just the same way.

注册 TriggerListener 的方式完全相同，参考上面例子。

Listeners are not used by most users of Quartz, but are handy when application requirements create the need for the notification of events, without the Job itself having to explicitly notify the application.

大多数 Quartz 用户并不使用监听器，当应用程序有创建事件通知的需要时，监听器很方便，而作业本身不会显式地通知应用程序。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 6: CronTriggers](/Tutorials/Lesson-6)**
- **Next Lesson（下一课）：[Lesson 8: SchedulerListeners](/Tutorials/Lesson-8)**
