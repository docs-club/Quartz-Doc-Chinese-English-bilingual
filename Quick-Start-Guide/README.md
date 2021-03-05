# Quartz Quick Start Guide

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/quick-start.html#download-and-install

(Primarily authored by Dafydd James)

Welcome to the QuickStart guide for Quartz. As you read this guide, expect to see details of:

- Downloading Quartz
- Installing Quartz
- Configuring Quartz to your own particular needs
- Starting a sample application

After becoming familiar with the basic functioning of Quartz Scheduler, consider more advanced features such as Where, an Enterprise feature that allows jobs and triggers to run on specified Terracotta clients instead of randomly chosen ones.

熟悉了 Quartz Scheduler 的基本功能后，可以考虑更高级的特性，比如 Where，它是一个企业特性，允许作业和触发器在特定的 Terracotta 客户端上运行，而不是随机选择的客户端。

## Download and Install

First, [Download the most recent stable release](http://www.quartz-scheduler.org/downloads/) - registration is not required. Unpack the distribution and install it so that your application can see it.

首先，下载最新的稳定版本（不需要注册）。解压发行版并安装它，以便你的应用程序可以看到它。

**The Quartz JAR Files**

The Quartz package includes a number of jar files, located in root directory of the distribution. The main Quartz library is named quartz-xxx.jar (where xxx is a version number). In order to use any of Quartz’s features, this jar must be located on your application’s classpath.

Quartz 包包含许多 jar 文件，它们位于发行版的根目录中。主要的 Quartz 库名为 quartz-xxx.jar（其中 xxx 是版本号）。为了使用 Quartz 的任何特性，这个 jar 必须位于应用程序的 classpath 中。

Once you’ve downloaded Quartz, unzip it somewhere, grab the quartz-xxx.jar and put it where you want it. (If you need information on how to unzip files, go away and learn before you go anywhere near a development environment or the Internet in general. Seriously.)

下载完 Quartz 后，把它解压到某个地方，将 quartz-xxx.jar，放到你想放的地方。（如果你需要关于如何解压缩文件的信息，那么在接近开发环境或一般的 Internet 之前，请先学习一下，讲认真的。）

I use Quartz primarily within an application server environment, so my preference is to include the Quartz JAR within my enterprise application (.ear or .war file). However, if you want to make Quartz available to many applications then simply make sure it’s on the classpath of your appserver. If you are making a stand-alone application, place it on the application’s classpath with all of the other JARs your application depends upon.

假设主要在一个应用程序服务器环境中使用 Quartz，所以我的首选是在我的企业应用程序（.ear 或 .war 文件）。但是，如果你想让 Quartz 对许多应用程序可用，那么只需确保它位于 appserver 的 classpath 上。如果要创建一个独立的应用程序，请将其与应用程序所依赖的所有其他 jar 一起放在应用程序的 classpath 中。

Quartz depends on a number of third-party libraries (in the form of jars) which are included in the distribution .zip file in the ‘lib’ directory. To use all the features of Quartz, all of these jars must also exist on your classpath. If you’re building a stand-alone Quartz application, I suggest you simply add all of them to the classpath. If you’re using Quartz within an app server environment, at least some of the jars will likely already exist on the classpath, so you can afford (if you want) to be a bit more selective as to which jars you include.

Quartz 依赖于许多第三方库（以 jar 的形式），这些库包含在 lib 目录下的 .zip 文件中。要使用 Quartz 的所有特性，所有这些 jar 都必须存在于 classpath 中。如果你正在构建一个独立的 Quartz 应用程序，我建议你简单地将它们全部添加到 classpath 中。如果你在应用服务器环境中使用 Quartz，那么至少有一些 jar 可能已经存在于 classpath 中，所以你可以（如果你愿意）在包含哪些 jar 方面有更大的选择性。

- In an appserver environment, beware of strange results when accidentally including two different versions of the same jar. For example, WebLogic includes an implementation of J2EE (inside weblogic.jar) which may differ to the one in servlet.jar. In this case, it's usually better to leave servlet.jar out of your application, so you know which classes are being utilized.

在 appserver 环境中，当不小心包含同一个 jar 的两个不同版本时，要小心奇怪的结果。例如，WebLogic 包括 J2EE 的实现（在 weblogic.jar 中），这可能与 servlet.jar 中的实现不同。在这种情况下，通常最好将 servlet.jar 排除在应用程序之外，这样你就知道哪些类被利用了。

**The Properties File**

Quartz uses a properties file called (kudos on the originality) quartz.properties. This isn’t necessary at first, but to use anything but the most basic configuration it must be located on your classpath.

Quartz 使用了一个名为 quartz.properties 的属性文件。这在一开始是没有必要的，但是为了使用最基本的配置，它必须位于 classpath 中。

Again, to give an example based on my personal situation, my application was developed using WebLogic Workshop. I keep all of my configuration files (including quartz.properties) in a project under the root of my application. When I package everything up into a .ear file, the config project gets packaged into a .jar which is included within the final .ear. This automatically puts quartz.properties on the classpath.

同样，为了给出一个基于个人情况的例子，我的应用程序是使用 WebLogic Workshop 开发的。我将所有的配置文件（包括 quartz.properties）保存在应用程序根目录下的一个项目中。当我将所有内容打包到 .ear 文件时，配置项目将打包到 .jar 中，该 .jar 包含在最终的 .ear 文件中。这将自动放置 quartz.properties 到 classpath 上。

If you’re building a web application (i.e. in the form of a .war file) that includes Quartz, you will likely want to place the quartz.properties file in the WEB-INF/classes folder in order for it to be on the classpath.

如果你正在构建一个包含 Quartz 的 web 应用程序（即以 .war 文件的形式），你可能希望放置 quartz.properties 文件在 WEB-INF/classes 文件夹中，以便它能出现在 classpath 中。

## Configuration

This is the big bit! Quartz is a very configurable application. The best way to configure Quartz is to edit a quartz.properties file, and place it in your application’s classpath (see Installation section above).

这是最重要的一点！Quartz 是一个非常可配置的应用程序。配置 Quartz 的最佳方式是编辑 quartz.properties 文件，并将其放在应用程序的 classpath 中（参见上面的安装部分）。

There are several example properties files that ship within the Quartz distribution, particularly under the examples/ directory. I would suggest you create your own quartz.properties file, rather than making a copy of one of the examples and deleting the bits you don’t need. It’s neater that way, and you’ll explore more of what Quartz has to offer.

Quartz 发行版中有几个属性文件例子，特别是在 examples/ 目录下。我建议你创造你自己的 quartz.properties 文件，而不是复制一个示例并删除不需要的东西。这样做更方便，但你可以探索 Quartz 提供的更多东西。

Full documentation of available properties is available in the Quartz Configuration Reference.

可用属性的完整文档可在 Quartz 配置参考中获得。

To get up and running quickly, a basic quartz.properties looks something like this:

要快速地启动和运行，一个基本的 quartz.properties 是这样的：

```java
org.quartz.scheduler.instanceName = MyScheduler
org.quartz.threadPool.threadCount = 3
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```

The scheduler created by this configuration has the following characteristics:

此配置创建的调度程序具有以下特征:

- org.quartz.scheduler.instanceName - This scheduler’s name will be “MyScheduler”.

这个调度器的名字将是 MyScheduler

- org.quartz.threadPool.threadCount - There are 3 threads in the thread pool, which means that a maximum of 3 jobs can be run simultaneously.

线程池中有 3 个线程，这意味着最多可以同时运行 3 个作业。

- org.quartz.jobStore.class - All of Quartz’s data, such as details of jobs and triggers, is held in memory (rather than in a database). Even if you have a database and want to use it with Quartz, I suggest you get Quartz working with the RamJobStore before you open up a whole new dimension by working with a database.

Quartz 的所有数据，比如作业和触发器的详细信息，都保存在内存中（而不是数据库中）。即使你有一个数据库，并且希望将它与 Quartz 一起使用，我建议你在使用数据库打开一个全新的维度之前，先让 Quartz 与 RamJobStore 一起工作。

## Starting a Sample Application

Now you’ve downloaded and installed Quartz, it’s time to get a sample application up and running. The following code obtains an instance of the scheduler, starts it, then shuts it down:

现在已经下载并安装了 Quartz，现在可以启动并运行示例应用程序了。用下面的代码获取一个调度程序实例，启动它，然后关闭它：

**QuartzTest.java**

```java
  import org.quartz.Scheduler;
  import org.quartz.SchedulerException;
  import org.quartz.impl.StdSchedulerFactory;
  import static org.quartz.JobBuilder.*;
  import static org.quartz.TriggerBuilder.*;
  import static org.quartz.SimpleScheduleBuilder.*;

  public class QuartzTest {

      public static void main(String[] args) {

          try {
              // Grab the Scheduler instance from the Factory
              Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

              // and start it off
              scheduler.start();

              scheduler.shutdown();

          } catch (SchedulerException se) {
              se.printStackTrace();
          }
      }
  }
```

> Once you obtain a scheduler using StdSchedulerFactory.getDefaultScheduler(), your application will not terminate until you call scheduler.shutdown(), because there will be active threads.

一旦你使用 StdSchedulerFactory.getDefaultScheduler() 获得了一个调度器，你的应用程序将不会终止，直到你调用 scheduler.shutdown()，才会有活动线程。

Note the static imports in the code example; these will come into play in the code example below.

请注意代码示例中的静态导入，它们将在下面的代码示例中发挥作用。

If you have not set up logging, all logs will be sent to the console and your output will look something like this:

如果你还没有设置日志记录，所有的日志都会被发送到控制台，你的输出会像这样：

```java
[INFO] 21 Jan 08:46:27.857 AM main [org.quartz.core.QuartzScheduler]
Quartz Scheduler v.2.0.0-SNAPSHOT created.

[INFO] 21 Jan 08:46:27.859 AM main [org.quartz.simpl.RAMJobStore]
RAMJobStore initialized.

[INFO] 21 Jan 08:46:27.865 AM main [org.quartz.core.QuartzScheduler]
Scheduler meta-data: Quartz Scheduler (v2.0.0) 'Scheduler' with instanceId 'NON_CLUSTERED'
  Scheduler class: 'org.quartz.core.QuartzScheduler' - running locally.
  NOT STARTED.
  Currently in standby mode.
  Number of jobs executed: 0
  Using thread pool 'org.quartz.simpl.SimpleThreadPool' - with 50 threads.
  Using job-store 'org.quartz.simpl.RAMJobStore' - which does not support persistence. and is not clustered.


[INFO] 21 Jan 08:46:27.865 AM main [org.quartz.impl.StdSchedulerFactory]
Quartz scheduler 'Scheduler' initialized from default resource file in Quartz package: 'quartz.properties'

[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.impl.StdSchedulerFactory]
Quartz scheduler version: 2.0.0

[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
Scheduler Scheduler_$_NON_CLUSTERED started.

[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
Scheduler Scheduler_$_NON_CLUSTERED shutting down.

[INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
Scheduler Scheduler_$_NON_CLUSTERED paused.

[INFO] 21 Jan 08:46:27.867 AM main [org.quartz.core.QuartzScheduler]
Scheduler Scheduler_$_NON_CLUSTERED shutdown complete.
```

To do something interesting, you need code between the start() and shutdown() calls.

下面要做一些有趣的事情，需要在 start() 和 shutdown() 调用之间编写代码。

```java
  // define the job and tie it to our HelloJob class
  JobDetail job = newJob(HelloJob.class)
      .withIdentity("job1", "group1")
      .build();

  // Trigger the job to run now, and then repeat every 40 seconds
  Trigger trigger = newTrigger()
      .withIdentity("trigger1", "group1")
      .startNow()
            .withSchedule(simpleSchedule()
              .withIntervalInSeconds(40)
              .repeatForever())
      .build();

  // Tell quartz to schedule the job using our trigger
  scheduler.scheduleJob(job, trigger);
```

(you will also need to allow some time for the job to be triggered and executed before calling shutdown() - for a simple example such as this, you might just want to add a Thread.sleep(60000) call).

在调用 shutdown() 之前，你还需要留出一些时间来触发和执行作业，对于这样一个简单的示例，只需要添加一个 Thread.sleep(60000) 调用即可。

Now go have some fun!

---

**[Back to contents（返回目录）](https://github.com/clxering/Quartz-Doc-Chinese-English-bilingual)**
