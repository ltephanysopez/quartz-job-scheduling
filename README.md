# Introduction to Job Scheduling with Quartz
<br>
<br>
## What is Quartz?
Quartz is an open-source job scheduling framework that can be integrated into a wide variety of applications. Applications that incorporate Quartz can reuse jobs from different events and also group multiple jobs for a single event.

### Quartz Object Model
The following is the basic object model for Quartz.

![Quartz Object Model](https://github.com/ltephanysopez/quartz-job-scheduling/blob/master/docs/quartz-object-model.png)

There are several classes in the diagram above, but for simplicity, we'll focus on the main classes that we need to interact with to product our scheduler module.

&nbsp;
<br>

## Jobs

A Job is an interface implemented by components to be executed by the scheduler with the basic contract:
```
   execute(JobExecutionContext context);
```

Every Quartz job *must* implement this method.

The following is a simple sequence diagram that shows how classes interact with each other to schedule and start a job.

![Job Scheduling Architecture](https://github.com/ltephanysopez/quartz-job-scheduling/blob/master/docs/job-scheduling-arch.png)

The scheduler is an interface that defines the contract for the Quartz Scheduler and is the class responsible for scheduling and running the jobs.


### Job Scheduling Module
The purpose of the Job Scheduling Module is to provide an easy to use interface for all microservices.

### JobScheduler
The JobScheduler is the facade used by all microservices to create and schedule jobs. JobBuilder is used to create jobs and the TriggerBuilder creates triggers via Quartz.


&nbsp;
To make a Java class executable, we can implement the `org.quartz.job` interface. The class below overrides the execute(JobExecutionContext context) method with a single output statement.

```
import java.util.Date;
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class SimpleQuartzJob implements Job {
   public SimpleQuartzJob() {

   }
   public void execute(JobExecutionContext context) throws JobExecutionException {
      System.out.println("In SimpleQuartzJob - executing its JOB at " + new Date() + " by " + context.getTrigger().getName());
   }
}
```
&nbsp;
<br>

`JobExecutionContext` provides the runtime context around the job instance, giving access to the scheduler and trigger.

`JobDetail` stores to job's listeners, group, data map, description, and other properties of the job.

The execute method in this example takes the `JobExecutionContext` object as an argument. Quartz separates the execution and the surrounding state of a job by placing the state in a `JobDetail` object and having the   `JobDetail` constructor initiate an instance of a job.


### JobAttempt
The JobAttempt is a custom class that implements the Job interface for Quartz. When a job is executed, it will ‘register’ the job attempt within the Quartz JobDataMap.

### JobExecutionContext
Context bundle containing handles to various environment information, that is given to JobDetail instance as it is executed, and to a Trigger instance after the execution completes.

### JobExecutionException
The JobExecutionException is the implementation that makes an exception while executing a job.

&nbsp;

## Triggers
There are two types of triggers that you are able to use with Quartz.

### SimpleTrigger
Trigger that is used to execute a Job at a given moment in time, and optionally repeated at a specified interval. For instance, with SimpleTrigger you can have the trigger fire at exactly 11:23:58 AM on September 3, 2018, or if you want it to fire at that time, and then repeat 10 more times, every 30 seconds.
The properties of a SimpleTrigger include a start-time, end-time, repeat count, and repeat interval.

&nbsp;
&nbsp;
<br>

The following will fire for a specific moment in time, with no repeats:
```
      SimpleTrigger trigger = (SimpleTrigger) newTrigger() {
         .withIdentity(“trigger1”, “group1”)
         .startAt(myStartTime) // some Date
         .forJob(“job1”, “group1”) //identify jog with name, group strings
         .build();
      }
```
&nbsp;
The following will fire for a specific moment in time, then repeat every 30 seconds ten times.
```
      trigger = newTrigger() {
         .withIdentity(“trigger3”, “group1”)
         .startAt(myTimeToStartFiring) // if a start time is not given, “now” is implied
         .withSchedule(simpleSchedule()
            .withIntervalInSeconds(10)
            .withRepeatCount(30)) // note that 30 repeats will give a total of 31 firings
         .forJob(myJob) //identify jog with handle to its JobDetail itself
         .build();
      }

```
&nbsp;
Lastly, the following will fire now, then repeat every five minutes, until the hour 22:00.
```
      trigger = newTrigger() {
         .withIdentity(“trigger7”, “group1”)
         .withSchedule(simpleSchedule()
            .withIntervalInMinutes(5)
            .repeatForever())
         .endAt(dateOf(22, 0, 0))
         .build();
      }
```

&nbsp;

### CronTrigger
Allows  job-firing schedule that recurs based on calendar-like notions, rather than on the exactly specified intervals of SimpleTrigger. For instance, with CronTrigger you can specify firing-schedules such as “every Wednesday at 12PM” or “every weekday at 8:00am”.

Like SimpleTrigger, CronTrigger has a startTime which specifies when the schedule is in force, and an optional endTime that specifies when the schedule should be discontinued.

Cron-Expressions are strings made up of seven sub-expressions, that describe individual details of the schedule, and used to configure instances of CronTrigger. The sub-expressions are separated with white-space and represent seconds, minutes, hours, day-of-month, month, day-of-week, and year.

&nbsp;
&nbsp;
&nbsp;
<br>

The following will fire every day at 10:42am:
```
      trigger = newTrigger() {
         .withIdentity(“trigger3”, “group1”)
         .withSchedule(cronSchedule(“0 42 10 * * ?”))
         .forJob(myJobKey)
         .build();
      }
```
or, it can also be written:
```
      trigger = newTrigger() {
         .withIdentity(“trigger3”, “group1”)
         .withSchedule(dailyAtHourAndMinute(10, 42))
         .forJob(myJobKey)
         .build();
      }
```
&nbsp;
The following will fire every Wednesday at 10:42 am, in a timezone other than the system’s default:
```
      trigger = newTrigger() {
         .withIdentity(“trigger3”, “group1”)
         .withSchedule(weeklyOnDayAndHourAndMinute(DateBuilder.WEDNESDAY, 10, 42))
         .forJob(myJobKey)
         .inTimeZone(TimeZone.getTimeZone(“America/Los_Angeles”))
         .build();
      }
```
Additionally, it can also be written as:
```
      trigger = newTrigger() {
         .withIdentity(“trigger3”, “group1”)
         .withSchedule(“0 42 10 ? * WED”))
         .inTimeZone(TimeZone.getTimeZone(“America/Los_Angeles”))
         .forJob(myJobKey)
         .build();
      }
```
&nbsp;

## How To Add The Scheduling Module to a Microservice

With Maven, adding in the Scheduling module is as simple as including the reference to this module in your `pom.xml` file.

```
<dependency>
	<groupId>${project.groupId}</groupId>
	<artifactId>example-scheduling</artifactId>
	<version>1.3.0-SNAPSHOT</version>
<dependency>
```

### Creating a New Job
When creating a new Job, you'll need to decide whether or not your job should support multiple attempts. In order to provide the ability for multiple attempts, you'll have to inherit the correct JobAttempt base class. These are as follows:

### JobAttempt
This base class does not support multiple attempts. If a job that extends this class directly fails, no new attempt will be made.

```
public class SampleJob extends JobAttempt {
   private static final CloudLogger logger = CloudLogger.getLogger(SampleJob.class.getName());

   @Override
   protected void executeAttempt(JobExecutionContext context) {
      String message = String.format("Sample Job has run for %d times", super.getAttemptCount(context));
      System.out.println(message);
   }
}
```

### RetryOnceJob
This base class supports an extra attempt. If a ob extends this class directly and fails, the scheduler will create a new job and reuse the JobDataMap using the cool down period to delay the execution slightly. Here is a basic job that uses this class:

```
public class SampleJob extends RetryOnceJob {
   private static final CloudLogger logger = CloudLogger.getLogger(SampleJob.class.getName());

   @Override
   protected void executeAttempt(JobExecutionContext context) {
      String message = String.format("Sample Job has run for %d times", super.getAttemptCount(context));
      System.out.println(message);
   }

   @Override
   public int getCoolDownIntervalInSeconds() {
      return 30;
   }
}
```

<br>
<br>
## RetryStrategies

### DefaultRetryStrateg
Does not reattempt to schedule a job, it only removed the failed Job from the scheduler.

### RetryOnceStrategy
Strategy that will schedule the failed Job to fire again using the cool down period (seconds). In essence, if the cool down period is 30 seconds and the current time is 10:30:00, the job will be scheduled to run at 10:30:30.

### RecurringRetryStrategy
Strategy that will run the job several times to a maximum as defined by the getMaximumAttempts() property. Each time the job is scheduled to run again the strategy will use the cool down period * the number of attempts as the cool down period for the next attempt. Using the algorithm the cool down period with each new attempt is increased.

### RetryJob
An interface that identifies the contract for a JobAttempt to be scheduled for another attempt should the job fail. A “RetryJob” can be either tried again via the RetryOnceJob or can have several attempts via the RecurringRetryJob subclass.

&nbsp;
&nbsp;
<br>

## Resources
[Quartz Documentation](http://www.quartz-scheduler.org/documentation/)
&nbsp;
[Quartz Tutorials](http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/)
