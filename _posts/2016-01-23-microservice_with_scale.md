---
layout: post
title:  "从伸缩模型看云上的微服务架构"
date:   2016-01-23 20:15:10
categories: docker
excerpt: 
---

* content
{:toc}


### 序

最近一直思考到底SOA和微服务的区别是什么，微服务的本质又是什么？网上也找了很多资料，也分析了Martin Fowler那段经典的定义，但还是没有找到答案。也许微服务仅仅是一个模式，试图仅从理论上说清楚并不是明智之举，还是需要看看当前的一些微服务的实践，才能明白其本质。今天我们换个角度，找几个实际例子来看看微服务架构应该长个什么样子。

---

如果对微服务感兴趣，谷歌或度娘上，很容易会搜到关于微服务模式的文章：

[http://microservices.io/articles/scalecube.html](http://microservices.io/articles/scalecube.html "http://microservices.io/articles/scalecube.html")


这里面提到了一个很有趣的模型，Scale Cube，此模型源于一本书《The Art of Scalability》，600多页的专著专门讲述企业级Web伸缩与架构，感兴趣可以网上下个电子版拜读下。这本书提出的一个重要模型概念：

![ruby-gems]({{ "/css/pics/201601/scaling_dimensions.jpg"}})   
** 图片来源：[http://microservices.io/articles/scalecube.html](http://microservices.io/articles/scalecube.html "http://microservices.io/articles/scalecube.html")

简单说，作者认为伸缩有3个维度：   
- X，副本级的伸缩，解决高可用问题   
- Y，分片级的伸缩，解决分布式调度问题   
- Z，功能级的解耦，区别于X和Y，解决系统级能力的伸缩   

---

便于理解，我们举几个例子，来看看这几个维度的伸缩应用。

#### 1. X、Y

先看X和Y，这类伸缩传统IT软件架构较为常见，我们看看应用较多的NoSQL数据库MongoDB的数据节点部署架构：

![ruby-gems]({{ "/css/pics/201601/mongodb_deploy.png"}})

MongoDB架构显然充分考虑到其伸缩扩展能力，支持从数据集和切片的2个维度的扩展：**数据集**，是对其存储数据的多份副本的拷贝，用来提高其高可用能力。**切片**，是将数据分片存放于不同物理节点中，以提供对大规模数据并发访问场景能力。
很容易理解上图正好对应了Scale Cube模型的X和Z轴的扩展。

#### 2. Z

这个Z轴的伸缩，实际正式微服务要解决的问题，我们也看几个例子

#### 2.1 OSGI

OSGi规范作为JAVA模块化的事实标准，目前也在探索微服务领域，见[我上一篇blog](http://mervinsun.github.io/2016/01/16/OSGI-enRoute/ "我上一篇blog")
目前Apache Karaf已经联合OSGi技术推出其DevOps栈(Dev:OSGi,Ops:Karaf),Karaf之上将许多原来JAVA库封装为微服务形式发布，你可以基于这些打造基于微服务架构的JAVA WEB应用了：

![ruby-gems]({{ "/css/pics/201601/Karaf_framework.png"}})   
**图片来源：[http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf](http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf "http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf")

当然你会问，此方式和我们原来的JAR的方式有何不同，最大不同是其真正使用OSGi架构实现了应用，用OSGi之父Peter Kriens的话说“100% the OSGi”，每个服务（组件）都是可以动态添加、可以热插拔，并且单独升级的。很遗憾虽然我们之前也做过OSGi架构的应用，但没有真正做到这一点，所以我们也就没有真正体会到它带给我们的优势，真心地期待Karaf能做到：）。

#### 2.2 OpenStack

即使没有用过OSGi，也没关系，我们应该知道OpenStack。OpenStack是个典型的SOA架构的成功案例，其组件内部完全使用了消息总线机制，组件间又用了REST API，这一点很像微服务，所以目前OpenStack自身是否是微服务存在争议，这一点我们后来再看。现在要说的是OpenStack有个Kolla项目，试图将OpenStack卸载到Docker容器中运行：

![ruby-gems]({{ "/css/pics/201601/kolla.png"}})   
** 图片来源：[http://ink.csdn.net/articles/show/55f1139d1b8895680f66ac5f](http://ink.csdn.net/articles/show/55f1139d1b8895680f66ac5f "http://ink.csdn.net/articles/show/55f1139d1b8895680f66ac5f")

一切看起来很美好，将OpenStack组件都解耦在容器上部署了，我们要计算的能力，只需要部署个Nova,要存储块能力，就拉来个Cinder。但我们不禁要问，如果OpenStack自身就是微服务的，为什么多此一举？（我们知道容器是实现微服务架构一个最佳技术实践）

这其实和微服务架构的本质有关系，根据总结网上各位大师牛人的定义，我们有个较为一致的结论，评价一个架构是否是微服务架构，有个重要的标准
- 服务间是否解耦，是否支持服务的独立部署和升级

而装过OpenStack组件的人，应该知道OpenStack虽然组件间解耦，但安装并非完全做到了只一点，例如要安装一个Nova，如果你系统中没有装KeyStone和RabbitMQ（或其它AMQP组件），那么是无法安装成功的（也可能是作者水平问题，如有问题，请大家指正哈）；另外OpenStack组件间的解耦也并不彻底，很多时候存在组件间对API版本的依赖和对公共库的依赖问题，所以要想用一个L版本的组件跑在一个K版本上并且不出问题并不是容易事。所以基于以上问题，个人认为OpenStack不能算真正意义上的微服务。

#### 2.3 AWS

我们再看看本文的重点，2015年AWS动作频频，推出了两个重要的服务Lambda和API Gateway，虽然AWS说很重要，但如果第一次看这两个服务的介绍，一定会认为是两个鸡肋的服务，但事实是否就是这样？如果说他们是AWS将其微服务架构的核心，我们是否还会这样认为呢，我们先简单看看这两个是何方神圣，想具体了解其应用实例，请看文章末尾的参考信息。

**AWS Lambda**官方介绍是一项计算服务，依响应资源事件来运行自定义的代码并自动管理底层计算资源。说白了是一个对AWS现有资源的事件监听，让后触发用户脚本以执行具体动作，脚本目前支持java和node.js。

**AWS API Gateway**是一个API发布框架，用以整合现有系统的后端服务，并且和AWS Lambda结合构筑基础Web服务。

正是这两个服务的推出，打破了我们对传统web App的架构定义，传统的B/S架构在AWS上变成了这样：

![ruby-gems]({{ "/css/pics/201601/aws_microservice.png"}})   
** 图片来源：[https://www.npmjs.com/package/deep-framework](https://www.npmjs.com/package/deep-framework "https://www.npmjs.com/package/deep-framework")

AWS称其为server-less architecture，也是其微服务架构的另一种叫法。从上图我们看出，通过Lambda和API Gateway将其原有CloudFront和后端的DynamoDB、RDS连接到一起，将原有的B/S架构中的Web层、App层和DB层拆分为若干个自服务的组合，且这其中没有Web Server计算资源，所以称其为server-less。而且更为重要的是AWS这几层服务间是完全解耦的，你可以独立申请和升级你的服务扩展（脚本、配置）。

### 总结

一个典型的Cloud-Native的微服务架构模型应该像下图这样，存在3个维度的伸缩能力，而微服务是解决Y轴方向的伸缩能力，系统级能力的伸缩:

![ruby-gems]({{ "/css/pics/201601/micorservice_scaling_archeticture.png"}})

所以个人认为，评价一个应用或软件系统是否是微服务架构的一个重要评判标准是否具备系统级的伸缩能力。



### 参考

Lambda和API Gateway：   
[http://chuansong.me/n/1849274](http://chuansong.me/n/1849274 "http://chuansong.me/n/1849274")   
对于微服务模式有个翻译版本：   
[http://my.oschina.net/douxingxiang/blog/358522](http://my.oschina.net/douxingxiang/blog/358522 "http://my.oschina.net/douxingxiang/blog/358522")   
[http://my.oschina.net/douxingxiang/blog/357425](http://my.oschina.net/douxingxiang/blog/357425 "http://my.oschina.net/douxingxiang/blog/357425")   
[http://my.oschina.net/douxingxiang/blog/358173](http://my.oschina.net/douxingxiang/blog/358173 "http://my.oschina.net/douxingxiang/blog/358173")


** 原创，引用请注明出处，谢谢。