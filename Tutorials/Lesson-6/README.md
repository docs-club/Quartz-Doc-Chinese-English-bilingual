# Lesson 6: CronTrigger

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-06.html

CronTrigger is often more useful than SimpleTrigger, if you need a job-firing schedule that recurs based on calendar-like notions, rather than on the exactly specified intervals of SimpleTrigger.

如果你需要一个基于类似日历的概念（而不是精确指定的 SimpleTrigger 间隔）重复执行的 job-firing 调度程序，那么 CronTrigger 通常比 SimpleTrigger 更有用。

With CronTrigger, you can specify firing-schedules such as `every Friday at noon`, or `every weekday and 9:30 am`, or even `every 5 minutes between 9:00 am and 10:00 am on every Monday, Wednesday and Friday during January`.

使用 CronTrigger，你可以指定 `每周五中午` 或 `每个工作日和上午 9:30`，甚至 `一月期间，每周一、三、五的上午 9:00 至上午 10:00 每 5 分钟` 等工作时间表。

Even so, like SimpleTrigger, CronTrigger has a startTime which specifies when the schedule is in force, and an (optional) endTime that specifies when the schedule should be discontinued.

## Cron Expressions

即便如此，与 SimpleTrigger 一样，CronTrigger 也有一个 startTime，指定计划何时生效，还有一个可选的 endTime，指定计划何时应该停止。

Cron-Expressions are used to configure instances of CronTrigger. Cron-Expressions are strings that are actually made up of seven sub-expressions, that describe individual details of the schedule. These sub-expression are separated with white-space, and represent:

Cron 表达式用于配置 CronTrigger 的实例。Cron 表达式实际上是由 7 个子表达式组成的字符串，这些子表达式描述了时间调度的各个细节。这些子表达式用空格隔开，表示为：

1. Seconds
2. Minutes
3. Hours
4. Day-of-Month
5. Month
6. Day-of-Week
7. Year (optional field)

An example of a complete cron-expression is the string `0 0 12 ? * WED` - which means `every Wednesday at 12:00:00 pm`.

完整的 cron 表达式的一个例子是字符串 `0 0 12 ? * WED`，意思是 `每周三下午 12:00:00`。

Individual sub-expressions can contain ranges and/or lists. For example, the day of week field in the previous (which reads `WED`) example could be replaced with `MON-FRI`, `MON,WED,FRI`, or even `MON-WED,SAT`.

单个子表达式可以包含范围和（或）列表。例如，上一个例子中的星期字段（读作 `WED`）可以被替换为 `MON-FRI`，`MON,WED,FRI`，甚至 `MON-WED,SAT`。

Wild-cards (the ‘’ character) can be used to say `every` possible value of this field. Therefore the ‘’ character in the `Month` field of the previous example simply means `every month`. A `*` in the Day-Of-Week field would therefore obviously mean `every day of the week`.

通配符（`*` 字符）可以用来表示该字段的 `每个` 可能值。因此，上例中 `Month` 字段中的 `*` 字符仅仅表示 `每个月`。Day-Of-Week 字段中的 `*` 显然意味着一周中的每一天。

注：原文可能是遗漏了字符。

All of the fields have a set of valid values that can be specified. These values should be fairly obvious - such as the numbers 0 to 59 for seconds and minutes, and the values 0 to 23 for hours. Day-of-Month can be any value 1-31, but you need to be careful about how many days are in a given month! Months can be specified as values between 0 and 11, or by using the strings JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV and DEC. Days-of-Week can be specified as values between 1 and 7 (1 = Sunday) or by using the strings SUN, MON, TUE, WED, THU, FRI and SAT.

所有字段都有一组可以指定的有效值。这些值应该是相当明显的。例如，代表秒和分钟的数字 0 到 59，以及代表小时的值 0 到 23。Day-of-Month 可以是任意值 1-31，但是你需要注意一个给定的月份中有多少天！月可以指定值 0 到 11 之间的值，或通过使用字符串 JAN、FEB、MAR、APR、MAY、JUN、JUL、AUG、SEP、OCT、NOV 以及 DEC，Days-of-Week 之间可以指定值 1 和 7（1 = 周日）或通过使用字符串 SUN、MON、TUE、WED、THU、FRI 以及 SAT

The ‘/’ character can be used to specify increments to values. For example, if you put ‘0/15’ in the Minutes field, it means ‘every 15th minute of the hour, starting at minute zero’. If you used ‘3/20’ in the Minutes field, it would mean ‘every 20th minute of the hour, starting at minute three’ - or in other words it is the same as specifying ‘3,23,43’ in the Minutes field. Note the subtlety that `/35` does `*`not mean `every 35 minutes` - it mean `every 35th minute of the hour, starting at minute zero` - or in other words the same as specifying ‘0,35’.

注：原文最后一句的 `*` 出现的位置可能不正确。

字符 ‘/’ 可用于指定值的增量。例如，如果你在分钟字段中输入 ‘0/15’，它意味着 `从零分钟开始，每隔 15 分钟，`。如果在分钟字段中使用 ‘3/20’，则意味着 `从第 3 分钟开始，每隔 20 分钟` ，或者换句话说，它与在分钟字段中指定 `3,23,43` 相同。请注意 `/35` 的微妙之处，它不表示 `每 35 分钟`，而是表示 `从零时开始每小时 35 分钟`，换句话说，与指定 `0,35` 相同。

The ‘?’ character is allowed for the day-of-month and day-of-week fields. It is used to specify `no specific value`. This is useful when you need to specify something in one of the two fields, but not the other. See the examples below (and CronTrigger JavaDoc) for clarification.

‘?’ 字符允许在 day-of-month 和 day-of-week 字段中使用。它表示 `不指定值`。当你需要在两个字段中的一个中指定某个内容，而不是另一个时，这是非常有用的。请参阅下面的示例（以及 CronTrigger JavaDoc）以了解详情。

注：`？` 可理解为占位符，不指定具体值

The ‘L’ character is allowed for the day-of-month and day-of-week fields. This character is short-hand for `last`, but it has different meaning in each of the two fields. For example, the value `L` in the day-of-month field means `the last day of the month` - day 31 for January, day 28 for February on non-leap years. If used in the day-of-week field by itself, it simply means `7` or `SAT`. But if used in the day-of-week field after another value, it means `the last xxx day of the month` - for example `6L` or `FRIL` both mean `the last friday of the month`. You can also specify an offset from the last day of the month, such as `L-3` which would mean the third-to-last day of the calendar month. When using the ‘L’ option, it is important not to specify lists, or ranges of values, as you’ll get confusing/unexpected results.

‘L’ 字符允许出现在 day-of-month 和 day-of-week 字段中。这个字符是 `last` 的简写，但是它在这两个字段中有不同的含义。例如，day-of-month 字段中的值 `L` 表示 `一个月的最后一天`。在非闰年中，1 月是第 31 天，2 月是第 28 天。如果单独用于一周的天数，它仅仅表示 `7` 或 `SAT`。但是，如果将 day-of-week 字段用于另一个值之后，则表示 `一个月的最后一个 xxx 日`。例如 `6L` 或 `FRIL` 都表示 `一个月的最后一个星期五`。还可以指定一个月最后一天的偏移量，例如 `L-3`，它表示日历月份的倒数第三天。当使用 ‘L’ 选项时，重要的是不要指定列表或值的范围，因为你会得到令人困惑/意外的结果。

The ‘W’ is used to specify the weekday (Monday-Friday) nearest the given day. As an example, if you were to specify `15W` as the value for the day-of-month field, the meaning is: `the nearest weekday to the 15th of the month`.

‘W’ 用于指定离指定日期最近的工作日（周一至周五）。例如，如果你指定 `15W` 作为 day-of-month 字段的值，则含义为：`离每月15日最近的工作日`。

The ‘#’ is used to specify `the nth` XXX weekday of the month. For example, the value of `6#3` or `FRI#3` in the day-of-week field means `the third Friday of the month`.

‘#’ 用于指定一个月的第 n 个工作日。例如，`6#3` 或 `FRI#3` 在 day-of-week 字段中表示 `一个月的第三个星期五`。

Here are a few more examples of expressions and their meanings - you can find even more in the JavaDoc for org.quartz.CronExpression

这里有更多关于表达式及其含义的例子，你可以在 JavaDoc 中找到更多关于 org.quartz.CronExpression 的例子

## Example Cron Expressions

CronTrigger Example 1 - an expression to create a trigger that simply fires every 5 minutes

创建一个触发器，每隔 5 分钟触发一次

`0 0/5 * * * ?`

CronTrigger Example 2 - an expression to create a trigger that fires every 5 minutes, at 10 seconds after the minute (i.e. 10:00:10 am, 10:05:10 am, etc.).

创建一个触发器，每 5 分钟触发一次，每分钟过 10 秒触发一次（比如 10:00:10 am，10:05:10 am，等等）。

`10 0/5 * * * ?`

CronTrigger Example 3 - an expression to create a trigger that fires at 10:30, 11:30, 12:30, and 13:30, on every Wednesday and Friday.

创建一个触发器，在每周三和周五的 10:30、11:30、12:30 和 13:30 触发。

`0 30 10-13 ? * WED,FRI`

CronTrigger Example 4 - an expression to create a trigger that fires every half hour between the hours of 8 am and 10 am on the 5th and 20th of every month. Note that the trigger will NOT fire at 10:00 am, just at 8:00, 8:30, 9:00 and 9:30

在每个月的 5 号和 20 号早上 8 点到 10 点之间每半小时制造一个触发点。请注意，触发器不会在上午 10 点触发，只会在 8:00、8:30、9:00 和 9:30 触发。

`0 0/30 8-9 5,20 * ?`

Note that some scheduling requirements are too complicated to express with a single trigger - such as `every 5 minutes between 9:00 am and 10:00 am, and every 20 minutes between 1:00 pm and 10:00 pm`. The solution in this scenario is to simply create two triggers, and register both of them to run the same job.

请注意，有些日程安排要求过于复杂，无法用单个触发器来表达，例如 `在上午 9 时至上午 10 时每隔 5 分钟，以及在下午 1 时至晚上 10 时每隔 20 分钟`。此场景中的解决方案是简单地创建两个触发器，并注册它们来运行相同的作业。

## Building CronTriggers

CronTrigger instances are built using TriggerBuilder (for the trigger’s main properties) and CronScheduleBuilder (for the CronTrigger-specific properties). To use these builders in a DSL-style, use static imports:

CronTrigger 实例使用 TriggerBuilder（用于触发器的主属性）和 CronScheduleBuilder（用于特定于 CronTrigger 的属性）构建。要在 DSL 风格中使用这些构建器，请使用静态导入：

```java
import static org.quartz.TriggerBuilder.*;
import static org.quartz.CronScheduleBuilder.*;
import static org.quartz.DateBuilder.*:
```

**Build a trigger that will fire every other minute, between 8am and 5pm, every day:**

每天早上 8 点到下午 5 点之间，每隔一分钟触发一次：

```java
trigger = newTrigger()
  .withIdentity("trigger3", "group1")
  .withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
  .forJob("myJob", "group1")
  .build();
```

**Build a trigger that will fire daily at 10:42 am:**

每日上午 10 时 42 分触发：

```java
trigger = newTrigger()
  .withIdentity("trigger3", "group1")
  .withSchedule(dailyAtHourAndMinute(10, 42))
  .forJob(myJobKey)
  .build();
```

或者：

```java
trigger = newTrigger()
  .withIdentity("trigger3", "group1")
  .withSchedule(cronSchedule("0 42 10 * * ?"))
  .forJob(myJobKey)
  .build();
```

**Build a trigger that will fire on Wednesdays at 10:42 am, in a TimeZone other than the system’s default:**

将于周三上午 10:42 在系统默认时区以外的时区启动：

```java
trigger = newTrigger()
  .withIdentity("trigger3", "group1")
  .withSchedule(weeklyOnDayAndHourAndMinute(DateBuilder.WEDNESDAY, 10, 42))
  .forJob(myJobKey)
  .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
  .build();
```

或者：

```java
trigger = newTrigger()
  .withIdentity("trigger3", "group1")
  .withSchedule(cronSchedule("0 42 10 ? * WED"))
  .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
  .forJob(myJobKey)
  .build();
```

## CronTrigger Misfire Instructions

The following instructions can be used to inform Quartz what it should do when a misfire occurs for CronTrigger. (Misfire situations were introduced in the More About Triggers section of this tutorial). These instructions are defined as constants on CronTrigger itself (including JavaDoc describing their behavior). The instructions include:

下面的指令可以用来告诉 Quartz，当 CronTrigger 发生失效时它应该做什么。（失效情况在本教程的 More About Triggers 一节中介绍）。这些指令被定义为 CronTrigger 本身的常量（包括描述它们行为的 JavaDoc）。指令包括：

**Misfire Instruction Constants of CronTrigger**

```java
MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
MISFIRE_INSTRUCTION_DO_NOTHING
MISFIRE_INSTRUCTION_FIRE_NOW
```

All triggers also have the Trigger.MISFIRE_INSTRUCTION_SMART_POLICY instruction available for use, and this instruction is also the default for all trigger types. The ‘smart policy’ instruction is interpreted by CronTrigger as MISFIRE_INSTRUCTION_FIRE_NOW. The JavaDoc for the CronTrigger.updateAfterMisfire() method explains the exact details of this behavior.

所有触发器都有可以使用的 Trigger.MISFIRE_INSTRUCTION_SMART_POLICY 指令，该指令也是所有触发器类型的默认指令。`smart policy` 指令被 CronTrigger 解释为 MISFIRE_INSTRUCTION_FIRE_NOW。关于 `CronTrigger.updateAfterMisfire()` 方法及其行为的确切细节在 JavaDoc 有解释。

When building CronTriggers, you specify the misfire instruction as part of the simple schedule (via CronSchedulerBuilder):

当构建 CronTrigger 时，你可以将失效指令作为简单调度程序的一部分（通过 CronSchedulerBuilder）：

```java
trigger = newTrigger()
  .withIdentity("trigger3", "group1")
  .withSchedule(cronSchedule("0 0/2 8-17 * * ?")
    ..withMisfireHandlingInstructionFireAndProceed())
  .forJob("myJob", "group1")
  .build();
```

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 5: SimpleTriggers](/Tutorials/Lesson-5)**
- **Next Lesson（下一课）：[Lesson 7: TriggerListeners & JobListeners](/Tutorials/Lesson-7)**
