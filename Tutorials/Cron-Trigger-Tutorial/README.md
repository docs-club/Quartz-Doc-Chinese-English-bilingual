# CronTrigger Tutorial

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/crontrigger.html

## Introduction

cron is a UNIX tool that has been around for a long time, so its scheduling capabilities are powerful and proven. The CronTrigger class is based on the scheduling capabilities of cron.

cron 是一种由来已久的 UNIX 工具，因此它的调度能力非常强大，而且经过了验证。CronTrigger 类基于 cron 的调度能力。

CronTrigger uses `cron expressions`, which are able to create firing schedules such as: `At 8:00am every Monday through Friday` or `At 1:30am every last Friday of the month`.

CronTrigger 使用 cron 表达式，它能够创建触发时间表，例如：每周一至周五上午 8 点或每月最后一个周五的凌晨 1:30。

Cron expressions are powerful, but can be pretty confusing. This tutorial aims to take some of the mystery out of creating a cron expression, giving users a resource which they can visit before having to ask in a forum or mailing list.

Cron 表达式很强大，但也很容易混淆。本教程旨在消除创建 cron 表达式的一些困惑，使用户通过论坛或邮件询问之前可以获得启发。

## Format

A cron expression is a string comprised of 6 or 7 fields separated by white space. Fields can contain any of the allowed values, along with various combinations of the allowed special characters for that field. The fields are as follows:

一个 cron 表达式是一个由 6 或 7 个由空格分隔的字段组成的字符串。字段可以包含任何允许的值，以及该字段允许的特殊字符的各种组合。各字段说明如下：

|  Field Name  | Mandatory |  Allowed Values  | Allowed Special Characters |
| :----------: | :-------: | :--------------: | :------------------------: |
|   Seconds    |    YES    |       0-59       |         `, - * /`          |
|   Minutes    |    YES    |       0-59       |         `, - * /`          |
|    Hours     |    YES    |       0-23       |         `, - * /`          |
| Day of month |    YES    |       1-31       |      `, - * ? / L W`       |
|    Month     |    YES    | 1-12 or JAN-DEC  |         `, - * /`          |
| Day of week  |    YES    |  1-7 or SUN-SAT  |      `, - * ? / L #`       |
|     Year     |    NO     | empty, 1970-2099 |         `, - * /`          |

So cron expressions can be as simple as this: `* * * * ? *`

因此，cron 表达式可以像这样简单：`* * * * ? *`

or more complex, like this: `0/5 14,18,3-39,52 * ? JAN,MAR,SEP MON-FRI 2002-2010`

或者更复杂的，像这样：`0/5 14,18,3-39,52 * ? JAN,MAR,SEP MON-FRI 2002-2010`

## Special characters

- `*` (`all values`) - used to select all values within a field. For example, `*` in the minute field means `every minute`.

所有值，用于选择一个字段内的所有值。例如，分钟字段中的 `*` 表示每分钟。

- `?` (`no specific value`) - useful when you need to specify something in one of the two fields in which the character is allowed, but not the other. For example, if I want my trigger to fire on a particular day of the month (say, the 10th), but don’t care what day of the week that happens to be, I would put `10` in the day-of-month field, and `?` in the day-of-week field. See the examples below for clarification.

不指定值，当你需要在两个允许字符的字段之一指定某些东西，而另一个不允许时，这很有用。例如，如果我希望我的触发器在一个月中的某一天（比如第 10 天）触发，但不关心它是星期几，那么我将在 day-of-month 字段中放入 10，然后 `?` 放在 day-of-week。请参见下面的示例进行说明。

- `-` used to specify ranges. For example, `10-12` in the hour field means `the hours 10, 11 and 12`.

用于指定范围。例如，小时字段中的 10-12 表示小时 10、11、12。

- `,` - used to specify additional values. For example, `MON,WED,FRI` in the day-of-week field means `the days Monday, Wednesday, and Friday`.

用于指定值。例如，`MON,WED,FRI` 在 day-of-week 字段中表示星期一、星期三和星期五。

- `/` - used to specify increments. For example, `0/15` in the seconds field means `the seconds 0, 15, 30, and 45`. And `5/15` in the seconds field means `the seconds 5, 20, 35, and 50`. You can also specify `/` after the ` ` character - in this case ` ` is equivalent to having `0` before the `/`. `1/3` in the day-of-month field means `fire every 3 days starting on the first day of the month`.

用于指定增量。例如，秒字段中的 0/15 表示第 0、15、30 和 45 秒；5/15 表示第 5 20 35 50 秒。你也可以在字符后指定 `/`，在这种情况下相当于在 `/` 之前有 0。例如，1/3 在 day-of-month 字段中是指从每月的第一天起，每 3 天触发一次。

- `L` (`last`) - has different meaning in each of the two fields in which it is allowed. For example, the value `L` in the day-of-month field means `the last day of the month` - day 31 for January, day 28 for February on non-leap years. If used in the day-of-week field by itself, it simply means `7` or `SAT`. But if used in the day-of-week field after another value, it means `the last xxx day of the month` - for example `6L` means `the last friday of the month`. You can also specify an offset from the last day of the month, such as `L-3` which would mean the third-to-last day of the calendar month. When using the `L` option, it is important not to specify lists, or ranges of values, as you’ll get confusing/unexpected results.

最后。在允许使用的两个字段中有不同的含义。例如，day-of-month 字段中的值 L 表示月份的最后一天（在非闰年中，1 月是第 31 天，2 月是第 28 天）。如果单独用于 day-of-week 字段，它仅表示 7 或 SAT。但如果用于 day-of-week 字段后的另一个值，它表示这个月的最后 xxx 天。例如 6L 表示这个月的最后一个星期五。还可以指定一个月最后一天的偏移量，例如 L-3，它表示日历月份的倒数第三天。当使用 L 选项时，重要的是不要指定列表或值的范围，因为你会得到令人困惑/意外的结果。

- `W` (`weekday`) - used to specify the weekday (Monday-Friday) nearest the given day. As an example, if you were to specify `15W` as the value for the day-of-month field, the meaning is: `the nearest weekday to the 15th of the month`. So if the 15th is a Saturday, the trigger will fire on Friday the 14th. If the 15th is a Sunday, the trigger will fire on Monday the 16th. If the 15th is a Tuesday, then it will fire on Tuesday the 15th. However if you specify `1W` as the value for day-of-month, and the 1st is a Saturday, the trigger will fire on Monday the 3rd, as it will not `jump` over the boundary of a month’s days. The `W` character can only be specified when the day-of-month is a single day, not a range or list of days.

用于指定离指定日期最近的工作日（周一至周五）。例如，如果指定 15W 作为 day-of-month 字段的值，其含义是：离每月 15 日最近的工作日。所以如果 15 号是星期六，触发器将在 14 号星期五启动。如果 15 号是星期天，触发器将在 16 号星期一启动。如果 15 号是星期二，那么它会在 15 号星期二触发。但是，如果你指定 1W 作为月的一天的值，并且第一个是星期六，那么触发器将在 3 号星期一触发，因为它不会跳过一个月的天数的边界。W 字符只能在某月的某一天，而不是某一范围或天数列表时指定。

> The `L` and `W` characters can also be combined in the day-of-month field to yield `LW`, which translates to "last weekday of the month".

L 和 W 字符也可以在 day-of-month 字段中组合，产生 LW，即一个月的最后一个工作日。

- `#` - used to specify `the nth` XXX day of the month. For example, the value of `6#3` in the day-of-week field means `the third Friday of the month` (day 6 = Friday and `#3` = the 3rd one in the month). Other examples: `2#1` = the first Monday of the month and `4#5` = the fifth Wednesday of the month. Note that if you specify `#5` and there is not 5 of the given day-of-week in the month, then no firing will occur that month.

用于指定一个月的第 n 个日期。例如，day-of-week 字段中的值 `6#3` 表示这个月的第三个星期五（第 6 天是星期五，`#3`表示这个月的第三个星期五）。其他例子:`2#1` 表示这个月的第一个星期一，`4#5` 表示这个月的第五个星期三。请注意，如果您指定了 `#5`，而这个月中给定的星期几中没有 5 个，那么这个月将不会发生触发。

> The legal characters and the names of months and days of the week are not case sensitive. MON is the same as mon.

合法的月份和星期的名称不区分大小写。MON 和 mon 是一样的。

## Examples

Here are some full examples:

| Expression                 | Meaning                                                                                                                             |
| :------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| `0 0 12 * * ?`             | Fire at 12pm (noon) every day                                                                                                       |
| `0 15 10 ? * *`            | Fire at 10:15am every day                                                                                                           |
| `0 15 10 * * ?`            | Fire at 10:15am every day                                                                                                           |
| `0 15 10 * * ? *`          | Fire at 10:15am every day                                                                                                           |
| `0 15 10 * * ? 2005`       | Fire at 10:15am every day during the year 2005                                                                                      |
| `0 * 14 * * ?`             | Fire every minute starting at 2pm and ending at 2:59pm, every day                                                                   |
| `0 0/5 14 * * ?`           | Fire every 5 minutes starting at 2pm and ending at 2:55pm, every day                                                                |
| `0 0/5 14,18 * * ?`        | Fire every 5 minutes starting at 2pm and ending at 2:55pm, AND fire every 5 minutes starting at 6pm and ending at 6:55pm, every day |
| `0 0-5 14 * * ?`           | Fire every minute starting at 2pm and ending at 2:05pm, every day                                                                   |
| `0 10,44 14 ? 3 WED`       | Fire at 2:10pm and at 2:44pm every Wednesday in the month of March.                                                                 |
| `0 15 10 ? * MON-FRI`      | Fire at 10:15am every Monday, Tuesday, Wednesday, Thursday and Friday                                                               |
| `0 15 10 15 * ?`           | Fire at 10:15am on the 15th day of every month                                                                                      |
| `0 15 10 L * ?`            | Fire at 10:15am on the last day of every month                                                                                      |
| `0 15 10 L-2 * ?`          | Fire at 10:15am on the 2nd-to-last last day of every month                                                                          |
| `0 15 10 ? * 6L`           | Fire at 10:15am on the last Friday of every month                                                                                   |
| `0 15 10 ? * 6L`           | Fire at 10:15am on the last Friday of every month                                                                                   |
| `0 15 10 ? * 6L 2002-2005` | Fire at 10:15am on every last friday of every month during the years 2002, 2003, 2004 and 2005                                      |
| `0 15 10 ? * 6#3`          | Fire at 10:15am on the third Friday of every month                                                                                  |
| `0 0 12 1/5 * ?`           | Fire at 12pm (noon) every 5 days every month, starting on the first day of the month.                                               |
| `0 11 11 11 11 ?`          | Fire every November 11th at 11:11am.                                                                                                |

> Pay attention to the effects of `?` and `*` in the day-of-week and day-of-month fields!

注意 `?` 和 `*` 在 day-of-week 和 day-of-month 字段中的影响！

## Notes

- Support for specifying both a day-of-week and a day-of-month value is not complete (you must currently use the `?` character in one of these fields).

同时支持指定 day-of-week 和 day-of-month 字段还不完整（当前必须使用 `?` 作为在这些字段中的一个字符）。

- Be careful when setting fire times between the hours of the morning when “daylight savings” changes occur in your locale (for US locales, this would typically be the hour before and after 2:00 AM - because the time shift can cause a skip or a repeat depending on whether the time moves back or jumps forward. You may find this wikipedia entry helpful in determining the specifics to your locale:
  https://secure.wikimedia.org/wikipedia/en/wiki/Daylight_saving_time_around_the_world

设置时要小心触发时候凌晨之间 `夏令时` 变化（这通常是一个小时之前和之后的凌晨两点，因为时间改变会导致跳过或重复，取决于时间移动或跳跃前进。你可能会发现这个维基百科条目在确定你的区域设置的细节时很有帮助。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**
