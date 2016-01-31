---
layout: post
title:  "深入浅出Docker在Mesos中的应用"
date:   2016-01-08 11:50:15
categories: Docker
excerpt: 
---

* content
{:toc}


### 序

Docker和Mesos都是近两年活跃的开源技术，Docker是当前容器的事实标准，Mesos则定位于DCOS，分布式管理数据中心中的计算资源和workload。Mesos被看做Docker生态中容器调度的一员，Mesosphere公司高级副总裁Matt Trifiro，把Docker+Mesos结合比喻是像花生酱和果冻、牛奶和饼干那样完美。究竟为什么二者会结合，结合又对Mesos带来什么好处？下面我来给大家分析下

---

### 我们先看看Mesos做了什么

#### 没有Mesos

先来看看不使用mesos，通常的workload是怎么做的，以hadoop的MR为例。在私有云或公有云上，我们申请了EMR服务，通常会先基于计算部署Hadoop集群，基于Hadoop集群运行我们的分析任务。而对于公有云或数据中心中，类似的workload不同的租户可能会启动很多个实例，也就是说系统中会存储很多的Hadoop的计算集群资源，运行着各自的workload，如下图是2个workload的例子：

![ruby-gems]({{ "/css/pics/201601/hadoop_whiout_mesos.png"}})

传统的方式是典型的结构化的IT架构，它有一个最明显的缺点是资源利用率的问题，上图中2个workload所占用的计算资源在不同时段中可能各不相同，在某些时段中workload1的资源可能负载高，而workload2则可能闲置，而在下个时段可能会相反，如果总体看整个计算资源的利用率，必然是不高的，除非各自都是全负载运行，但这毕竟是极端场景。
究其根本是这种结构化的架构导致的，每个计算集群只能运行单一的应用，当然我们也可以做到将两个workload运行到一起，但这样势必要人为的处理冲突、资源竞争等等问题。（源于架构本身并不是面向云，这其实也是云原生应用要解决的问题）

#### 有了Mesos

上面的问题如何解决，我们看看加了Mesos的变化：

![ruby-gems]({{ "/css/pics/201601/hadoop_with_mesos.png"}})

和之前不同的是，Mesos引入了两个新东西，一个是Framework Scheduler，一个是Executor。先说这个Framework Scheduler，在Mesos的架构中把我们之前的workload叫做Framework，这个Framework不但可以是同一应用的不同工作负载，也可以是不同的应用，例如可以是一个Hadoop分析，也可以是个Spark的机器学习或者是Hama的图形处理任务。Executor则是具体框架执行器，执行框架自身的任务，例如Hadoop应用中Executor会去执行MR的TaskTracker。
除了架构上变化，从Node负载的工作来看，也和之前大不相同，现在的Node上会同时执行两个Framework（workload）的任务，而且两个workload的利用率结合在一起，提升了Node节点上计算资源的整体利用率，从而提升了整个DC计算资源利用率。

### Mesos如何做到的

Mesos之所以能将workload卸载到多台Node运行得益于其架构的两个特点：

#### 1、双层调度

类似于Hadoop的Yarn，Mesos将资源的调度从业务调度中独立出来，Mesos Master负责管理整体计算资源，然后根据资源邀约将资源发给Framework，而Framework自身的Scheduler则只专注于其自身的调度即可，这样将整个DC的workload资源做了整合：

![ruby-gems]({{ "/css/pics/201601/two_level_scheduler.png"}})

#### 2、资源隔离

这第二点才是本文的重点，第一点只是统一管理资源，并分配资源，但真正运行起来没有配额控制可不行，而且还需要解决节点上不同Framework的资源冲突，对于这个问题理所当然的想到了虚拟化技术，但使用VM还是Container呢，从隔离性、轻量级、效率、性能综合考虑，Mesos还是选择了容器，而Docker则被其待如上宾了，我们看看其基于容器隔离的方案：

![ruby-gems]({{ "/css/pics/201601/mesos_with_docker.png"}})

上图中间一层是容器技术，Mesos支持容器插件式的容器框架，其自身带一个容器隔离机，只是基于cgroup做简单隔离，其也考虑到Docker生态的影响力，原生支持了Docker引擎，Mesos需要将其容器隔离方案推向生产环境，而正像Matt Trifiro说的Docker正是其不二之选，利用Docker成熟的技术、运维能力和生态的支持，二者真是完美的组合。

### 参考

[http://mesos.apache.org/](http://mesos.apache.org/ "http://mesos.apache.org/")

[http://www.infoq.com/cn/articles/analyse-mesos-part-03](http://www.infoq.com/cn/articles/analyse-mesos-part-03 "http://www.infoq.com/cn/articles/analyse-mesos-part-03")

[http://cloud.chinabyte.com/vertical/287/13285787.shtml](http://cloud.chinabyte.com/vertical/287/13285787.shtml "http://cloud.chinabyte.com/vertical/287/13285787.shtml")

《Mesos 大数据资源调度于大规模容器运行最佳实践》 -Dharmesh Kakadia

** 原创，引用请注明出处，谢谢。