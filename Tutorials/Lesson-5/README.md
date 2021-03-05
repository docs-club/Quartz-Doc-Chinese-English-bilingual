# Lesson 5: SimpleTrigger

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-05.html

SimpleTrigger should meet your scheduling needs if you need to have a job execute exactly once at a specific moment in time, or at a specific moment in time followed by repeats at a specific interval. For example, if you want the trigger to fire at exactly 11:23:54 AM on January 13, 2015, or if you want it to fire at that time, and then fire five more times, every ten seconds.

如果你需要在特定的时间点精确地执行一次作业，或者在特定的时间点，特定的间隔内重复执行作业，那么 SimpleTrigger 应该满足你的调度需求。例如，如果你想让触发器在 2015 年 1 月 13 日上午 11:23:54 准时工作，或者如果你想让它在那个时间工作，然后再工作 5 次，每 10 秒工作一次。

With this description, you may not find it surprising to find that the properties of a SimpleTrigger include: a start-time, and end-time, a repeat count, and a repeat interval. All of these properties are exactly what you’d expect them to be, with only a couple special notes related to the end-time property.

通过这样的描述，你可能不会对 SimpleTrigger 的属性感到吃惊，包括：开始时间、结束时间、重复次数和重复间隔。所有这些属性都符合你的期望，只有一些与结束时间属性相关的特别注意事项。

The repeat count can be zero, a positive integer, or the constant value SimpleTrigger.REPEAT_INDEFINITELY. The repeat interval property must be zero, or a positive long value, and represents a number of milliseconds. Note that a repeat interval of zero will cause ‘repeat count’ firings of the trigger to happen concurrently (or as close to concurrently as the scheduler can manage).

重复次数可以是零、正整数或常量值 SimpleTrigger.REPEAT_INDEFINITELY。重复间隔属性必须为零或为正的 long 类型值，表示毫秒数。请注意，重复间隔为 0 将导致触发器的 `重复次数` 以并发方式触发（或者在调度程序能够管理的范围内接近并发方式触发）。

If you’re not already familiar with Quartz’s DateBuilder class, you may find it helpful for computing your trigger fire-times, depending on the startTime (or endTime) that you’re trying to create.

Quartz 的 DateBuilder 类有助于计算触发器的工作时间，取决于你创建的开始时间（或结束时间）。

The endTime property (if it is specified) overrides the repeat count property. This can be useful if you wish to create a trigger such as one that fires every 10 seconds until a given moment in time - rather than having to compute the number of times it would repeat between the start-time and the end-time, you can simply specify the end-time and then use a repeat count of REPEAT_INDEFINITELY (you could even specify a repeat count of some huge number that is sure to be more than the number of times the trigger will actually fire before the end-time arrives).

结束时间属性（如果指定了）将覆盖重复次数属性。如果你希望创建一个触发器，在给定的时间内每 10 秒触发一次，这是非常有用的（不必计算在开始时间和结束时间之间它将重复的次数），你可以简单地指定结束时间，然后使用 REPEAT_INDEFINITELY 作为重复次数，甚至可以指定一个巨大的重复次数，这个重复次数肯定会超过触发器在结束时间到达之前实际触发的次数。

SimpleTrigger instances are built using TriggerBuilder (for the trigger’s main properties) and SimpleScheduleBuilder (for the SimpleTrigger-specific properties). To use these builders in a DSL-style, use static imports:

使用 TriggerBuilder（触发器的主要属性）和 SimpleScheduleBuilder（SimpleTrigger 特有的属性）构建 SimpleTrigger 实例。要在 DSL 风格中使用这些构建器，请使用静态导入：

```java
import static org.quartz.TriggerBuilder.*;
import static org.quartz.SimpleScheduleBuilder.*;
import static org.quartz.DateBuilder.*:
```

Here are various examples of defining triggers with simple schedules, read through them all, as they each show at least one new/different point:

下面是用简单的调度程序定义触发器的各种例子，掌握它们，因为它们每个都至少显示了一个新的/不同的观点：

**Build a trigger for a specific moment in time, with no repeats:**

在特定的时间点建立一个触发器，不重复：

```java
SimpleTrigger trigger = (SimpleTrigger) newTrigger()
  .withIdentity("trigger1", "group1")
  .startAt(myStartTime) // some Date
  .forJob("job1", "group1") // identify job with name, group strings
  .build();
```

**Build a trigger for a specific moment in time, then repeating every ten seconds ten times:**

在特定时刻创建一个触发器，然后每隔 10 秒重复 10 次：

```java
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .startAt(myTimeToStartFiring)  // if a start time is not given (if this line were omitted), "now" is implied
    .withSchedule(simpleSchedule()
        .withIntervalInSeconds(10)
        .withRepeatCount(10)) // note that 10 repeats will give a total of 11 firings
    .forJob(myJob) // identify job with handle to its JobDetail itself
    .build();
```

**Build a trigger that will fire once, five minutes in the future:**

在未来 5 分钟内制造一个触发器，可以触发一次：

```java
  trigger = (SimpleTrigger) newTrigger()
    .withIdentity("trigger5", "group1")
    .startAt(futureDate(5, IntervalUnit.MINUTE)) // use DateBuilder to create a date in the future
    .forJob(myJobKey) // identify job with its JobKey
    .build();
```

**Build a trigger that will fire now, then repeat every five minutes, until the hour 22:00:**

现在建造一个触发器，然后每五分钟重复一次，直到 22:00

```java
trigger = newTrigger()
  .withIdentity("trigger7", "group1")
  .withSchedule(simpleSchedule()
      .withIntervalInMinutes(5)
      .repeatForever())
  .endAt(dateOf(22, 0, 0))
  .build();
```

**Build a trigger that will fire at the top of the next hour, then repeat every 2 hours, forever:**

创建一个触发器，将在满一个小时后工作，然后每 2 小时执行一次，永远：

```java
trigger = newTrigger()
  .withIdentity("trigger8") // because group is not specified, "trigger8" will be in the default group
  .startAt(evenHourDate(null)) // get the next even-hour (minutes and seconds zero ("00:00"))
  .withSchedule(simpleSchedule()
      .withIntervalInHours(2)
      .repeatForever())
  // note that in this example, 'forJob(..)' is not called
  //  - which is valid if the trigger is passed to the scheduler along with the job
  .build();

  scheduler.scheduleJob(trigger, job);
```

Spend some time looking at all of the available methods in the language defined by TriggerBuilder and SimpleScheduleBuilder so that you can be familiar with options available to you that may not have been demonstrated in the examples above.

花一些时间查看 TriggerBuilder 和 SimpleScheduleBuilder 定义的语言中的所有可用方法，以便熟悉上面示例中可能没有演示的可用选项。

> Note that TriggerBuilder (and Quartz's other builders) will generally choose a reasonable value for properties that you do not explicitly set. For examples: if you don't call one of the _withIdentity(..)_ methods, then TriggerBuilder will generate a random name for your trigger; if you don't call _startAt(..)_ then the current time (immediately) is assumed.

请注意，TriggerBuilder（和 Quartz 的其他构建器）通常会为你没有显式设置的属性选择一个合理的值。例如：如果你没有调用 `withIdentity(..)` 方法之一，那么 TriggerBuilder 将为你的触发器生成一个随机名称；如果不调用 `startAt(..)`，则假定为当前时间（即时）。

## SimpleTrigger Misfire Instructions

SimpleTrigger 失效指令

SimpleTrigger has several instructions that can be used to inform Quartz what it should do when a misfire occurs. (Misfire situations were introduced in `Lesson 4: More About Triggers`). These instructions are defined as constants on SimpleTrigger itself (including JavaDoc describing their behavior). The instructions include:

SimpleTrigger 有几个指令，可以用来通知 Quartz 在失败发生时应该做什么。（第四课介绍了失效情况）。这些指令被定义为 SimpleTrigger 本身的常量（JavaDoc 描述了行为）。指令包括：

**Misfire Instruction Constants of SimpleTrigger**

```java
MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
MISFIRE_INSTRUCTION_FIRE_NOW
MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT
```

You should recall from the earlier lessons that all triggers have the Trigger.MISFIRE_INSTRUCTION_SMART_POLICY instruction available for use, and this instruction is also the default for all trigger types.

你应该从前面的课程中回忆起，所有触发器都有可以使用的 MISFIRE_INSTRUCTION_SMART_POLICY 指令，该指令也是所有触发器类型的默认指令。

If the ‘smart policy’ instruction is used, SimpleTrigger dynamically chooses between its various MISFIRE instructions, based on the configuration and state of the given SimpleTrigger instance. The JavaDoc for the SimpleTrigger.updateAfterMisfire() method explains the exact details of this dynamic behavior.

如果使用了 `smart policy` 指令，SimpleTrigger 会根据给定 SimpleTrigger 实例的配置和状态，在其各种 MISFIRE 指令之间进行动态选择。SimpleTrigger.updateAfterMisfire() 方法的 JavaDoc 解释了这种动态行为的确切细节。

When building SimpleTriggers, you specify the misfire instruction as part of the simple schedule (via SimpleSchedulerBuilder):

当构建 SimpleTriggers 时，指定的失效指令是简单调度的一部分（通过 SimpleSchedulerBuilder）：

```java
trigger = newTrigger()
  .withIdentity("trigger7", "group1")
  .withSchedule(simpleSchedule()
    .withIntervalInMinutes(5)
    .repeatForever()
    .withMisfireHandlingInstructionNextWithExistingCount())
  .build();
```

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 4: More About Triggers](/Tutorials/Lesson-4)**
- **Next Lesson（下一课）：[Lesson 6: CronTriggers](/Tutorials/Lesson-6)**