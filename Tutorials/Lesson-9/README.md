# Lesson 9: Job Stores

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-09.html#

JobStore’s are responsible for keeping track of all the `work data` that you give to the scheduler: jobs, triggers, calendars, etc. Selecting the appropriate JobStore for your Quartz scheduler instance is an important step. Luckily, the choice should be a very easy one once you understand the differences between them. You declare which JobStore your scheduler should use (and it’s configuration settings) in the properties file (or object) that you provide to the SchedulerFactory that you use to produce your scheduler instance.

JobStore 负责跟踪你给调度器的所有 `工作数据`：作业、触发器、日历等等。为您的 Quartz 调度器实例选择适当的 JobStore 是一个重要步骤。幸运的是，一旦你理解了它们之间的区别，选择应该是非常容易的。您可以在提供给 SchedulerFactory（用于生成调度程序实例）的属性文件（或对象）中声明调度程序应该使用哪个 JobStore（及其配置设置）。

> Never use a JobStore instance directly in your code. For some reason many people attempt to do this. The JobStore is for behind-the-scenes use of Quartz itself. You have to tell Quartz (through configuration) which JobStore to use, but then you should only work with the Scheduler interface in your code.

永远不要在代码中直接使用 JobStore 实例。由于某些原因，许多人试图这样做。JobStore 是用于在幕后使用 Quartz 本身的。您必须（通过配置）告诉 Quartz 使用哪个 JobStore，但是您应该只在代码中使用调度程序接口。

## RAMJobStore

RAMJobStore is the simplest JobStore to use, it is also the most performant (in terms of CPU time). RAMJobStore gets its name in the obvious way: it keeps all of its data in RAM. This is why it’s lightning-fast, and also why it’s so simple to configure. The drawback is that when your application ends (or crashes) all of the scheduling information is lost - this means RAMJobStore cannot honor the setting of `non-volatility` on jobs and triggers. For some applications this is acceptable - or even the desired behavior, but for other applications, this may be disastrous.

RAMJobStore 是最简单的 JobStore，也是性能最好的（就 CPU 时间而言）。RAMJobStore 的名称由来很明显：它将所有数据保存在 RAM 中。这就是为什么它的速度像闪电一样快，而且配置起来也很简单。缺点是当应用程序结束（或崩溃）时，所有的调度信息都丢失了，这意味着 RAMJobStore 不能执行作业和触发器的 `非波动性` 设置。对于某些应用程序，这是可以接受的，甚至是期望的行为，但对于其他应用程序，这可能是灾难性的。

To use RAMJobStore (and assuming you’re using StdSchedulerFactory) simply specify the class name org.quartz.simpl.RAMJobStore as the JobStore class property that you use to configure quartz:

要使用 RAMJobStore（假设您正在使用 StdSchedulerFactory），只需指定类名 org.quartz.simpl.RAMJobStore 作为用于配置 quartz 的 JobStore 类属性：

**Configuring Quartz to use RAMJobStore**

```java
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```

There are no other settings you need to worry about.

不需要担心其他设置。

## JDBCJobStore

JDBCJobStore is also aptly named - it keeps all of its data in a database via JDBC. Because of this it is a bit more complicated to configure than RAMJobStore, and it also is not as fast. However, the performance draw-back is not terribly bad, especially if you build the database tables with indexes on the primary keys. On fairly modern set of machines with a decent LAN (between the scheduler and database) the time to retrieve and update a firing trigger will typically be less than 10 milliseconds.

JDBCJobStore 的名称很贴切（它通过 JDBC 将所有数据保存在数据库中）。因此，它的配置要比 RAMJobStore 复杂一些，而且速度也没有 RAMJobStore 快。但是，性能上的缺点并不是非常糟糕，尤其是在构建主键上有索引的数据库表时。在具有良好局域网的现代的机器上（在调度器和数据库之间），检索和更新触发触发器的时间通常少于 10 毫秒。

JDBCJobStore works with nearly any database, it has been used widely with Oracle, PostgreSQL, MySQL, MS SQLServer, HSQLDB, and DB2. To use JDBCJobStore, you must first create a set of database tables for Quartz to use. You can find table-creation SQL scripts in the `docs/dbTables` directory of the Quartz distribution. If there is not already a script for your database type, just look at one of the existing ones, and modify it in any way necessary for your DB. One thing to note is that in these scripts, all the the tables start with the prefix `QRTZ_` (such as the tables `QRTZ_TRIGGERS`, and `QRTZ_JOB_DETAIL`). This prefix can actually be anything you’d like, as long as you inform JDBCJobStore what the prefix is (in your Quartz properties). Using different prefixes may be useful for creating multiple sets of tables, for multiple scheduler instances, within the same database.

JDBCJobStore 几乎适用于任何数据库，它已广泛应用于 Oracle、PostgreSQL、MySQL、MS SQLServer、HSQLDB 和 DB2。要使用 JDBCJobStore，必须首先创建一组供 Quartz 使用的数据库表。你可以在 Quartz 发行版的 `docs/dbTables` 目录中找到创建表的 SQL 脚本。如果还没有适合的数据库类型的脚本，只需要查看一个现有的脚本，并以您的数据库需要的任何方式修改它。需要注意的一点是，在这些脚本中，所有表都以前缀 `QRTZ_` 开头（例如表 `QRTZ_TRIGGERS` 和 `QRTZ_JOB_DETAIL`）。这个前缀实际上可以是任何您想要的，只要您通知 JDBCJobStore 这个前缀是什么（在您的 Quartz 属性中）。对于同一个数据库中的多个调度器实例，使用不同的前缀可能有助于创建多组表。

Once you’ve got the tables created, you have one more major decision to make before configuring and firing up JDBCJobStore. You need to decide what type of transactions your application needs. If you don’t need to tie your scheduling commands (such as adding and removing triggers) to other transactions, then you can let Quartz manage the transaction by using JobStoreTX as your JobStore (this is the most common selection).

创建了表之后，在配置和启动 JDBCJobStore 之前，还需要做一个重要的决定。您需要决定应用程序要哪种类型的事务。如果您不需要将调度命令（例如添加和删除触发器）绑定到其他事务，那么您可以让 Quartz 通过使用 JobStoreTX 作为 JobStore（这是最常见的选择）的事务管理。

If you need Quartz to work along with other transactions (i.e. within a J2EE application server), then you should use JobStoreCMT - in which case Quartz will let the app server container manage the transactions.

如果您需要 Quartz 与其他事务一起工作（即在 J2EE 应用服务器内），那么您应该使用 JobStoreCMT（在这种情况下，Quartz 将让应用服务器容器管理事务）。

The last piece of the puzzle is setting up a DataSource from which JDBCJobStore can get connections to your database. DataSources are defined in your Quartz properties using one of a few different approaches. One approach is to have Quartz create and manage the DataSource itself - by providing all of the connection information for the database. Another approach is to have Quartz use a DataSource that is managed by an application server that Quartz is running inside of - by providing JDBCJobStore the JNDI name of the DataSource. For details on the properties, consult the example config files in the `docs/config` folder.

这个难题的最后一部分是设置一个数据源，JDBCJobStore 可以从该数据源连接到数据库。在 Quartz 属性中使用几种不同的方法定义数据源。一种方法是让 Quartz 创建和管理数据源本身（通过提供数据库的所有连接信息）。另一种方法是通过提供 JDBCJobStore 数据源的 JNDI 名称，让 Quartz 使用一个由 Quartz 运行在其内部的应用服务器管理的数据源。有关属性的详细信息，请参阅 `docs/config` 文件夹中的示例配置文件。

To use JDBCJobStore (and assuming you’re using StdSchedulerFactory) you first need to set the JobStore class property of your Quartz configuration to be either org.quartz.impl.jdbcjobstore.JobStoreTX or org.quartz.impl.jdbcjobstore.JobStoreCMT - depending on the selection you made based on the explanations in the above few paragraphs.

要使用 JDBCJobStore（并假设您正在使用 StdSchedulerFactory），首先需要将 Quartz 配置的 JobStore 类属性设置为 org.quartz.impl.jdbcjobstore.JobStoreTX 或 org.quartz.impl.jdbcjobstore.JobStoreCMT（这取决于你基于上面几个段落的解释做出的选择）。

**Configuring Quartz to use JobStoreTx**

```java
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
```

Next, you need to select a DriverDelegate for the JobStore to use. The DriverDelegate is responsible for doing any JDBC work that may be needed for your specific database. StdJDBCDelegate is a delegate that uses `vanilla` JDBC code (and SQL statements) to do its work. If there isn’t another delegate made specifically for your database, try using this delegate - we’ve only made database-specific delegates for databases that we’ve found problems using StdJDBCDelegate with (which seems to be most!). Other delegates can be found in the `org.quartz.impl.jdbcjobstore` package, or in its sub-packages. Other delegates include DB2v6Delegate (for DB2 version 6 and earlier), HSQLDBDelegate (for HSQLDB), MSSQLDelegate (for Microsoft SQLServer), PostgreSQLDelegate (for PostgreSQL), WeblogicDelegate (for using JDBC drivers made by Weblogic), OracleDelegate (for using Oracle), and others.

接下来，您需要为 JobStore 选择一个 DriverDelegate 来使用。驱动委托负责执行特定数据库可能需要的任何 JDBC 工作。StdJDBCDelegate 是一个使用 `普通的` JDBC 代码（和 SQL 语句）来完成其工作的委托。如果没有其他专门为您的数据库制作的委托，请尝试使用这个委托，我们只为使用 StdJDBCDelegate（似乎是最多的）发现问题的数据库制作了特定于数据库的委托。其他委托可以在 `org.quartz.impl.jdbcjobstore` 中找到，或者其子包。其他委托包括 DB2v6Delegate（用于 DB2 版本 6 和更早的版本）、HSQLDBDelegate（用于 HSQLDB）、MSSQLDelegate（用于 Microsoft SQLServer）、PostgreSQLDelegate（用于 PostgreSQL）、WeblogicDelegate（用于使用 Weblogic 制作的 JDBC 驱动程序）、OracleDelegate（用于使用 Oracle）等等。

Once you’ve selected your delegate, set its class name as the delegate for JDBCJobStore to use.

选择了委托之后，将其类名设置为 JDBCJobStore 要使用的委托。

**Configuring JDBCJobStore to use a DriverDelegate**

```java
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
```

Next, you need to inform the JobStore what table prefix (discussed above) you are using.

接下来，您需要通知 JobStore 您正在使用什么表前缀（上面讨论过）。

**Configuring JDBCJobStore with the Table Prefix**

```java
org.quartz.jobStore.tablePrefix = QRTZ_
```

And finally, you need to set which DataSource should be used by the JobStore. The named DataSource must also be defined in your Quartz properties. In this case, we’re specifying that Quartz should use the DataSource name `myDS` (that is defined elsewhere in the configuration properties).

最后，您需要设置 JobStore 应该使用哪个数据源。命名的数据源还必须在 Quartz 属性中定义。在本例中，我们指定 Quartz 应该使用数据源名称 `myDS`（在配置属性的其他地方定义）。

**Configuring JDBCJobStore with the name of the DataSource to use**

```java
org.quartz.jobStore.dataSource = myDS
```

> If your Scheduler is busy (i.e. nearly always executing the same number of jobs as the size of the thread pool, then you should probably set the number of connections in the DataSource to be the about the size of the thread pool + 2.

如果您的调度程序很忙，即几乎总是执行与线程池大小相同数量的任务，那么您可能应该将数据源中的连接数设置为线程池大小+2

> The "org.quartz.jobStore.useProperties" config parameter can be set to "true" (defaults to false) in order to instruct JDBCJobStore that all values in JobDataMaps will be Strings, and therefore can be stored as name-value pairs, rather than storing more complex objects in their serialized form in the BLOB column. This is much safer in the long term, as you avoid the class versioning issues that there are with serializing your non-String classes into a BLOB.

org.quartz.jobStore.useProperties 配置参数可以设置为 true（默认为 false），以便指示 JDBCJobStore，在 JobDataMaps 中的所有值都将是字符串，因此可以以键值对的形式存储，而不是将更复杂的对象以序列化形式存储在 BLOB 列中。从长远来看，这样做要安全得多，因为您可以避免将非字符串类序列化为 BLOB 时出现的类版本控制问题。

## TerracottaJobStore

TerracottaJobStore provides a means for scaling and robustness without the use of a database. This means your database can be kept free of load from Quartz, and can instead have all of its resources saved for the rest of your application.

TerracottaJobStore 提供了一种无需使用数据库就可伸缩和健壮的方法。这意味着您的数据库可以免于 Quartz 的负载，而可以为应用程序的其余部分保存所有资源。

TerracottaJobStore can be ran clustered or non-clustered, and in either case provides a storage medium for your job data that is persistent between application restarts, because the data is stored in the Terracotta server. It’s performance is much better than using a database via JDBCJobStore (about an order of magnitude better), but fairly slower than RAMJobStore.

TerracottaJobStore 可以集群运行，也可以非集群运行，这两种情况都为您的作业数据提供了存储介质，这些数据在应用程序重启之间持久存在，因为这些数据存储在 Terracotta 服务器中。它的性能比通过 JDBCJobStore 使用数据库要好得多（大约好一个数量级），但比 RAMJobStore 要慢得多。

To use TerracottaJobStore (and assuming you’re using StdSchedulerFactory) simply specify the class name org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore as the JobStore class property that you use to configure quartz, and add one extra line of configuration to specify the location of the Terracotta server:

要使用 TerracottaJobStore（假设您使用的是 StdSchedulerFactory），只需指定类名 org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore 作为用于配置 quartz 的 JobStore 类属性，并添加额外的一行配置来指定 Terracotta 服务器的位置：

**Configuring Quartz to use TerracottaJobStore**

```java
org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore
org.quartz.jobStore.tcConfigUrl = localhost:9510
```

More information about this JobStore and Terracotta can be found at http://www.terracotta.org/quartz

关于 JobStore 和 Terracotta 的更多信息可详见 http://www.terracotta.org/quartz

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 8: SchedulerListeners](/Tutorials/Lesson-8)**
- **Next Lesson（下一课）：[Lesson 10: Configuration, Resource Usage and SchedulerFactory](/Tutorials/Lesson-10)**