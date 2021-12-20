## Axon

### 什么是 Axon

Axon Framework 通过支持开发人员应用命令查询责任隔离（CQRS）架构模式来帮助构建可扩展和可维护的应用程序。它通过提供最重要的构建块（例如聚合，存储库和事件总线（事件的分发机制））的实现来实现。此外 Axon 提供注释支持，它使您可以构建聚合和事件侦听器，而无需将代码与 Axon 特定的逻辑绑定在一起。这使您可以专注于业务逻辑而不是管道，并可以使您的代码更易于独立测试。

并非每个应用程序都会从 Axon 中受益。不期望扩展的简单 CRUD 应用程序可能不会从 CQRS 或 Axon 中受益。然而，有各种各样的应用程序确实受益于 Axon。
可能受益于 CQRS 和 Axon 的应用:

1. (系统功能需要频繁迭代新功能)应用程序很可能在很长一段时间内使用新功能进行扩展。例如，在线商店可能从订单模块进度的系统开始。在稍后阶段，这可以通过库存信息进行扩展，以确保库存在出售时得到更新。甚至在以后，会计可以要求记录销售的财务统计，等等。虽然很难预测软件项目在未来将如何发展，但大多数这类应用程序都是这样的。
2. (频繁读写的应用)应用程序具有高的读写比。这意味着数据只写几次，并多次读。由于查询的数据源与用于命令验证的数据源不同，因此可以优化这些数据源以实现快速查询。重复数据不再是问题，因为数据更改时会发布事件。
3. 应用程序以多种格式显示数据。现在，许多应用程序在网页上显示信息时不会停止。例如，一些应用程序每月发送电子邮件，通知用户可能发生的与其相关的更改。搜索引擎是另一个例子。它们使用的数据与应用程序使用的数据相同，但以一种为快速搜索而优化的方式。报表工具将信息聚合到显示数据随时间演变的报表中。同样，这也是同一数据的不同格式。使用 Axon，可以在实时或计划的基础上独立地更新每个数据源。
4. 当应用程序明显地将组件与不同的对象分开时，它也可以从 Axon 中受益。这种应用程序的一个例子是在线商店。员工将在网站上更新产品信息和可用性，而客户则会发出订单并查询其订单状态。使用 Axon，这些组件可以部署在不同的机器上，并使用不同的策略进行缩放。它们使用事件保持最新状态，无论部署在哪台机器上，Axon 都会将这些事件分派给所有订阅的组件。
5. 与其他应用程序集成可能是一项繁琐的工作。使用命令和事件严格定义应用程序的 API 可以使其更容易与外部应用程序集成。任何应用程序都可以发送命令或侦听应用程序生成的事件。

### 架构概述

CQRS 本身是非常简单的模式。它只规定处理命令的应用程序的组件应该与处理查询的组件分离。虽然这种分离本身非常简单，但是当与其它模式结合时，它提供了许多非常强大的特征。

axon 提供构建块，使得能够更容易地实现可以与 CQRS 组合使用的不同模式。下图显示了基于CQRS的事件驱动架构的扩展布局的示例。
![1570761795693](D:\Typora\image\1570761795693.png)
左侧显示的 UI 组件以两种方式与应用程序的其余部分交互：它向应用程序发送命令 Command(在顶部显示)，并查询应用程序以获取信息(在底部显示)。

命令通常由简单和直接的对象表示，对象包含命令处理程序执行该命令所需的所有数据。命令以其名称表示其意图。在 Java 术语中，这意味着使用类名称来找出需要完成的操作，并且命令的字段提供执行该操作所需的信息。命令总线接收命令并将它们路由到命令处理程序。每个命令处理程序响应特定类型的命令并基于命令的内容执行逻辑。但是，在某些情况下，您还希望执行逻辑，而不管实际的命令类型，例如验证、日志记录或授权。命令处理程序从存储库中检索域对象(聚合)，并对其执行方法以更改其状态。这些聚合通常包含实际的业务逻辑，因此负责保护它们自己的不变量。

聚合的状态变化导致生成域事件。域事件和聚合都形成域模型。存储库负责提供对聚合的访问。通常，这些存储库被优化用于仅通过其唯一标识符来查找聚合。一些存储库将存储聚合本身的状态(例如，使用对象关系映射)，而其他存储库存储聚合已在事件存储中经历的状态更改。存储库还负责持久存储在其备份存储中对聚合进行的更改。

Axon 为持久聚合的直接方式（例如使用对象关系映射）和事件来源提供支持。事件总线将事件分派给所有感兴趣的事件侦听器。这可以同步地或异步地完成。异步事件调度允许命令执行返回并将控制移交给用户，而在后台调度和处理事件。不必等待事件处理才能完成应用程序的响应。另一方面，同步事件处理更简单，是明智的默认处理。默认情况下，同步处理将处理同样处理该命令的同一事务中的事件侦听器。事件侦听器接收事件并处理它们。一些处理程序将更新用于查询的数据源，而另一些处理程序将消息发送到外部系统。正如您可能注意到的，命令处理程序完全不知道它们所做的更改感兴趣的组件。这意味着将应用扩展到新的功能是非常非侵入性的。您只需添加另一个事件侦听器。这些事件将应用程序中的所有组件松散地耦合在一起。在某些情况下，事件处理需要向应用程序发送新命令。

这是当接收到订单时的示例。这可能意味着客户的账户应借记购买的金额，并且必须告知装运准备装运所购买的货物。在许多应用中，逻辑会变得更加复杂：如果客户没有及时付款怎么办？您将立即发货，还是先等待付款？Saga 是负责管理这些复杂业务交易的 CQRS 概念。用户接口和数据源之间的薄数据层提供了与所使用的实际查询实现的明确定义的接口。此数据层通常将只读DTO 返回到包含查询结果的对象。这些 DTO 的内容通常由用户接口的需要来驱动。在大多数情况下，它们直接映射到 UI 中的特定视图(也称为表视图)。Axon 不能为该部分的应用提供任何构建块。主要原因是这非常简单，与分层体系结构不同。

#### 模块结构

AxonFramework 由多个模块组成，这些模块针对 CQRS 的特定问题区域。根据项目的确切需求，您需要包含一个或多个这些模块。

1. core: 模块包含 Axon 的核心组件。如果使用单节点设置，则此模块可能会提供所需的所有组件。所有其他 Axon 模块都依赖于这个模块，因此它必须始终在类路径上可用。
2. test: 测试模块包含测试工具，可以用来测试基于 Axon 的组件，例如命令处理程序、Aggregates 和 Sagas。通常在运行时不需要这个模块，只需要在测试期间添加到类路径中。
3. Distributed: 分布式 CommandBus 模块包含可用于在多个节点上分发命令的实现。它附带了用于连接这些节点的JGroups 和 SpringCloud 连接器。
4. AMQP: AMQP 模块提供的组件允许您使用基于 AMQP 的消息代理作为分发机制来构建 EventBus。这允许保证交付，即使事件处理程序节点暂时不可用。
5. Spring: Spring 模块允许在 SpringApplication 上下文中配置 Axon 组件.它还提供了许多特定于 Spring 框架的构建块实现，例如在 Spring 消息通道上发布和检索 Axon 事件的适配器。
6. MongoDB: 是一个基于文档的 NoSQL 数据库.Mongo 模块提供事件和 Saga Store 实现，这些实现将事件流和 sagas 存储在 MongoDB 数据库中。 几个 Axon Framework 组件提供监控信息。度量模块提供了基于 Codehale 的基本实现来收集监控信息。 

#### Spring 支持

Axon Framework 为 Spring 提供了广泛的支持，但不需要您使用 Spring 即可使用 Axon。 所有组件都可以通过编程方式配置，并且不需要在类路径上使用 Spring。 但是，如果确实使用 Spring，则通过使用 Spring 的注释支持可以简化许多配置。

### 消息传递

Axon 的核心概念之一是消息传递。组件之间的所有通信都是使用消息对象完成的。这为这些组件提供了必要的位置透明性，以便在必要时能够扩展和分发这些组件。尽管所有这些消息都实现了消息接口，但不同类型的消息与如何处理它们之间有着明显的区别。所有消息都包含有效负载、元数据和唯一标识符。消息的有效负载是对消息含义的功能描述。这个对象的类名和它所携带的数据的组合，描述了应用程序对消息的意义。元数据允许您描述发送消息的上下文。例如，您可以存储跟踪信息，以允许跟踪邮件的来源或原因。您还可以存储信息来描述执行命令的安全上下文。

#### Command

Command 描述更改应用程序状态的意图。它们被实现为(最好是只读) POJO，它们使用一个 CommandMessage 实现包起来。 命令总是有一个目的地。虽然发送方不关心哪个组件处理命令或该组件所在的位置，但了解它的结果可能会很有趣。这就是为什么通过命令总线发送的命令消息允许返回结果的原因。

#### Event

Event 是描述应用程序中发生的事情的对象。事件的一个典型来源是聚合。当聚合中发生重要事件时，它将引发一个事件。在Axon 框架中，事件可以是任何对象。我们非常鼓励您确保所有事件都是可序列化的。当事件被分派时，Axon 会将它们封装在EventMessage 中。实际使用的消息类型取决于事件的起源。当事件由聚合引发时，它被包装在 DomainEventMessage (扩展 EventMessage)中。所有其他事件都封装在 EventMessage 中。除了通用消息属性(如唯一标识符)外，EventMessage 还包含时间戳。DomainEventMessage 还包含引发事件的聚合的类型和标识符。它还包含聚合的事件流中事件的序列号，这允许复制事件的顺序。

#### Unit of Work

工作单元是 Axon 框架中的一个重要概念，尽管在大多数情况下，您不太可能直接与它交互。消息的处理被看作是一个单一的单元。工作单位的目的是协调在处理消息(命令或事件)期间所执行的行动。组件可以注册在工作单元的每个阶段执行的操作。

例如 onPrepareCommit 或 onCleanup。你不太可能需要直接访问工作单元。它主要由 Axon 提供的构建块使用。如果您确实需要访问它，无论出于什么原因，有几种方法可以获得它。

1. Handler 通过 Handle 方法中的参数接收工作单元。如果使用注释支持，则可以将 UnitOfWork 类型的参数添加到带注释的方法中。
2. 在其他位置，可以通过调用 CurrentUnitOfWork.get() 检索绑定到当前线程的工作单元。注意，如果没有绑定到当前线程的工作单元，此方法将引发异常。使用 CurrentUnitOfWork.isStarted() 查找是否可用。要求访问当前工作单元的原因之一是，附加在消息处理过程中需要多次重复使用的资源，或者在工作单元完成时是否需要清理创建的资源。在这种情况下，unitOfWork.getOrComputeResource() 和生命周期回调方法(如 onRollback()、postCommit() 和onCleanup() 允许您注册资源并声明在此工作单元处理过程中要采取的操作。

请注意，工作单元只是变更的缓冲，而不是事务的替代。虽然所有分阶段的更改都是在提交工作单元时才提交的，但它的提交并不是原子的。这意味着，当提交失败时，一些更改可能会持久化，而其他更改则不会。最佳实践规定，命令不应包含多个操作。如果你坚持这种做法，一个工作单位将包含一个单一的行动，使它安全地使用原样。如果工作单元中有更多的操作，那么可以考虑将事务附加到 Work 的提交。 使用 unitOfWork.onCommit(..) 注册在提交工作单元时需要执行的操作。

处理程序可能会由于处理消息而引发异常。默认情况下，未经检查的异常将导致 UnitOfWork 回滚所有更改。Axon提供了一些开箱即用的回滚策略：

1. RollbackConfigurationType.NEVER 将始终提交工作单元
2. RollbackConfigurationType.ANY_THROWABLE 发生异常时将始终回滚
3. RollbackConfigurationType.UNCHECKED_EXCEPTIONS 将回滚错误和运行时异常
4. RollbackConfigurationType.RUNTIME_EXCEPTION 将回滚运行时异常但是不包括 Error

当使用 Axon 组件处理消息时，将自动管理工作单元的生命周期。如果选择不使用这些组件，而是自己实现处理，则需要以编程方式启动和提交(或回滚)一个工作单元。在大多数情况下，DefaultUnitOfWork 将为您提供所需的功能。它期望在单个线程中进行处理。要在工作单元的上下文中执行任务，只需在新的 DefaultUnitOfWork 上调用UnitOfWork.ExecutWithResult(可调用)。工作单元将在任务完成时启动和提交，或者在任务失败时回滚。如果需要更多的控制，也可以选择手动启动、提交或回滚工作单元。示例如下:

```java
UnitOfWork uow = DefaultUnitOfWork.startAndGet(message);
// then, either use the autocommit approach:
uow.executeWithResult(() -> ... logic here);

// or manually commit or rollback:
try {
    // business logic comes here
    uow.commit();
} catch (Exception e) {
    uow.rollback(e);
    // maybe rethrow...
}
```

一个工作单元包含几个阶段。 每次进入另一个阶段时，都会通知 UnitOfWork 侦听器。

1. Active phase: 工作单位已启动。工作单位通常在此阶段(通过 CurrentUnitOfWork.set(UNITOFWork))注册。随后，该消息通常由这个阶段中的消息处理器处理。
2. Commit phase: 在完成消息处理之后，但在提交工作单元之前，将调用 onPrepareCommit 侦听器。如果工作单元绑定到事务，则调用 onCommit 侦听器来提交任何支持事务。当提交成功时，将调用 postCommit 侦听器。如果提交或之前的任何步骤失败，则调用 onRollback 侦听器。如果可用，消息处理程序结果包含在工作单元的 ExecutionResult中。
3. Cleanup phase: 这是该单位所持有的任何资源(如锁)将被释放的阶段。如果嵌套了多个工作单位，则清理阶段将被推迟，直到工作的外部单元准备好清理。

消息处理过程可以被认为是原子过程；它要么完全被处理，要么根本不被处理。Axon 框架使用工作单位跟踪消息处理程序执行的操作。在处理程序完成后，Axon 将尝试提交与工作单位注册的操作。可以将事务绑定到工作单位。许多组件（例如CommandBus 实现和所有异步处理事件处理器）允许您配置事务管理器。然后，该事务管理器将被用于创建与用于管理消息处理的工作单元绑定的事务。当应用程序组件在消息处理的不同阶段需要资源时，例如数据库连接或 EntityManager，这些资源可以附加到工作单位。unitOfWork.GetResources() 方法允许您访问连接到当前工作单元的资源。在工作单元上直接提供了几种辅助方法，以便更容易地与资源一起工作。当嵌套的工作单位需要能够访问资源时，建议将其注册到可以使用unitOfWork.root() 访问的工作的根单元上。如果工作单位是聚合根，它将简单地返回自己。

### 快速开始

下面就让我们快速开始吧，首先我们可以从 Maven、Gradle 中来获取依赖.

```xml
<!-- Maven 依赖 -->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-core</artifactId>
    <version>${axon.version}</version>
</dependency>
```

AxonFramework 是在 Java 8 上构建和测试的。由于 Axon 本身不创建任何连接或线程，所以在 ApplicationServer 上运行是安全的。Axon 通过使用执行器抽象所有异步行为，这意味着您可以轻松地传递一个容器托管线程池。如果您不使用完整的 ApplicationServer (例如 Tomcat、Jetty 或独立应用程序)，您可以使用 Executors 类或SpringFramework 来创建和配置线程池。

以上一个简单的 axon 框架环境就搭建好了.是不是很简单.

### Command Model

在基于 CQRS 的应用程序中，域模型可以是一种非常强大的机制，可以利用状态更改的验证和执行所涉及的复杂性。虽然典型的领域模型有大量的构建块，但其中一个模块在 CQRS 中的命令处理中起着主导作用：聚合。应用程序中的状态更改以命令开始。命令是表示意图(它描述了您想要做的事情)以及基于该意图进行操作所需的信息的组合。命令模型用于处理传入的命令，验证它并定义结果。在此模型中，命令处理程序负责处理特定类型的命令，并根据其中包含的信息采取行动。

#### Aggregate

聚合是一个实体或一组实体，总是保持一致的状态。聚合根是聚合树顶部的对象，负责保持这种一致的状态。这使得聚合成为在任何基于 CQRS 的应用程序中实现命令模型的主要构建块。
例如，“Contact(联系人)”聚合可以包含两个实体：联系人和地址。为了使整个聚合保持一致状态，应通过联系人实体为联系人添加地址。在这种情况下，联系人实体是指定的聚合根。

在 Axon 中，聚合由聚合标识符标识。这可能是任何对象，但是对于标识符的良好实现有一些准则。标识符必须:

1. 实现 eques 和 hashCode，以确保与其他实例进行良好的平等比较
2. 实现一个提供一致结果的 toString 方法
3. 实现 Serializable 成为可序列化的

当聚合使用不兼容的标识符时，测试模块将验证这些条件并通过测试。字符串、UUID 和数字类型的标识符始终适用。不要使用原始类型作为标识符，因为它们不允许延迟初始化。在某些情况下，Axon 可能会错误地将图元的默认值假定为标识符的值。

聚合总是通过一个名为聚合根的实体访问。通常，该实体的名称与聚合的名称完全相同。例如，订单聚合可能由一个 Order 实体组成，该实体引用多个 Orderline 实体。有序与有序相结合，形成集合。聚合是一个常规对象，它包含状态和改变该状态的方法。虽然根据 CQRS 原则并不完全正确，但也可以通过访问器方法公开聚合的状态。聚合根必须声明包含聚合标识符的字段。必须最迟在发布第一个事件时初始化此标识符。此标识符字段必须由 @AggregateIdentifier 注释进行注释。如果您使用JPA 并在聚合上有 JPA 注释，Axon 也可以使用 JPA 提供的 @ID 注释。聚合可以使用 AggregateLifecycle.Apply() 方法注册事件以供发布。与 EventBus 不同，EventBus 需要将消息包装在EventMessage 中，Apply() 允许您直接传递有效负载对象。

```java
@Entity // 标记聚合是一个 jpa 实体
public class MyAggregate {
    @Id // 使用 JPA @Id 进行注释时，不需要 @AggregateIdentifier 注释
    private String id;

    // fields containing state...

    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        // ... update state
        apply(new MyAggregateCreatedEvent(...));
    }

    // constructor needed by JPA
    protected MyAggregate() {
    }
}
```

聚合中的实体可以通过定义 @EventHandler 注释方法来侦听聚合发布的事件。这些方法将在 EventMessage 发布时调用(在任何外部处理程序发布之前)。

#### Event sourced aggregates

除了存储聚合的当前状态外，还可以根据过去发布的事件重新构建聚合的状态。要使此操作正常，所有状态更改都必须由一个事件表示。对于主要部分，事件源聚合类似于“常规”聚合：它们必须声明一个标识符，并且可以使用 Apply 方法发布事件。但是，事件源集合中的状态更改(即字段值的任何更改)必须在 @EventSourcingHandler 注释的方法中独占执行。这包括设置聚合标识符。注意，聚合标识符必须在聚合发布的第一个事件的 @EventSourcingHandler 中设置。这通常是创建事件。事件源聚合的聚合根还必须包含无 Arg 构造函数。AxonFramework 使用此构造函数在使用过去的事件初始化聚合实例之前创建一个空聚合实例。未能提供此构造函数将导致加载聚合时出现异常。

```java
public class MyAggregateRoot {

    @AggregateIdentifier
    private String aggregateIdentifier;

    // fields containing state...

    @CommandHandler
    public MyAggregateRoot(CreateMyAggregate cmd) {
        apply(new MyAggregateCreatedEvent(cmd.getId()));
    }

    // constructor needed for reconstruction
    protected MyAggregateRoot() {
    }

    @EventSourcingHandler
    private void handleMyAggregateCreatedEvent(MyAggregateCreatedEvent event) {
        // 确保标识符始终正确初始化
        this.aggregateIdentifier = event.getMyAggregateIdentifier();
        // ... update state
    }
}
```

在某些情况下，特别是当聚合结构超过了几个实体时，对在同一聚合的其他实体中发布的事件作出反应是比较干净的。但是，由于在重构聚合状态时也会调用事件处理程序方法，因此必须采取特殊的预防措施。可以在事件 SourcingHandler 方法中Apply() 新事件。这使得实体 B 能够将事件应用于实体 A 所做的事情。当重播历史事件时，Axon 将忽略 Apply() 调用。请注意，在本例中，内部 Apply() 调用的事件仅在所有实体接收到第一个事件之后才发布到实体。如果需要发布更多事件，则根据应用内部事件后实体的状态，使用 Apply(.).和 ThenApply(.) 您还可以使用静态的AggregateLifecycle.isLive() 方法来检查聚合是否是“live”。基本上，如果一个集合已经完成了历史事件的重播，它就被认为是实时的。在重播这些事件时，isLive() 将返回 false。使用此 isLive() 方法，您可以执行仅在处理新生成的事件时才应该执行的活动。

#### Complex Aggregate structures

复杂的业务逻辑通常所需要的不只是仅具有聚合根的聚合所能提供的。在这种情况下，重要的是将复杂性分布在聚合中的多个实体上。在使用事件源时，不仅聚合根需要使用事件来触发状态转换，而且聚合中的每个实体也需要使用事件来触发状态转换。
注意聚合不应公开状态的规则的一个常见误解是，任何实体都不应该包含任何属性访问器方法。 不是这种情况。 实际上，如果聚合中的实体向同一聚合中的其他实体公开状态，则聚合可能会受益匪浅。 但是，建议不要将状态暴露在聚合外部。

Axon 为复杂聚合结构中的事件源提供支持。就像聚合根一样，实体是简单的对象。声明子实体的字段必须使用@AggregateMember 注释。 此注释告诉 Axon，带注释的字段包含应检查命令和事件处理程序的类。

当一个实体(包括聚合根)应用一个事件时，它首先由聚合根处理，然后通过所有 @AggregateMember 注释字段冒泡直至其子实体。包含子实体的字段必须使用 @AggregateMember 注释。此批注可用于多种字段类型:

1. 在字段中直接引用的实体类型
2. 包含 Iterable(包含所有集合，例如 Set，List 等)的内部字段
3. 在包含 java.util.Map 的字段的值中

#### Handling commands in an Aggregate

建议直接在包含处理该命令状态的 Aggregate 中定义命令处理程序，因为命令处理程序不太可能需要该 Aggregate 的状态来完成其工作。

若要在聚合中定义命令处理程序，只需使用 @CommandHandler 注释命令处理方法。@CommandHandler 注释方法的规则与任何处理程序方法相同。然而，命令不仅是由它们的有效负载路由的。命令消息带有一个名称，默认为 Command 对象的完全限定类名。

默认情况下，@CommandHandler 注释方法允许以下参数类型:

1. 第一参数是命令消息的有效载荷。如果 @CommandHandler 注释明确定义了处理程序可以处理的命令的名称，则它也可以是类型消息或 CommandMessage。缺省情况下，命令名称是命令的有效负载的完全限定的类名称。
2. 用 @MetadataValue 注释的参数将以注释所示的键解析为元数据值。如果需要为 false(默认)，则在元数据值不存在时传递 NULL。如果需要，则解析器将不匹配，并且在元数据值不存在时阻止该方法被调用。
3. MetaData 类型的参数将注入 CommandMessage 的整个 MetaData。
4. 类型为 UnitOfWork 的参数获取当前注入的工作单位。这使命令处理程序可以注册要在工作单元的特定阶段执行的操作，或者访问对其注册的资源。
5. 类型为 Message 或 CommandMessage 的参数将获取完整的消息以及有效负载和元数据。如果方法需要几个元数据字段或包装的 Message 的其他属性，则此方法很有用。

为了使 Axon 知道哪个 Aggregate 类型的实例应处理命令消息，必须在 @CommandAggregateIdentifier 处注释Command 对象中带有 AggregateIdentifier 的属性。 注释可以放在字段或访问器方法(例如 getter)上。

创建聚合实例的命令无需标识目标聚合标识符，尽管建议也对它们进行注释。如果您更喜欢使用另一种机制来路由命令，则可以通过提供自定义 CommandTargetResolver 来覆盖该行为。该类应该根据给定的命令返回聚合标识符和预期版本(如果有的话)。

当 @CommandHandler 注释放置在聚合 Aggregate 构造函数上时，相应的命令将创建该聚合的新实例并将其添加到存储库中。这些命令不需要针对特定的聚合实例。因此，这些命令不需要任何 @TargetAggregateIdentifier 或@TargetAggregateVersion 注释，也不会为这些命令调用自定义 CommandTargetResolver。 当命令创建聚合实例时，当命令成功执行时，该命令的回调将接收聚合标识符。

```java
public class MyAggregate {

    @AggregateIdentifier
    private String id;

    @AggregateMember
    private MyEntity entity;

    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        apply(new MyAggregateCreatedEvent(...);
    }

    // no-arg constructor for Axon
    MyAggregate() {
    }

    @CommandHandler
    public void doSomething(DoSomethingCommand command) {
        // do something...
    }
}

public class MyEntity {

    @CommandHandler
    public void handleSomeCommand(DoSomethingInEntityCommand command) {
        // do something
    }
}
```

@CommandHandler 注释不限于聚合根。将所有命令处理程序放置在根中有时会导致聚合根的大量方法，而其中许多方法简单地将调用转发到底层实体之一。如果是这种情况，则可以将 @CommandHandler 注释放置在下面的实体之一“方法”上。对于Axon 找到这些注释的方法，必须用 @AggregateMember 标记在聚合根中声明实体的字段。请注意，只有声明类型的注释字段被检查为命令处理程序。如果在传入命令到达该实体时，字段值为 NULL，则会引发异常。

请注意，每个命令在聚合中必须有一个处理程序。这意味着您不能用使用 @CommandHandler 注释处理相同命令类型的多个实体(无论是根节点还是不是根节点)。如果需要有条件地将命令路由到实体，则这些实体的父实体应该处理该命令，并根据适用的条件转发命令。字段的运行时类型不必完全是声明的类型。但是，对于 @CommandHandler 方法，只检查@AggregateMenger 注释字段的声明类型。

也可以使用 @AggregateMember 注释包含实体的 Collections 和 Maps。在后一种情况下，映射的值应包含实体，而键包含的值将用作其引用。

由于需要将命令路由到正确的实例，因此必须正确标识这些实例。他们的“id”字段必须用 @EntityId 注释。将用于查找消息应该路由到的实体的命令上的属性，默认为已注释的字段的名称。例如，在注释字段“myEntityId”时，命令必须定义一个名称相同的属性。这意味着必须存在 getMyEntityId 或 myEntityId() 方法。如果字段的名称和路由属性不同，您可以使用 @EntityId(routingKey=“customRoutingProperty”) 显式地提供一个值。

如果在带注释的 Collection 或 Map 中找不到任何 Entity，则 Axon 会引发 IllegalStateException；否则，它将抛出 IllegalStateException。 显然，聚合不能在该时间点处理该命令。Collection 或 Map 的字段声明应包含适当的泛型，以允许 Axon 标识 Collection 或 Map 中包含的实体的类型。 如果无法在声明中添加泛型(例如，因为您使用的是已经定义了泛型类型的自定义实现)，则必须在 @AggregateMember 批注中指定 entityType 属性中使用的实体类型。

#### External Command Handlers

在某些情况下，不可能或希望将命令直接路由到 Aggregate 实例。在这种情况下，可以注册命令处理程序对象。
Command Handler 对象是一个简单的(常规)对象，具有 @CommandHandler 注释方法。与聚合不同，命令处理程序对象只有一个实例，该对象处理在其方法中声明的所有类型的命令。

```java
public class MyAnnotatedHandler {

    @CommandHandler
    public void handleSomeCommand(SomeCommand command, @MetaDataValue("userId") String userId) {
        // whatever logic here
    }

    @CommandHandler(commandName = "myCustomCommand")
    public void handleCustomCommand(SomeCommand command) {
       // handling logic here
    }

}

// 要将注释的处理程序注册到命令总线，请执行以下操作
Configurer configurer = ...
configurer.registerCommandHandler(c -> new MyAnnotatedHandler());
```

### Event Handling

Event listeners 是作用于传入事件的组件。它们通常基于由命令模型作出的决定来执行逻辑。通常，这包括更新视图模型或将更新转发到其他组件，例如第三方集成。在某些情况下，事件处理程序将根据收到的事件(模式)抛出事件，或者甚至发送命令来触发进一步的更改。

#### Defining Event Handlers

在 Axon 中，对象可以通过用 @eventHandler 注释它们来声明多个事件处理程序方法。该方法的声明参数定义了它将接收的事件.

Axon 为以下参数类型提供开箱即用的支持:

1. 第一个参数始终是事件消息的有效负载。如果事件处理程序不需要访问消息的有效负载，则可以在 @EventHandler 注释中指定预期的有效负载类型。当指定第一个参数时，将使用下面指定的规则解析第一个参数。如果希望将有效负载作为参数传递，请不要在注释上配置有效负载类型。
2. 用 @MetadataValue 注释的参数将以注释所示的键解析为元数据值。如果需要为 false(默认)，则在元数据值不存在时传递 NULL。如果需要，则解析器将不匹配，并且在元数据值不存在时阻止该方法被调用。
3. MetaData 类型的参数将注入 EventMessage 的整个 MetaData。
4. 用 @Timestamp 注释并且类型为 java.time.Instant(或 java.time.temporal.Temporal)的参数将解析为EventMessage 的时间戳。 这是生成事件的时间。
5. 用 @SequenceNumber 注释并且类型为 java.lang.Long 或 long 的参数将解析为 DomainEventMessage 的sequenceNumber。这提供了事件生成的顺序(在其起源的聚合范围内)。
6. 可分配给 Message 的参数将注入整个 EventMessage(如果消息可分配给该参数)。如果第一个参数的类型为message，则它有效地匹配任何类型的事件，即使通用参数会另外建议也是如此。由于类型擦除，Axon 无法检测到期望的参数。在这种情况下，最好声明有效负载类型的参数，然后声明消息类型的参数。
7. 使用 Spring 并激活 Axon 配置时(通过包括 Axon Spring Boot Starter 模块，或者通过在 @Configuration文件中指定 @EnableAxon)，如果在控制台中只有一个可注入的候选对象，则任何其他参数都将解析为自动装配的 bean。应用程序上下文。这使您可以将资源直接注入到 @EventHandler 带注释的方法中。

您可以通过实现 ParamResolerFactory 接口并创建名为 /META-INF/Service/org.axonframework.common.annotation.ParameterResolverFactory 的文件来配置其他参数解析器，其中包含实现类的完全限定名称。

在所有情况下，每个监听器实例最多调用一个事件处理程序方法。Axon 将使用以下规则搜索要调用的最特定方法:

1. 在类层次结构的实际实例级别上(由this.getClass())返回)，将评估所有带注释的方法。
2. 如果找到一个或多个方法可以将所有参数解析为一个值，则选择并调用具有最特定类型的方法。
3. 如果在该类层次结构的此级别上没有找到任何方法，则计算父类的方法是否相同。
4. 当到达层次结构的顶层，而没有找到合适的事件处理程序时，该事件将被忽略。

```java
public class TopListener {

    @EventHandler
    public void handle(EventA event) {
    }

    @EventHandler
    public void handle(EventC event) {
    }
}

public class SubListener extends TopListener {

    @EventHandler
    public void handle(EventB event) {
    }

}
```

在上面的示例中，SubListener 将接收 EventB 和 EventC(扩展EventB) 的所有实例。换句话说，TopListener 根本不会接收 EventC 的任何调用。由于 Eventa 不能分配给 EventB(它是它的超类)，这些类将由 TopListener 处理。

#### Registering Event Handlers

事件处理组件使用 EventHandlingConfiguration 类定义，该类被注册为带有全局 Axon 配置程序的模块。通常，应用程序将定义单个 EventHandlingConfiguration，但更大的模块化应用程序可以选择每个模块定义一个。

若要使用 @EventHandler 方法注册对象，请在 EventHandlingConfiguration 上使用 RegistrerEventHandler方法:

```java
// 定义EventHandlingConfiguration
EventHandlingConfiguration ehConfiguration = new EventHandlingConfiguration()
    .registerEventHandler(conf -> new MyEventHandlerClass());

// 该模块需要在 Axon 配置中注册。
Configurer axonConfigurer = DefaultConfigurer.defaultConfiguration().registerModule(ehConfiguration);
```