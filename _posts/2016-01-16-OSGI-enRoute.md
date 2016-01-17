---
layout: post
title:  "OSGi enRoute — 一个新的OSGi应用框架（翻译）"
date:   2016-01-16 22:50:10
categories: OSGi
excerpt: 
---

* content
{:toc}


### 序

2011年由于工作的需要，让我开始接触了OSGi，一开始被其模块化的能力所吸引，但真正使用起来也被其标新立异的开发方法所折磨一番，后面产品架构转向SOA，慢慢的冷落这个相伴多年的“老朋友”，如今微服务火爆了起来，让我无意间又关注到了它。不知不觉中OSGI规范从R4发展到了如今的R6版本，OSGi联盟关注的重心开始转向IOT、企业级领域相关规范的制定与应用，甚至是微服务领域。

OSGi之父Peter Kriens正致力于OSGi enRoute，一个一站式的P层服务（个人理解），并提出了μservice模型的概念。 Peter Kriens认为OSGi μservice服务模型是一个成功的模式转变，但大部分旁观者仍然无视它的优势。今天我们看看Peter Kriens是如何看待OSGI发展现状和问题对未来有何展望，其中我们能看到在模块化领域Peter Kriens有着怎样的努力、无耐与希望，由于笔者水平有限，翻译不一定准确，有条件的TX请阅读原文，连接位于文章末尾，谢谢。

![ruby-gems]({{ "/css/pics/201601/osig_iot.png"}})

---

当我们开始使用OSGi，我们将建立一个世界级的框架，面向于小型网络。这与今天的树莓派是一个巨大的对比。现在，在16年后，OSGi成为了Java应用程序市场中的主要的模块化解决方案。我们相信一个模块化的解决方案是每一个复杂的Java应用程序必然选择。这也使我们认为当前的一些博客中关于OSGi观点是不正确的。

OSGi对于行业的影响，如同面向对象程序设计和控制反转/依赖注入这些设计原则一样，同时它还提供了一个新的开发模式的转变：面向服务的程序设计（service oriented programming），并成为该领域唯一的解决方案。这里并不是要借此炒作概念，面向服务的程序设计并不是在OSGi之后才提出的。此设计原则的提出要追溯到1998年创立的Service Oriented Systems (SOS)，而SOS一直被认为是OSGi设计的基石（过去我们的服务被叫做“微服务”，之后SOA盗用了这一术语，现在我们使用“μservice”这一名字，因为微服务正更多的应用于REST服务较重的模式中。）

虽然μservice模式是OSGi的最重要的特征，但它也是其最大的挑战，因为它需要开发者改变原来的开发思维方式。 Java领域中，特别是Java EE中，存在着一些流行但却糟糕的习惯。例如，类的载入机制并没有用作为API的扩展机制，另一个例子是使用静态成员变量作为全局变量的做法，它们都具有类似的缺点：它们都引出令人讨厌上下文和单例问题。更不幸的是，Java不提供更强大的扩展模型（*），来使用模块（可重复使用的）构建应用，而OSGi提供了实际的模块化解决方案。但是，OSGI正试图改掉这些糟糕的习惯，确反而使得OSGI很难被主流的JAVA程序员接受。也就是说，只有当这些开发者意识其深陷泥藻时才能理解到OSGi是其必然的解决之法。

*Jigsaw已被Oracle至少推迟到Java 9才发布(译者)

### OSGi μservice 模型是一个模式转变

模式转变的最大的问题是，观望者往往只是对新模式的优势的盲从。这好比一本描述三维世界美景的书籍。要真正看懂它，我们无法通过一维的点来概括二维的线，也无法通过二维的面来概括三维的体（以点概面、以偏概全）。这也如同当我们试图去思考4维或5维世界的时候为什么会迷失一样。我们可以理解这个道理，我们可以有成千上万的维度，但我们绝不会“感觉”它真实所在一样。一个例子，在Sun的Java问题跟踪中有人提出需要提供Multi Map实现（key with multiple values，即一个键对应多个值的HashMap,在实际开发中这类Map是有存在场景的，笔者开发产品中就多次用过，只能用Apache Common提供的扩展包实现，by译者），此种实现被视作Java集合API中是个另类。这个问题后来被Sun关闭了，因为其作者不认为有实现的必要（而这不是通过投票方式决定的）。类似一个老话题，关于在Java中添加匿名函数（lambdas）的讨论也曾持续进行过几年的时间。当时很多开发者都持怀疑态度，但事实证明在短短几年后，这些开发者就无法想象缺少匿名函数的世界会变成什么样。这个世界应该是一个净土——如果我们只谈到了技术——但实际上我们却带着愤怒。

因此，在OSGi联盟，对于软件设计我们提出了一个美好和新的维度，但发现对其推广却很艰难。也就是说，只有当人们体验到它的好处，他们才会不耐烦地问我们为什么没有人早告诉他们！然后他们通常会写一篇不易理解的OSGI“Hello World”的放入他们的博客。这正像在上个世纪向人们推广面向对象技术一样。是的，曾经有一段时间，整个行业对面向对象也是十分的怀疑。


###  OSGI有什么特别之处


让我们来个简短的介绍以补充一些技术背景。

在OSGi中，当你创建一个组件，您可以创建一个类，将作为μservice，并将其依赖作为μservices。例如，下面的代码是一个提供了Speak功能的μservice组件

    public interface Speak { void say(String text); }

    @Component
    public class SpeakLogImpl implements Speak {

        Logger logger;

        public void say(String text) {

          logger.info(text);
        }

	@Reference
        void setLog( Logger log) { this.logger = log; }
   	}
   

这里存在一个最常见的质疑：“为什么没有使用依赖注入？”嗯，这是一个新维度的问题，有很多不可见的因素。首先，虽然这里不可见的，speak组件实际是完全动态创建的，创建期间还要确保很多并发动作。例如，Speak仅在日志服务被注册后再去注册。

谁会使用Speak μservice，日志服务又从何而来？这在我们组件创建阶段不需要关注。该组件已表示出其所能提供的能力（speak μservice）和提供能力所需要满足的需求（日志μservice）。部署/编译人员的任务是创建匹配器要求和能力的环境。因为该组件自身是根本不关注这些细节的，应该只关注其领域功能（这可以说是很普通的例子）。

在这个例子中，我们可以先忽略动态性，因为系统发现有相关依赖时会激活一个SpacekLogImpl实例，并在依赖结束时会停用它。实例是不能真正感知到它的生命周期的，但它确实可以在一个动态的世界做出正确的行为。这如同吃了你的蛋糕一样！

事实上，它是动态的，在真实的三维的世界，这可能让人吃惊，但却能让享受其中。在现实世界中事物在改变，我们往往会处理那些因特定领域动态变化的代码。通过使用原始的μservice，我们有一个极好的通用基础来管理数量惊人的生命周期，初始化，排序，故障，情景模式变化或什么都不做（事件或动作），特定的领域代码。其结果是，基于μservice系统往往是非常坚固和可靠的。

μservice应用中最具代表性的例子是在分布式系统。因为组件注册了服务而不必关注谁可以使用该服务，使得任何其它组件可以发现并注册服务。根据这样的服务，因此可以注册为一个communication endpoint，并通过网络服务发现层的服务特征（和鉴权）提供它。另外OSGi系统能够获取该服务并注册一个本地代理的μservice。由于服务是动态的，我们仍可以正确处理如下情况：当连接断开或者原始服务提供者决定注销其服务时，或者因为服务被停止或服务相关的依赖已不再满足时。任何人一旦使用了这种模式编程就会喜欢上它，因其在处理大量的故障情况时的优雅与简洁。

例子中第二个不可见的部分中是配置。通过在OSGi配置可以动态并实时地管理Speak μservice，定义其实例个数包括其个数是一个或多个，所有实例的不同属性。

也许对你来说这没什么（就如同在八十年代解释面向对象编程一样），但我可以保证当面对复杂性的系统时，在一个相当大的系统中OSGi能够全面的降低其复杂性。其中关键原因在于使用了μservice模型的API对复杂性的降低带来了效果。因为这些API不必关注他们自身的生命周期、动态性、初始化、配置和其它与其领域无关的细节，而这些往往都是数量繁多、琐碎，而且更为简单的问题。

在OSGi中，使用μservice的API的唯一关注的是因为配置和生命周期问题而需要的协作。如果组件是执行者，那么服务是他们所要扮演的角色。一个μserviceAPI描述了这些角色之间的协作，就像出演一场戏一样。因此，一个μserviceAPI需要的不是仅仅是一个接口，一般我们都要定义多个角色，又叫做interfaces。例如，在事件管理μservice描述中，我们有事件处理的角色和事件管理的角色。因此，一个μserviceAPI是在Java包中指定。那么这个包还可以容纳辅助类，例如像在事件管理μservice中的事件对象。一个好的μservice API会高内聚地定义在一个包中，而不是覆盖到其它的API。

所有组件都需要被部署在一个运行时环境，这里通过bundle实现。bundle是一个包含了类，资源和元数据的JAR文件。Bundle在运行时环境中实例化。也就是说，组件可查看其bundle，并根据bundle的信息，扩展这些bundle。例如，在OSGi enRoute 有一个Web服务器的扩展，将其bundle下的‘/static’目录的内容映射到，当前的Web服务器上，同时提供高速缓存的管理和更高级的HTTP的特性，如（缓存）压缩和范围。这种模式被称为extender模型，是OSGi中最流行的模式之一。

###模块化

Bundle化即模块化。之所以做到模块化，因为在bundle间提供了隔离：创建私有和公共空间。这降低了总体的复杂性，因为只有较少的移动部件。模块化是被普遍认可和实践的设计。OSGi模块化与传统的JAVA的不同之处在于其依赖模型。我们从面相对象技术中学到了直接依赖其它的类并不是一个好主意。用这种方式创建的系统如同将口香糖粘到了头发上：很棘手并难于处理。 Java的创新是，它增加了接口打破传递类型的依赖，挽救了九十年大量的面向对象的失败或近乎失败的项目。接口打破传递式的依赖关系，因为它将接口实现者于接口的使用者的依赖关系解除。现在可以在其它上下文重用这些库，只需引入这些接口而不必引入其它的使用者和接口实现者。

如果对这个问题感觉很熟悉，那么你可能会想到的Maven。Maven使用了相同的依赖模型，但是在处理类依赖关系中并不成功。虽然Maven不显示地从网上下载依赖，而是人为手动下载，但是直接使用模块间的依赖势必造成依赖传递，这使得模型还是迫切地需要从网络下载。而这不是我们真正需要的，那么什么才是真正需要的？正像JAVA接口给面向对象所带来的东西。

在OSGi中，我们将package作为用于模块的接口，因为它已经是μserviceAPI的单位。因此，一个bundle可以导出一个package，一个bundle也可以导入一个package。如果一个bundle导入了一个package，那么另一个bundle就必须导出的这个package。在这个模型中，可能有许多bundle提供了相同的package，那么问题来了：如何找到我所需要的package导出？

使用Maven，如果你有一个仓库的URL，一个组ID，一个artifactID和版本号，那么你可以下载这个artifact。但是你也锁定了它，就像你依赖了一个具体实现类一样。在OSGi中，你不存在这样的锁定，bundle可以在不同上下文中被广泛的重用，但代价是它需要额外的装配步骤，然后才能运行。

渐渐地我们了解到，packages实际上是一个通用能力模型的示例。我们现在用其定义了依赖的能力，同时也允许用户自定义功能。通过这些功能的版本控制，我们可以确信，也可以检测到不兼容性。这一切可能引起了你的担忧：维护这些能力需要巨大的工作量：我们可以向你保证，一些工具可以提取大部分的类文件，甚至可以产生必要的元数据注解。这个优势在于，bundle现在是自描述的，我们可以利用工具来组装bundle以生成应用。

### Fast Forward

随着时间的推移OSGi联盟创建了基于组件的系统的坚实基础，并通过μservice作为组件之间的联系。Bundle是部署单元并具有自描述能力，因此可以通过工具组装成应用程序。这个基础是可靠的，因为OSGi规范包含一些惊人的捷径方法，便于有据可查，确保其可操作性。很难想象一个真正的软件工程组件模型有如此好的规范作为基础。

这就是说，我们了解到对于开发者我们没有给予适当的重视。我们专注于规范的执行者。我们主要讲述的应用开发者，他们现在有一个梦幻般的乐高积木盒子，它们必须弄清楚自己下面该如何做。如果他们做不到，那是他们自己的问题。

我认为作为失败的另一个方面是一些我们在OSGi企业规范中规定的服务的API设计。大多数这些规范被衍生或与相应的Java EE的API相同。虽然我明白的其中的原因，Java EE的API有大批追随者和许多实现，我认为这是一个错误。重用另一个领域开发的API，使得它很难利用OSGi提供的独特功能。其结果是太多”跟随“规范，与原始目标背离，产生偏差，按照原来的定位受益却微小。这就是说，说不尽的公司都在使用它，令人惊讶的是总还能温馨地听到很多地方的人正在采用这种技术。

但是，我们相信，有正确的OSGi的API是值得的。实际上，我们认为随着使用主流API的生产环境的增多，我们可以说服许多主流开发商进来在我们的方向倡导之下，帮助他们度过难关。

显然，这不是一件容易事，与主流的竞争往往是一场失败的战争（当然或者能坚守到成功）。一个大问题是如何实现，因为大多数开源项目都集中在Java EE或SE。幸运的是，经验表明，由于在OSGi的API往往要小得多，通常用现有的（开源）的实现并不困难，因为其中隐藏着大量惊人的实现细则。只要把所有的类（包括通常依赖的）放到一个bundle中，并不做任何导出，只是小的API集合。这些东西，看起来有一个陡峭的学习曲线，但在实际操作中并非如此，而这其间能真正体会到OSGi带来的好处并实现模块化。

### 为什么使用OSGi enRoute?

我们希望实现真正的OSGi服务并且向世界展示OSGi能力是OSGi联盟启动了OSGi enRoute项目的原因。愿景是创建一个开放源码环境用以生成完全的100％的OSGi方式的应用程序。发掘对OSGi有益和OSGi发展或缺的东西，并与现有的开源基金会尽可能合作。 OSGi的enRoute项目试图提供一站式服务，包括最佳实践，全面的工具链，配置文件，文档，教程和运行时分发。通过OSGi的enRoute，你可以在数分钟内启动和运行Web应用程序项目基于REST的backend（OK，不包括下载时间:)）。

![ruby-gems]({{ "/css/pics/201601/osgi_cd.png"}})
** 基于Karaf的一站式管理，图片来自：[http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf](http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf "http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf")

### OSGi enRoute是什么？

完整的工具链。它提供了（基于Eclipse Bndtools）一个IDE，基于Travis的持续集成（Gradle与BND），与GitHub的整合，一个Maven索引仓库，被称为JPM4J。在IDE中创建的项目可以推到GitHub上，并会自动在Travis建构建而不需要额外的工作。

Bndtools是首屈一指的OSGi的开发环境。 OSGi的enRoute在GitHub上提供和模板base workspace模板为API创建不同的项目类型，一个API，测试bundle和Web应用程序。 Web应用程序创建一个小的应用程序展示一个Bootstrap/Angular的用户接口和REST后端。 Bndtools提供了简短示例：一个新网页，显示的修改后，通过Javascript调用到后端REST服务器的整个开发周期。流程中没有可妥协的错误。由于工具链是基于BND，错误和不正确的做法在发布之前会被标记出来。它甚至集成了基线，持续比对上一版本和当前API的差异，如果需要还可以标记语义级的版本违规。而当你准备好发布，可以选择发布了一些仓库或待持续集成通过后才发布。 （作为一个在BND的代码提交者，可能存在我个人的偏好，但我从来没看到一个环境如此容易的开发OSGi和Java代码，也可以是Javascript或其他Web语言。）

一个OSGi enRoute配置文件是一组服务API，扩展和web资源文件。它们被聚合在JAR文件仅是一个规则。这个JAR文件可以放在你的构建路径，并允许您开发无厂商锁定的应用程序。外部的依赖可以通过JPM库中添加。

目前OSGi的enRoute Base Profile包括以下服务：

org.osgi.service.cm – Provides a push and pull model to configure components.  
org.osgi.service.component – An extender for Declarative Services components.  
org.osgi.service.coordinator – A coordinator for thread based requests with the possibility to get acallback at the end of a request.  
org.osgi.service.event – A simple publish and subscribe mechanism for generic events.  
org.osgi.service.http – An HTTP server framework.  
org.osgi.service.log – A service to log errors, warning, and information to.  
org.osgi.service.metatype – A standard to describe data structures with the intent to create userinterfaces from.  
org.osgi.service.url – A facility to register URL handlers via the service registry.  
org.osgi.service.useradmin – A service to manage user information like groups, credentials, andgeneral properties.  
org.osgi.util.promise – A utility to do asynchronous processing using Promises.  
org.osgi.util.tracker – Utilities to reliably track services and bundles.  
osgi.enroute.authentication.api – Providing an authenticated id based on variable usercredentials.  
osgi.enroute.authorization.api – Authorising applications to execute their actions by providingcurrent user based permissions.   
osgi.enroute.capabilities – All specific non-code capabilities found in an enRoute Base profile.  
osgi.enroute.configurer.api – An extender that gets configuration information from a bundle.  
osgi.enroute.debug.api – Constants and helpers for debugging.  
osgi.enroute.dto.api – Support for converting objects to other objects or JSON.  
osgi.enroute.jsonrpc.api – A white-board approach to JSON RPC.  
osgi.enroute.launch.api – API to interact with the launcher, including getting access to the startuparguments.  
osgi.enroute.logger.api – A facility to manage logging on a platform when not only OSGi loggingis used.  
osgi.enroute.rest.api – Provides REST end-points that are based on method naming pattern withtype safe use of the pay-load, parameters, and result body.  
osgi.enroute.scheduler.api – Provides time based functions like delays, (cron and period based)schedules, and timeouts on Promises.  
The following web resources:  
  
Angular Javascript  
Angular UI Javascript  
D3 Javascript  
Bootstrap CSS  
Pagedown Javascript Markdown editor  
The following support bundles:  
  
A web server that serves static resources from bundles.  
A configurator that reads configuration from bundles  
An OSGi Event Admin μservice to Server Sent Events (Javascript) bridge  
The Base Profile and its services are OSGi Core Release 6 and Java 8 based. The new APIs leverage lambdas to the hilt.  

### Workflow

profile使用于复杂的Web应用程序。最重要的发展是在enRoute网站上提供的服务目录。该服务目录为当前如何使用一个服务提供了指南。意图在于每个服务将有一个附带的应用程序，允许开发人员复制并粘贴使用模式，并在调试器服务使用。

Service Oriented Systems由可被重用的bundle组成。即使是一次性的bundle，也应该考虑其后的发展，它有可能被重用，因为这些考虑往往在开发阶段，场景可能过于简单了。定义适当的API，并确保只有API packages被导出。当组件开发完成或从存储库选择之后它们需要被打包成一个应用程序。在传统的环境中，创建这个应用程序流程往往十分痛苦，因为它必须包含所有的传递依赖，并没有其它的帮助，这可能是非常痛苦的。出于这个原因，Bndtools具有决心可以在快速创建bundle的封闭，并提供所有必要的功能。

一旦应用程序被定义好，Bndtools会提供一个调试和测试环境，这是非常敏捷的。您可以创建一个bndrun文件，指定喜欢的框架，属性等，这bndrun文件可以加载和调试。实际上IDE中的所有更改会立即生效;你几乎不需要重新启动框架。

输出的包依赖于目标运行时环境。目前，默认情况下，我们打包为一个可执行的JAR，它包含的框架，属性和其它的包。该JAR可以运行任何标准Java8运行时环境中。但是，我们正在与IBM合作，使其同时部署（和调试）在IBM WebSphere Application Server Liberty Core，与Paremus合作部署服务到Paremus Service Fabric中。我们也寻找到Karaf Kars和其它的运行时环境。我们欢迎任何其它目标。这些目标必须提供一个分布式;此分布式必须为每一个profile定义能力。

![ruby-gems]({{ "/css/pics/201601/Karaf_framework.png"}})
** 微服务实现，图片来自：[http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf](http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf "http://events.linuxfoundation.org/sites/events/files/slides/Microservices-OSGi-Running-with-Apache-Karaf.pdf")

文档和教程（目前所有的工作都是聚焦点），可在enroute.osgi.org。我们所做的一切都是在GitHub上。如果你想弄明白所有的部分，Enroute Handbook会是个良好的开端。

## Where are we?

那么，OSGi的enRoute目前的进展如何？我们基本上是功能齐全，我们已经准备好进行集成测试。这意味着，我们需要热情或将要更早的准备帮助实现OSGi的enRoute走的更远，经历艰难险阻，欢呼雀跃。 1.0版的发布日期是在这个夏天，与计划OSGi R6的发布日期一致，以便它可以包括一些必要的更新。

期间主要的出色工作是文档编写，创建示例，制作关于OSGi enRoute教程和传教。

在OSGi的enRoute工作已经明确，我们确实提供了如乐高般的一个美好的方块世界，虽然我们忘记了汇编指令。很显然，OSGi的模块化真正的作品是它提供了一个复杂的依赖模型，能够真正构建应用了可重用的组件。然而，显然我们需要真正的OSGi服务为每一个开发人员解决他们面临的共同任务。这条路上我们没有太多选择。目前，OSGi的enRoute工作促成了十个OSGi的要求建议书（163-172），这只是规范化进程中的第一步。

因此，如果你敢于第一个吃螃蟹，或者如果你有时间对社区贡献，请不要犹豫与我联系。其它需要关注的只是，因为一旦我们完全生存下来，OSGi的将是令人兴奋的！

### 参考

原文：[https://jaxenter.com/osgi-enroute-a-new-framework-for-osgi-applications-117514.html](https://jaxenter.com/osgi-enroute-a-new-framework-for-osgi-applications-117514.html "https://jaxenter.com/osgi-enroute-a-new-framework-for-osgi-applications-117514.html")

** 原创，引用请注明出处，谢谢。