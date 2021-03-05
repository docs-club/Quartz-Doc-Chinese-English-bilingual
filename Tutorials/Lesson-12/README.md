# Lesson 12: Miscellaneous Features of Quartz

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-12.html

## Plug-Ins

Quartz provides an interface (org.quartz.spi.SchedulerPlugin) for plugging-in additional functionality.

Quartz 提供了一个接口（org.quartz.spi.SchedulerPlugin）来插入额外的功能。

Plugins that ship with Quartz to provide various utility capabilities can be found documented in the org.quartz.plugins package. They provide functionality such as auto-scheduling of jobs upon scheduler startup, logging a history of job and trigger events, and ensuring that the scheduler shuts down cleanly when the JVM exits.

Quartz 附带的提供各种实用功能的插件可以在 org.quartz.plugins 中找到文档。它们提供了一些功能，比如在调度器启动时自动调度作业、记录作业历史和触发事件，并确保调度器在 JVM 退出时干净地关闭。

## JobFactory

When a trigger fires, the Job it is associated to is instantiated via the JobFactory configured on the Scheduler. The default JobFactory simply calls newInstance() on the job class. You may want to create your own implementation of JobFactory to accomplish things such as having your application’s IoC or DI container produce/initialize the job instance.

当触发器触发时，它关联的作业将通过调度程序上配置的 JobFactory 实例化。默认的 JobFactory 只是在作业类上调用 newInstance()。可能希望创建自己的 JobFactory 实现来完成一些事情，比如让应用程序的 IoC 或 DI 容器生成/初始化作业实例。

See the org.quartz.spi.JobFactory interface, and the associated Scheduler.setJobFactory(fact) method.

详见 org.quartz.spi.JobFactory 接口，以及相关的 Scheduler.setJobFactory(fact) 方法。

## Factory-Shipped Jobs

Quartz also provides a number of utility Jobs that you can use in your application for doing things like sending e-mails and invoking EJBs. These out-of-the-box Jobs can be found documented in the org.quartz.jobs package.

Quartz 还提供了许多实用程序作业，可以在应用程序中使用它们来做诸如发送电子邮件和调用 ejb 之类的事情。这些开箱即用的作业可以在 org.quartz.jobs 中找到。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 11: Advanced (Enterprise) Features](/Tutorials/Lesson-11)**
