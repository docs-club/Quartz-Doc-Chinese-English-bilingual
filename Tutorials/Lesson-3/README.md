# Lesson 3: More About Jobs and Job Details

> 原文地址：http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/tutorial-lesson-03.html

As you saw in Lesson 2, Jobs are rather easy to implement, having just a single ‘execute’ method in the interface. There are just a few more things that you need to understand about the nature of jobs, about the execute(..) method of the Job interface, and about JobDetails.

正如你在第二课中看到的，作业非常容易实现，接口中只有一个 `execute` 方法。关于作业的性质、作业接口的 `execute(..)` 方法和 JobDetails，还有一些事情需要了解。

While a job class that you implement has the code that knows how do do the actual work of the particular type of job, Quartz needs to be informed about various attributes that you may wish an instance of that job to have. This is done via the JobDetail class, which was mentioned briefly in the previous section.

虽然你实现的作业类具有完成特定类型作业的代码，但需要告诉 Quartz 你可能希望该作业实例具有的各种属性。这是通过 JobDetail 类完成的，在前一节中简要提到过。

JobDetail instances are built using the JobBuilder class. You will typically want to use a static import of all of its methods, in order to have the DSL-feel within your code.

JobDetail 实例是使用 JobBuilder 类构建的。你通常希望使用所有方法的静态导入，以便在代码中具有 DSL 的感觉。

```java
import static org.quartz.JobBuilder.*;
```

Let’s take a moment now to discuss a bit about the ‘nature’ of Jobs and the life-cycle of job instances within Quartz. First lets take a look back at some of that snippet of code we saw in Lesson 1:

现在，让我们花一点时间来讨论一下 Quartz 中作业的 `本质` 和作业实例的生命周期。首先让我们回顾一下我们在第一课中看到的代码片段：

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

Now consider the job class `HelloJob` defined as such:

现在将作业类 HelloJob 定义成这样：

```java
  public class HelloJob implements Job {

    public HelloJob() {
    }

    public void execute(JobExecutionContext context) throws JobExecutionException{
      System.err.println("Hello!  HelloJob is executing.");
    }
  }
```

Notice that we give the scheduler a JobDetail instance, and that it knows the type of job to be executed by simply providing the job’s class as we build the JobDetail. Each (and every) time the scheduler executes the job, it creates a new instance of the class before calling its execute(..) method. When the execution is complete, references to the job class instance are dropped, and the instance is then garbage collected. One of the ramifications of this behavior is the fact that jobs must have a no-argument constructor (when using the default JobFactory implementation). Another ramification is that it does not make sense to have state data-fields defined on the job class - as their values would not be preserved between job executions.

请注意，我们为调度程序提供了一个 JobDetail 实例，并且在构建 JobDetail 时，只需提供作业的 Class 对象，调度程序就知道要执行的作业类型。每次调度程序执行作业时，它都会在调用 `execute(..)` 方法之前创建一个类的新实例。执行完成后，将删除对作业类实例的引用，然后对实例进行垃圾收集。这种行为的一个后果是 jobs 必须有一个无参数构造函数（使用默认的 JobFactory 实现）。另一方面，在作业类上定义状态数据字段没有意义，因为它们的值不会在作业执行之间保留。

You may now be wanting to ask `how can I provide properties/configuration for a Job instance?` and `how can I keep track of a job’s state between executions?` The answer to these questions are the same: the key is the JobDataMap, which is part of the JobDetail object.

你现在可能想问 `我如何为作业实例提供属性或配置？` 以及 `如何在执行时跟踪任务的状态？` 这些问题的答案是相同的：关键在于 JobDataMap，它是 JobDetail 对象的一部分。

## JobDataMap

The JobDataMap can be used to hold any amount of (serializable) data objects which you wish to have made available to the job instance when it executes. JobDataMap is an implementation of the Java Map interface, and has some added convenience methods for storing and retrieving data of primitive types.

JobDataMap 可用于保存你希望在作业实例执行时提供给它的任意数量的（可序列化的）数据对象。JobDataMap 是 Java Map 接口的一个实现，它添加了一些便利的方法来存储和检索基本数据类型的数据。

Here’s some quick snippets of putting data into the JobDataMap while defining/building the JobDetail, prior to adding the job to the scheduler:

在将作业添加到调度程序之前，有一个在定义或构建 JobDetail 时将数据放入 JobDataMap 的例子：

```java
  // define the job and tie it to our DumbJob class
  JobDetail job = newJob(DumbJob.class)
      .withIdentity("myJob", "group1") // name "myJob", group "group1"
      .usingJobData("jobSays", "Hello World!")
      .usingJobData("myFloatValue", 3.141f)
      .build();
```

Here’s a quick example of getting data from the JobDataMap during the job’s execution:

下面是一个在作业执行过程中从 JobDataMap 获取数据的简单示例：

```java
public class DumbJob implements Job {

    public DumbJob() {
    }

    public void execute(JobExecutionContext context) throws JobExecutionException
    {

      JobKey key = context.getJobDetail().getKey();

      JobDataMap dataMap = context.getJobDetail().getJobDataMap();

      String jobSays = dataMap.getString("jobSays");
      float myFloatValue = dataMap.getFloat("myFloatValue");

      // 输出内容为 Instance group1.myJob of DumbJob says: Hello World!, and val is: 3.141
      System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
  }
```

If you use a persistent JobStore (discussed in the JobStore section of this tutorial) you should use some care in deciding what you place in the JobDataMap, because the object in it will be serialized, and they therefore become prone to class-versioning problems. Obviously standard Java types should be very safe, but beyond that, any time someone changes the definition of a class for which you have serialized instances, care has to be taken not to break compatibility. Optionally, you can put JDBC-JobStore and JobDataMap into a mode where only primitives and strings are allowed to be stored in the map, thus eliminating any possibility of later serialization problems.

如果使用持久的 JobStore（在本教程的 JobStore 一节中讨论），在决定将什么放在 JobDataMap 中时应该格外小心，因为其中的对象将被序列化，因此它们很容易出现类版本控制问题。显然，标准 Java 类型应该是非常安全的，但除此之外，任何时候有人更改你已经序列化了实例的类的定义时，都必须注意不要破坏兼容性。你还可以选择将 JDBC-JobStore 和 JobDataMap 置于只允许在映射中存储基本类型和字符串的模式中，从而消除以后出现序列化问题的任何可能性。

If you add setter methods to your job class that correspond to the names of keys in the JobDataMap (such as a setJobSays(String val) method for the data in the example above), then Quartz’s default JobFactory implementation will automatically call those setters when the job is instantiated, thus preventing the need to explicitly get the values out of the map within your execute method.

如果你添加 setter 方法工作类，对应键的名字 JobDataMap（如上面的示例中的 `setJobSays(String val)` 方法），然后 Quartz 的默认 JobFactory 实现将自动调用这些 setter 当工作被实例化，从而防止需要显式地获得值的地图在你的执行方法。

Triggers can also have JobDataMaps associated with them. This can be useful in the case where you have a Job that is stored in the scheduler for regular/repeated use by multiple Triggers, yet with each independent triggering, you want to supply the Job with different data inputs.

触发器也可以与 JobDataMaps 相关联。如果你有一个作业存储在调度程序中，供多个触发器定期/重复使用，但对于每个独立的触发器，你希望为该作业提供不同的数据输入，那么这可能会很有用。

The JobDataMap that is found on the JobExecutionContext during Job execution serves as a convenience. It is a merge of the JobDataMap found on the JobDetail and the one found on the Trigger, with the values in the latter overriding any same-named values in the former.

作业执行期间在 JobExecutionContext 上找到的 JobDataMap 可以提供方便。它合并了在 JobDetail 上找到的 JobDataMap 和在触发器上找到的 JobDataMap，后者中的值覆盖了前者中的任何同名值。

Here’s a quick example of getting data from the JobExecutionContext’s merged JobDataMap during the job’s execution:

下面是一个在作业执行过程中从合并到 JobExecutionContext 的 JobDataMap 获取数据的示例：

```java
public class DumbJob implements Job {

    public DumbJob() {
    }

    public void execute(JobExecutionContext context) throws JobExecutionException{
      JobKey key = context.getJobDetail().getKey();

      JobDataMap dataMap = context.getMergedJobDataMap();  // Note the difference from the previous example

      String jobSays = dataMap.getString("jobSays");
      float myFloatValue = dataMap.getFloat("myFloatValue");
      ArrayList state = (ArrayList)dataMap.get("myStateData");
      state.add(new Date());

      System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
  }
```

Or if you wish to rely on the JobFactory `injecting` the data map values onto your class, it might look like this instead:

或者，如果你希望依赖于 JobFactory `注入` 数据映射值到你的类，它可能会像这样：

```java
  public class DumbJob implements Job {

    String jobSays;
    float myFloatValue;
    ArrayList state;

    public DumbJob() {
    }

    public void execute(JobExecutionContext context) throws JobExecutionException{
      JobKey key = context.getJobDetail().getKey();

      JobDataMap dataMap = context.getMergedJobDataMap();  // Note the difference from the previous example

      state.add(new Date());

      System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }

    public void setJobSays(String jobSays) {
      this.jobSays = jobSays;
    }

    public void setMyFloatValue(float myFloatValue) {
      myFloatValue = myFloatValue;
    }

    public void setState(ArrayList state) {
      state = state;
    }

  }
```

You’ll notice that the overall code of the class is longer, but the code in the execute() method is cleaner. One could also argue that although the code is longer, that it actually took less coding, if the programmer’s IDE was used to auto-generate the setter methods, rather than having to hand-code the individual calls to retrieve the values from the JobDataMap. The choice is yours.

你将注意到，类的整体代码更长，但 `execute()` 方法中的代码更干净。有人可能会说，虽然代码更长，但实际上需要的编码更少，如果程序员的 IDE 用于自动生成 setter 方法，就不必手工编写单个调用来从 JobDataMap 检索值。选择权在你。

## Job Instances

Job 实例

Many users spend time being confused about what exactly constitutes a `job instance`. We’ll try to clear that up here and in the section below about job state and concurrency.

许多用户花时间搞不清楚 `作业实例` 的确切构成。我们将尝试在这里和下面关于作业状态和并发性的一节中澄清这一点。

You can create a single job class, and store many ‘instance definitions’ of it within the scheduler by creating multiple instances of JobDetails - each with its own set of properties and JobDataMap - and adding them all to the scheduler.

你可以创建单个作业类，并在调度器中存储它的许多 `实例定义`，方法是创建多个 JobDetails 实例（每个实例都有自己的一组属性和 JobDataMap），并将它们全部添加到调度器。

For example, you can create a class that implements the Job interface called `SalesReportJob`. The job might be coded to expect parameters sent to it (via the JobDataMap) to specify the name of the sales person that the sales report should be based on. They may then create multiple definitions (JobDetails) of the job, such as `SalesReportForJoe` and `SalesReportForMike` which have `joe` and `mike` specified in the corresponding JobDataMaps as input to the respective jobs.

例如，你可以创建一个实现名为 `SalesReportJob` 的作业接口的类。可以对作业进行编码，使其期望收到（通过 JobDataMap）发送的参数，以指定销售报告应基于的销售人员的名称。然后，它们可以创建作业的多个定义（JobDetails），例如 `SalesReportForJoe` 和 `SalesReportForMike`，它们在相应的 JobDataMaps 中指定 `joe` 和 `mike` 作为各自作业的输入。

When a trigger fires, the JobDetail (instance definition) it is associated to is loaded, and the job class it refers to is instantiated via the JobFactory configured on the Scheduler. The default JobFactory simply calls newInstance() on the job class, then attempts to call setter methods on the class that match the names of keys within the JobDataMap. You may want to create your own implementation of JobFactory to accomplish things such as having your application’s IoC or DI container produce/initialize the job instance.

当触发器触发时，将加载它关联的 JobDetail（实例定义），并通过调度程序上配置的 JobFactory 实例化它引用的作业类。默认的 JobFactory 只是在作业类上调用 newInstance()，然后尝试调用类上与 JobDataMap 内键名匹配的 setter 方法。你可能希望创建自己的 JobFactory 实现来完成一些事情，比如让应用程序的 IoC 或 DI 容器生成/初始化作业实例。

In `Quartz speak`, we refer to each stored JobDetail as a `job definition` or `JobDetail instance`, and we refer to a each executing job as a `job instance` or `instance of a job definition`. Usually if we just use the word `job` we are referring to a named definition, or JobDetail. When we are referring to the class implementing the job interface, we usually use the term `job class`.

## Job State and Concurrency

在 `Quartz speak` 中，我们将每个存储的 JobDetail 称为 `作业定义` 或 `JobDetail 实例` ，将每个正在执行的作业称为 `作业实例` 或 `作业定义实例`。通常，如果我们只使用 `job` 这个词，我们指的是一个命名定义，即 JobDetail。当我们引用实现作业接口的类时，我们通常使用术语 `作业类`。

Now, some additional notes about a job’s state data (aka JobDataMap) and concurrency. There are a couple annotations that can be added to your Job class that affect Quartz’s behavior with respect to these aspects.

现在，一些关于作业状态数据（又名 JobDataMap）和并发性的额外注意事项。可以将一些注解添加到作业类中，这些注解会影响 Quartz 在这些方面的行为。

@DisallowConcurrentExecution is an annotation that can be added to the Job class that tells Quartz not to execute multiple instances of a given job definition (that refers to the given job class) concurrently.

@DisallowConcurrentExecution 是一个可以添加到作业类中的注解，它告诉 Quartz 不要并发地执行给定作业定义（引用给定作业类）的多个实例。

Notice the wording there, as it was chosen very carefully. In the example from the previous section, if `SalesReportJob` has this annotation, than only one instance of `SalesReportForJoe` can execute at a given time, but it can execute concurrently with an instance of `SalesReportForMike`. The constraint is based upon an instance definition (JobDetail), not on instances of the job class. However, it was decided (during the design of Quartz) to have the annotation carried on the class itself, because it does often make a difference to how the class is coded.

注意这里的措辞，因为它是精心挑选的。在上一节的示例中，如果 `SalesReportJob` 具有此注解，那么在给定的时间只有 `SalesReportForJoe` 的一个实例可以执行，但它可以与 `SalesReportForMike` 的实例同时执行。约束基于实例定义（JobDetail），而不是 job 类的实例。然而，（在 Quartz 的设计期间）决定在类本身上进行注解，因为这通常会对类的编码方式产生影响。

@PersistJobDataAfterExecution is an annotation that can be added to the Job class that tells Quartz to update the stored copy of the JobDetail’s JobDataMap after the execute() method completes successfully (without throwing an exception), such that the next execution of the same job (JobDetail) receives the updated values rather than the originally stored values. Like the @DisallowConcurrentExecution annotation, this applies to a job definition instance, not a job class instance, though it was decided to have the job class carry the attribute because it does often make a difference to how the class is coded (e.g. the ‘statefulness’ will need to be explicitly ‘understood’ by the code within the execute method).

@PersistJobDataAfterExecution 是一个注解，可以添加到工作类告诉 Quartz 更新存储复制 JobDetail JobDataMap 的 execute() 方法成功完成后（没有抛出异常），这样下一个执行同样的工作（JobDetail）收到更新后的值而不是原先存储的值。像@DisallowConcurrentExecution 注解，这适用于工作定义实例，类实例不是一份工作，虽然这是决定工作类的属性，因为它经常改变类是如何编码的（例如，`有状态性` 需要显式地 `理解` 的代码在执行方法）。

If you use the @PersistJobDataAfterExecution annotation, you should strongly consider also using the @DisallowConcurrentExecution annotation, in order to avoid possible confusion (race conditions) of what data was left stored when two instances of the same job (JobDetail) executed concurrently.

如果使用 @PersistJobDataAfterExecution 注解，那么还应该强烈考虑使用 @DisallowConcurrentExecution 注解，以避免在同时执行同一个作业（JobDetail）的两个实例时可能产生的混淆（竞争条件）。

## Other Attributes Of Jobs

Jobs 的其他属性

Here’s a quick summary of the other properties which can be defined for a job instance via the JobDetail object:

以下是可以通过 JobDetail 对象为作业实例定义的其他属性的快速摘要：

- Durability - if a job is non-durable, it is automatically deleted from the scheduler once there are no longer any active triggers associated with it. In other words, non-durable jobs have a life span bounded by the existence of its triggers.

持久性，如果一个作业是非持久性的，一旦不再有任何与之相关的活动触发器，它将自动从调度程序中删除。换句话说，非持久作业的生命周期取决于其触发器的存在。

- RequestsRecovery - if a job `requests recovery`, and it is executing during the time of a ‘hard shutdown’ of the scheduler (i.e. the process it is running within crashes, or the machine is shut off), then it is re-executed when the scheduler is started again. In this case, the JobExecutionContext.isRecovering() method will return true.

RequestsRecovery，如果一个作业 `请求恢复`，并且它是在调度器 `硬关闭` 期间执行的（即它正在运行的进程崩溃了，或者机器被关闭），那么当调度器再次启动时，它会重新执行。在本例中，JobExecutionContext.isRecovering() 方法将返回 true。

## JobExecutionException

Finally, we need to inform you of a few details of the Job.execute(..) method. The only type of exception (including RuntimeExceptions) that you are allowed to throw from the execute method is the JobExecutionException. Because of this, you should generally wrap the entire contents of the execute method with a ‘try-catch’ block. You should also spend some time looking at the documentation for the JobExecutionException, as your job can use it to provide the scheduler various directives as to how you want the exception to be handled.

最后，我们需要通知你关于 `Job.execute(..)` 方法的一些细节。允许从 execute 方法抛出的唯一类型的异常（包括 RuntimeExceptions）是 JobExecutionException。因此，通常应该用 `try-catch` 块包装 execute 方法的整个内容。你还应该花一些时间查看关于 JobExecutionException 的文档，因为你的作业可以使用它向调度程序提供关于如何处理异常的各种指示。

---

**[Back to contents of the chapter（返回章节目录）](/Tutorials)**

- **Previous Lesson（上一课）：[Lesson 2: The Quartz API, and Introduction to Jobs And Triggers](/Tutorials/Lesson-2)**
- **Next Lesson（下一课）：[Lesson 4: More About Triggers](/Tutorials/Lesson-4)**
