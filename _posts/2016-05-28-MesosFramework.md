---
layout: post
title: The Framework Of Mesos
description: "Learning Mesos"
reading_time: true
modified: 2016-05-28
tags: [Mesos Framework]
image:
  feature: abstract-10.jpg

---

<strong>Note：This is a Beginning for Understanding the Framework of Mesos.</strong>  
<h1>Mesos Frameworks</h1>
<h2>Frameworks</h2>
   <strong>The Architecture of Mesos:</strong>  
![mesos-architecture.jpg](https://raw.githubusercontent.com/Sun-zhe/sun-zhe.github.io/master/images/mesos-architecture.jpg)  
   Apache Mesos is offen explained as being a kernel for data-centre, meaning that cluster resources (CPU, GPU, RAM, Disk...) are tracked ad offered to "user space" programs(i.e Frameworks) to do computations on the cluster.
   From the above figure <Mesos-Architecture>, we have one elected master that track resources on slaves and offer these resources to frameworks. Frameworks can take the offers and use this to launch a task on the slaves. These tasks run on an executor usually the built-in. Command Executor, that manages the task for us on the machine. So the framwork itself is actually a type of scheduler.  

<h3>Register The Framework</h3>
   One of the first things that a Mesos Framework should do is to register itself with the elected Mesos master so that it can start receiving resource offers. These offers then need to end up in our scheduler implementation. In Java we can use the <strong>MesosSchdulerDriver</strong> to take care off this wiring for us. we set our new <strong>MesosSchedulerDriver</strong> up by passing in a reference to our scheduler and by telling it everything it needs to know to communicate and register with the Mesos master.  

   If we want wo develop our own Framework, the Library and the Interface below need to be included.  
<strong>-</strong> *The Scheduler Library Of Mesos*  
![SchedulerLibraryOfMesos.png](https://raw.githubusercontent.com/Sun-zhe/sun-zhe.github.io/master/images/mesos/SchedulerLibraryOfMesos.png)

<strong>-</strong> *The Executor Library Of Mesos*  
![ExecutorLibraryOfMesos.png](https://raw.githubusercontent.com/Sun-zhe/sun-zhe.github.io/master/images/mesos/ExecutorLibraryOfMesos.png)

<strong>-</strong> *The FrameworkScheduler Interface*  
![SchedulerInterface.png](https://raw.githubusercontent.com/Sun-zhe/sun-zhe.github.io/master/images/mesos/SchedulerInterface.png)
  
<strong>-</strong> *The FrameworkExecutor Interface*  
![ExecutorInterface.png](https://raw.githubusercontent.com/Sun-zhe/sun-zhe.github.io/master/images/mesos/ExecutorInterface.png)

<strong>-</strong> *Run FrameworkScheduler*  
![runFramework.png](https://raw.githubusercontent.com/Sun-zhe/sun-zhe.github.io/master/images/mesos/runFramework.png)


<strong>-</strong> *Run FrameworkExecutor*  
![runExecutor.png](https://raw.githubusercontent.com/Sun-zhe/sun-zhe.github.io/master/images/mesos/runExecutor.png)

   The first thing a Framework insterting into Mesos is to prepare two stuffs:

   <strong>-</strong> One is the Scheduler of Framework
   FrameworkScheduler, a core class to schedule the resources achieved from Mesos-Master and execute the tasks that match the resources. FrameworkScheduler insteract with Mesos-Master trough MesosSchedulerDriver.

   <strong>-</strong> The other is the Executor of Framework
   Execute the tasks that Frameworks deploy on Mesos-Slave.

   When we create a FrameworkScheduler instance, with the (FrameworkID, MasterID, FrameworkScheduler) create SchedulerDriver instance. The instance of SchedulerDriver use for two things:

   <strong>-</strong> Manage the FrameworkScheduler all the life-cycle, include <strong>start, stop, wait...</strong>







<link rel="stylesheet" href="/css/backtop.css">  <!-- Back Top -->
<script type="text/javascript" src="/js/backtop.js"></script>  <!-- Back Top -->

<div id="back-top">
  <a href="#top" title="Back Top"></a>
</div>




