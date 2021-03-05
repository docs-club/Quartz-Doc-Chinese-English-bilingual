# Lesson 8: SchedulerListeners

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-08.html

SchedulerListeners are much like TriggerListeners and JobListeners, except they receive notification of events within the Scheduler itself - not necessarily events related to a specific trigger or job.

SchedulerListener 很像 TriggerListener 和 JobListener，只是它们接收的是调度程序本身内的事件通知（不一定是与特定触发器或作业相关的事件）。

Scheduler-related events include: the addition of a job/trigger, the removal of a job/trigger, a serious error within the scheduler, notification of the scheduler being shutdown, and others.

与调度器相关的事件包括：作业/触发器的添加、作业/触发器的删除、调度器中的严重错误、调度器被关闭的通知等等。

**The org.quartz.SchedulerListener Interface**

```java
public interface SchedulerListener {

    public void jobScheduled(Trigger trigger);

    public void jobUnscheduled(String triggerName, String triggerGroup);

    public void triggerFinalized(Trigger trigger);

    public void triggersPaused(String triggerName, String triggerGroup);

    public void triggersResumed(String triggerName, String triggerGroup);

    public void jobsPaused(String jobName, String jobGroup);

    public void jobsResumed(String jobName, String jobGroup);

    public void schedulerError(String msg, SchedulerException cause);

    public void schedulerStarted();

    public void schedulerInStandbyMode();

    public void schedulerShutdown();

    public void schedulingDataCleared();
}
```

SchedulerListeners are registered with the scheduler’s ListenerManager. SchedulerListeners can be virtually any object that implements the org.quartz.SchedulerListener interface.

Schedulerlistener 在调度程序的 ListenerManager 中注册。SchedulerListener 实际上可以是实现 org.quartz.SchedulerListener 接口的任何对象。

**Adding a SchedulerListener:**

```java
scheduler.getListenerManager().addSchedulerListener(mySchedListener);
```

**Removing a SchedulerListener:**

```java
scheduler.getListenerManager().removeSchedulerListener(mySchedListener);
```

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 7: TriggerListeners & JobListeners](/Tutorials/Lesson-7)**
- **Next Lesson（下一课）：[Lesson 9: JobStores](/Tutorials/Lesson-9)**