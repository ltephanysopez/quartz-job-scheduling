# Introduction to Job Scheduling with Quartz

## What is Quartz?
Quartz is an open-source job scheduling framework that can be integrated into a wide variety of applications. Applications that incorporate Quartz can reuse jobs from different events and also group multiple jobs for a single event.

### Quartz Object Model
The following is the basic object model for Quartz.

![Quartz Object Model](https://github.com/ltephanysopez/quartz-job-scheduling/blob/master/docs/quartz-object-model.png)

There are several classes in the diagram above, but for simplicity, we'll focus on the main classes that we need to interact with to product our scheduler module.

### How Jobs Are Scheduled

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

##### JobScheduler
The JobScheduler is the facade used by all microservices to create and schedule jobs. JobBuilder is used to create jobs and the TriggerBuilder creates triggers via Quartz.

##### JobAttempt
The JobAttempt is a custom class that implements the Job interface for Quartz. When a job is executed, it will ‘register’ the job attempt within the Quartz JobDataMap.

##### JobExecutionContext
Context bundle containing handles to various environment information, that is given to JobDetail instance as it is executed, and to a Trigger instance after the execution completes.

##### JobExecutionException
The JobExecutionException is the implementation that makes an exception while executing a job.


### How To Add The Scheduling Module to a Microservice

With Maven, adding in the Scheduling module is as simple as including the reference to this module in your `pom.xml` file.

```
<dependency>
	<groupId>${project.groupId}</groupId>
	<artifactId>example-scheduling</artifactId>
	<version>1.3.0-SNAPSHOT</version>
<dependency>
```
