# Lesson 4: More About Triggers

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-04.html

Like jobs, triggers are quite easy to work with, but do contain a variety of customizable options that you need to be aware of and understand before you can make full use of Quartz. Also, as noted earlier, there are different types of triggers that you can select from to meet different scheduling needs.

和 jobs 一样，触发器也很容易使用，但它包含了各种可定制的选项，在充分使用 Quartz 之前，你需要了解和理解这些选项。此外，如前所述，你可以选择不同类型的触发器来满足不同的调度需求。

You will learn about the two most common types of triggers in Lesson 5: Simple Triggers and Lesson 6: Cron Triggers.

你将在第 5 课（Simple Triggers）和第 6 课（Cron Triggers）中了解两种最常见的触发器类型。

## Common Trigger Attributes

Aside from the fact that all trigger types have TriggerKey properties for tracking their identities, there are a number of other properties that are common to all trigger types. These common properties are set using the TriggerBuilder when you are building the trigger definition (examples of that will follow).

所有触发器类型都有用于跟踪其标识的 TriggerKey 属性，除此之外还有许多其他通用属性。在构建触发器定义时使用 TriggerBuilder 设置这些通用属性（下面将给出示例）。

Here is a listing of properties common to all trigger types:

以下是所有触发器类型的通用属性列表：

- The `jobKey` property indicates the identity of the job that should be executed when the trigger fires.

jobKey 属性表示当触发器触发时应该执行的作业的标识。

- The `startTime` property indicates when the trigger’s schedule first comes into affect. The value is a java.util.Date object that defines a moment in time on a given calendar date. For some trigger types, the trigger will actually fire at the start time, for others it simply marks the time that the schedule should start being followed. This means you can store a trigger with a schedule such as `every 5th day of the month` during January, and if the startTime property is set to April 1st, it will be a few months before the first firing.

startTime 属性指示触发器的调度第一次生效的时间。它的值是一个 java.util.Date 对象，定义了给定日历日期上的某一时刻。对于某些触发器类型，触发器实际上会在执行时就触发，对于其他类型，它只是标记应该开始执行计划的时间。这意味着你可以在第一次触发之前的几个月，比如一月份，存储一个带有调度的触发器，比如 `每个月5日`，startTime 属性设置为 `4月1日`。

- The `endTime` property indicates when the trigger’s schedule should no longer be in effect. In other words, a trigger with a schedule of `every 5th day of the month` and with an end time of July 1st will fire for it’s last time on June 5th.

endTime 属性指示触发器的计划何时不再生效。换句话说，一个 `每个月5日` 的触发器，并在 `7月1日` 结束，将在 `6月5日` 触发它的最后一次。

Other properties, which take a bit more explanation are discussed in the following sub-sections.

下面的小节将对需要更多解释的其他属性进行讨论。

## Priority

Sometimes, when you have many Triggers (or few worker threads in your Quartz thread pool), Quartz may not have enough resources to immediately fire all of the Triggers that are scheduled to fire at the same time. In this case, you may want to control which of your Triggers get first crack at the available Quartz worker threads. For this purpose, you can set the priority property on a Trigger. If N Triggers are to fire at the same time, but there are only Z worker threads currently available, then the first Z Triggers with the highest priority will be executed first. If you do not set a priority on a Trigger, then it will use the default priority of 5. Any integer value is allowed for priority, positive or negative.

有时，当你有很多触发器（或者 Quartz 线程池中的工作线程很少）时，Quartz 可能没有足够的资源来立即触发所有计划同时触发的触发器。在这种情况下，可能想要控制哪个触发器首先得到可用的 Quartz 工作线程。为此，你可以在触发器上设置 priority 属性。如果同时触发 N 个触发器，但目前只有 Z 个工作线程可用，那么前 Z 个具有最高优先级的触发器将首先执行。如果你没有在触发器上设置优先级，那么它将使用默认优先级 `5`。优先级允许任何正或负的整数值。

Note: Priorities are only compared when triggers have the same fire time. A trigger scheduled to fire at 10:59 will always fire before one scheduled to fire at 11:00.

注意：只有当触发器具有相同的触发时间时才会比较优先级。原定于 10:59 开火的扳机总是会比原定于 11:00 开火的扳机先开火。

Note: When a trigger’s job is detected to require recovery, its recovery is scheduled with the same priority as the original trigger.

注意：当检测到一个触发器的作业需要恢复时，它的恢复将以与原始触发器相同的优先级。

## Misfire Instructions

失效指令

Another important property of a Trigger is its `misfire instruction`. A misfire occurs if a persistent trigger `misses` its firing time because of the scheduler being shutdown, or because there are no available threads in Quartz’s thread pool for executing the job. The different trigger types have different misfire instructions available to them. By default they use a ‘smart policy’ instruction - which has dynamic behavior based on trigger type and configuration. When the scheduler starts, it searches for any persistent triggers that have misfired, and it then updates each of them based on their individually configured misfire instructions. When you start using Quartz in your own projects, you should make yourself familiar with the misfire instructions that are defined on the given trigger types, and explained in their JavaDoc. More specific information about misfire instructions will be given within the tutorial lessons specific to each trigger type.

触发器的另一个重要属性是它的 `失效指令`。如果持久性触发器因为调度程序关闭或 Quartz 的线程池中没有可用的线程来执行作业而 `错过` 其触发时间，则会发生误触发。不同的触发类型有不同的失效指令。默认情况下，它们使用 `smart policy` 指令——根据触发器类型和配置，该指令具有动态行为。当调度程序启动时，它会搜索任何错误触发的持久触发器，然后根据它们各自配置的错误触发指令更新它们。当你开始在自己的项目中使用 Quartz 时，你应该熟悉在给定触发器类型上定义并在项目的 JavaDoc 文档中解释失效指令。关于失效指令的更具体的信息将在每个触发类型的教程课程中给出。

## Calendars

Quartz `Calendar` objects (not java.util.Calendar objects) can be associated with triggers at the time the trigger is defined and stored in the scheduler. Calendars are useful for excluding blocks of time from the the trigger’s firing schedule. For instance, you could create a trigger that fires a job every weekday at 9:30 am, but then add a Calendar that excludes all of the business’s holidays.

Quartz 日历对象（不是 java.util.Calendar 对象）可以在触发器被定义并存储在调度程序中时与触发器相关联。日历用于从触发器的触发计划中排除时间块。例如，你可以创建一个触发器，在每个工作日的上午 9:30 开启一个工作，然后添加一个日历，包含不开展业务的假日。

Calendar’s can be any serializable objects that implement the Calendar interface, which looks like this:

Calendar 可以是任何实现 Calendar 接口的可序列化对象，如下所示：

**The Calendar Interface**

```java
package org.quartz;

public interface Calendar {

public boolean isTimeIncluded(long timeStamp);

public long getNextIncludedTime(long timeStamp);

}
```

Notice that the parameters to these methods are of the long type. As you may guess, they are timestamps in millisecond format. This means that calendars can ‘block out’ sections of time as narrow as a millisecond. Most likely, you’ll be interested in ‘blocking-out’ entire days. As a convenience, Quartz includes the class org.quartz.impl.HolidayCalendar, which does just that.

注意，这些方法的参数都是 long 类型的。正如你可能猜到的，它们是以毫秒格式表示的时间戳。这意味着日历可以 `屏蔽` 短至一毫秒的时间。最有可能的是，你会对 `屏蔽` 一整天感兴趣。为了方便起见，Quartz 包含了 org.quartz.impl.HolidayCalendar 类，它就是做这事的。

Calendars must be instantiated and registered with the scheduler via the addCalendar(..) method. If you use HolidayCalendar, after instantiating it, you should use its addExcludedDate(Date date) method in order to populate it with the days you wish to have excluded from scheduling. The same calendar instance can be used with multiple triggers such as this:

日历必须通过 `addCalendar(..)` 方法实例化并注册到调度程序。如果你使用 HolidayCalendar，在实例化它之后，你应该使用它的 addExcludedDate(Date date) 方法来填充你希望排除的日期。同一个日历实例可以和多个触发器一起使用，例如：

**Calendar Example**

```java
HolidayCalendar cal = new HolidayCalendar();
cal.addExcludedDate( someDate );
cal.addExcludedDate( someOtherDate );

sched.addCalendar("myHolidays", cal, false);

Trigger t = newTrigger()
.withIdentity("myTrigger")
.forJob("myJob")
.withSchedule(dailyAtHourAndMinute(9, 30)) // execute job daily at 9:30
.modifiedByCalendar("myHolidays") // but not on holidays
.build();

// .. schedule job with trigger

Trigger t2 = newTrigger()
.withIdentity("myTrigger2")
.forJob("myJob2")
.withSchedule(dailyAtHourAndMinute(11, 30)) // execute job daily at 11:30
.modifiedByCalendar("myHolidays") // but not on holidays
.build();

// .. schedule job with trigger2
```

The details of the construction/building of triggers will be given in the next couple lessons. For now, just believe that the code above creates two triggers, each scheduled to fire daily. However, any of the firings that would have occurred during the period excluded by the calendar will be skipped.

构造/建造触发器的细节将在接下来的几节课中给出。现在，只需相信上面的代码创建了两个触发器，每个触发器计划每天触发。但是，在日历排除的时间段内的任何触发都将被跳过。

See the org.quartz.impl.calendar package for a number of Calendar implementations that may suit your needs.

org.quartz.impl.calendar 包的许多日历实现可能适合你的需要。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 3: More About Jobs & JobDetails](/Tutorials/Lesson-3)**
- **Next Lesson（下一课）：[Lesson 5: SimpleTriggers](/Tutorials/Lesson-5)**
