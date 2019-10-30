# Migration to Microservices Overview

Migrating a monolithic application to a microservices architecture means a series of modernization processes, a bit like what these generations of developers have been doing. In real time, when migrating, we can reuse some ideas.

One strategy is: don't rewrite the code on a big bang (you can override this method only if you are rebuilding a new microservice-based application). Rewriting the code sounds great, but it's actually full of rish and may end up failing, as Martin Fowler said.

> The only thing a Big Bang rewrite guarantees is a Big Bang!

Instad, a strategy of gradual migration of monolithic applications should be adopted. By gradually generating new microservices applications and integrating with the old monolithic applications, over time, the proportion of monolithic applications in the entrie architecture gradually declines until disappears or becomes Part of the microservices architecture. This strategy is a bit like maintaining a speed limit of 70 on the highway. Although it is challenging, it is much less risky then rewriting.

Martin Fowler used this modernizatino strategy as a Strangler application, named after the strangler vine in the rainforest, also known as the strangler fig. In order to climb to the top of the forest, the vines are entangled in the growth of the uncle. After a while, the tree dies and leaves the tree-shaped vine. This application also uses the same model to develop new microservices applications around traditional applications, and traditional applications will gradually step out of the stage.

Let's take a look at other possible strategies.

# Strategy 1 - Stop mining

Law of Holes means that you should stop digging when you enter the hole. This is the best advice when a monolithic applciation is not manageable. In other words, you should stop letting monolithic applications continue to grow larger, which means that when developing new features, you should not add new code to the old single application. The best approach would be to develop the new functionality into a standslone microservice. As shown below:

![1](/images/Law-of-Holes.png)

In addition to new services and legacy applications, there are two modules, one of which is the request router, which handles the ingress (http) request, a bit like the API gateway mentioned earlier. The router sends new feature requests to the newly developed service, and the legacy request is sent to the single application.

The other is the glut code, which integrates microservices with single applications. Microservices rarely exist independently and often access data from a single application. The glue code, which may be in a single application or as a service or both, is responsible for data integration. Microservices read and write data from a single application using glue code.

Microservices have three ways to access single application data:

- Remote API provided by the Ventilation Single Application
- Direct access to the single application database
- Maintain a data that is synchronized from a single applciation

The glue code is also known as the anti-corruption layer because the glue code protects the micro-service new domain model from the traditional single-application domain model. The gule code provides translation between the two nodels. The term anti-corruption layer first appeared in Eric Evans's must-read book *Domain Driven Design*, which was later refined into a white paper. Developing a disaster recovery layer may not be very important, but it is a necessary part of avoiding a monolithic quagmire.

Implementing new features in a lightweight microservice approach has many advantages, such as prevening individual applications from becoming more unmanageable. Microservices themselves can be developed, deployed, and scaled independently. Adopting a micro-service architecture will give developers different personal experiences.

然而，这方法并不解决任何单体式本身问题，为了解决单体式本身问题必须深入单体应用​做出改变。我们来看看这么做的策略。

# 策略 2——将前端和后端分离

减小单体式应用复杂度的策略是讲表现层和业务逻辑、数据访问层分开。典型的企业应用至少有三个不同元素构成：

1. 表现层——处理 HTTP 请求，要么响应一个 RESTAPI 请求，要么是提供一个基于 HTML 的图形接口。对于一个复杂用户接口应用，表现层经常是代码重要的部分。

2. 业务逻辑层——完成业务逻辑的应用核心。

3. 数据访问层——访问基础元素，例如数据库和消息代理。

在表现层与业务数据访问层之间有清晰的隔离。业务层有由若干方面组成的粗粒度（coarse-grained）的 API，内部包含了业务逻辑元素。API 是可以将单体业务分割成两个更小应用的天然边界，其中一个应用是表现层，另外一个是业务和数据访问逻辑。分割后，表现逻辑应用远程调用业务逻辑应用，下图表示迁移前后架构不同：​
![2](/images/Before-and-after-migration.png)

单体应用这么分割有两个好处，其一使得应用两部分开发、部署和扩展各自独立，特别地，允许表现层开发者在用户界面上快速选择，进行 A/B 测试；其二，使得一些远程 API 可以被微服务调用。

然而，这种策略只是部分的解决方案。很可能应用的两部分之一或者全部都是不可管理的，因此需要使用第三种策略来消除剩余的单体架构。

# 策略 3——抽出服务

第三种迁移策略就是从单体应用中抽取出某些模块成为独立微服务。每当抽取一个模块变成微服务，单体应用就变简单一些；一旦转换足够多的模块，单体应用本身已经不成为问题了，要么消失了，要么简单到成为一个服务。

## 排序那个模块应该被转成微服务

一个巨大的复杂单体应用由成十上百个模块构成，每个都是被抽取对象。决定第一个被抽取模块一般都是挑战，一般最好是从最容易抽取的模块开始，这会让开发者积累足够经验，这些经验可以为后续模块化工作带来巨大好处。

转换模块成为微服务一般很耗费时间，一般可以根据获益程度来排序，一般从经常变化模块开始会获益最大。一旦转换一个模块为微服务，就可以将其开发部署成独立模块，从而加速开发进程。

将资源消耗大户先抽取出来也是排序标准之一。例如，将内存数据库抽取出来成为一个微服务会非常有用，可以将其部署在大内存主机上。同样的，将对计算资源很敏感的算法应用抽取出来也是非常有益的，这种服务可以被部署在有很多 CPU 的主机上。通过将资源消耗模块转换成微服务，可以使得应用易于扩展。

查找现有粗粒度边界来决定哪个模块应该被抽取，也是很有益的，这使得移植工作更容易和简单。例如，只与其他应用异步同步消息的模块就是一个明显边界，可以很简单容易地将其转换为微服务。

## 如何抽取模块

抽取模块第一步就是定义好模块和单体应用之间粗粒度接口，由于单体应用需要微服务的数据，反之亦然，因此更像是一个双向 API。因为必须在负责依赖关系和细粒度接口模式之间做好平衡，因此开发这种 API 很有挑战性，尤其对使用域模型模式的业务逻辑层来说更具有挑战，因此经常需要改变代码来解决依赖性问题，如图所示：

一旦完成粗粒度接口，也就将此模块转换成独立微服务。为了实现，必须写代码使得单体应用和微服务之间通过使用进程间通信（IPC）机制的 API 来交换信息。如图所示迁移前后对比：

![3](/images/30103116_ZCcM.png)

此例中，正在使用 Y 模块的 Z 模块是备选抽取模块，其元素正在被 X 模块使用，迁移第一步就是定义一套粗粒度 APIs，第一个接口应该是被 X 模块使用的内部接口，用于激活 Z 模块；第二个接口是被 Z 模块使用的外部接口，用于激活 Y 模块。

迁移第二步就是将模块转换成独立服务。内部和外部接口都使用基于 IPC 机制的代码，一般都会将 Z 模块整合成一个微服务基础框架，来出来割接过程中的问题，例如服务发现。

抽取完模块，也就可以开发、部署和扩展另外一个服务，此服务独立于单体应用和其它服务。可以从头写代码实现服务；这种情况下，将服务和单体应用整合的 API 代码成为容灾层，在两种域模型之间进行翻译工作。每抽取一个服务，就朝着微服务方向前进一步。随着时间推移，单体应用将会越来越简单，用户就可以增加更多独立的微服务。
将现有应用迁移成微服务架构的现代化应用，不应该通过从头重写代码方式实现，相反，应该通过逐步迁移的方式。有三种策略可以考虑：将新功能以微服务方式实现；将表现层与业务数据访问层分离；将现存模块抽取变成微服务。随着时间推移，微服务数量会增加，开发团队的弹性和效率将会大大增加。
