# 概览 #





## 12因子应用 ##

要构建云端应用和服务，仅将旧的整体封装在 _Docker Image_ 中并在 _Kubernetes_ 中运行是不够的。 我们重视 __Heroku__ 定义的称为 “[十二要素应用程序](https://12factor.net)” 的原则：[从透视角度看](https://content.pivotal.io/ebooks/beyond-the-12-factor-app)。 没有这些指导，就很难在分布式环境中进行扩展。 __Activiti Cloud__ 重新定位流程引擎，以更好地与分布式环境中的其他组件进行交互。衡量 __Activiti Cloud__ 成功的标准是与其他微服务及其设计，构建和部署方式的低阻抗失配。

### 1. 一个代码库，一个应用程序 ###

__Activiti__ 的示例服务都位于不同的存储库中，并且每个服务代表一个单独的Spring Boot应用程序，该应用程序也已通过 __Spring Cloud__ 库启用。 这些服务存储库中的每一个都包含一组工件，这些工件使它们适合于 _CI/CD_ 管道：

* __Jenkinsfile__ ：（或其他pipiline定义）管道，用于将当前服务构建，部署和提升到 _Kubernetes_ 集群。
* __Maven项目__ ：定义通过 _Spring Boot_ 和 _Activiti Cloud Starters_ 构建的服务。
* __Dockerfile__ ：定义如何为服务构建Docker映像
* __HELM图表__ ：定义清单集（ _kubernetes_ 描述符）以将服务部署到正在运行的 _Kubernetes_ 集群中。

__Activiti__ 为每个构建块提供一个 _Spring Boot Starter_ ，可以根据特定领域要求对其进行自定义和扩展。

* [Activiti Cloud Runtime Bundle](https://github.com/Activiti/activiti-cloud-runtime-bundle-service)
* [Activiti Cloud Query Service](https://github.com/Activiti/activiti-cloud-query-service)

* [Activiti Cloud Audit Service](https://github.com/Activiti/activiti-cloud-audit-service)
* [Activiti Cloud Connectors Service](https://github.com/Activiti/activiti-cloud-connectors)

### 2.API优先 ###

为每个服务定义了REST和消息驱动API，并且可以在组件文档中找到每个API的规范。 我们要确保合同定义明确，以便可以支持不同的实现。 支持不同实现（基于不同技术）的目标使 _Activiti_ 能够支持更广泛的场景（低延迟，数据密集型，自定义要求等）

### 3.依赖管理 ###

_Activiti Core_ 和 _Activiti Cloud_ 依赖关系使用物料清单（BoMs）进行管理，这些物料清单集中了服务和库的依赖关系。 这些 _BoMs_ 可以在以下两个存储库中找到：

* [Activiti Core Dependencies](https://github.com/Activiti/activiti-dependencies)
* [Activiti Cloud Dependencies](https://github.com/Activiti/activiti-cloud-dependencies)

### 4.设计、构建、发布、运行 ###

* __设计__：我们在设计，RFC和编码之间保持闭环，以确保我们快速构建可改善服务的小功能，并且可以频繁发布（至少每月一次）。
* __构建__ ：我们的 _CI_ 服务器构建我们的每项服务，并发布到 _Alfresco Nexus_ 和 _Maven Central_，因此可以依赖经常发布的版本。 可以使用环境变量来配置我们所有的服务，因此可以使它们保持不变，并且可以使用标准容器实践来更改其配置。
* __发布__ ：针对特定于领域的模块/服务，我们提供了机制，以确保您可以复制发布版本，以便在出现问题时进行审核和故障排除
* __运行__ ：我们提供分层的 _HELM图表_ ，使您可以轻松部署基础架构和应用程序。 可以在[入门指南]()中找到如何开始使用这些HELM图表。

### 5.配置 、凭证和编码 ###

所有 _Activiti Cloud_ 服务的配置均由 _env_ 变量提供，并且 _Spring Cloud_ 配置服务可用于系统/平台范围的配置。 这些配置可以按环境划分，通过使用 _Kubernetes_ 的 _Secrets_ ，我们可以与其他 _Secret provider_ 集成。

### 6.日志 ###

_Activiti Cloud_ 服务的日志将委派给基础架构。 可以连接 _ELK技术栈_ 来分析和监视日志。 环境变量用于配置日志的写入位置。 我们希望依靠通用工具进行监视和记录，而不是为此提供我们自己的自定义解决方案。

### 7.一次性 ###

_Activiti Cloud_ 服务旨在在30秒内快速启动，因此可以根据需要快速扩展。 这些服务不存储状态，因此基础架构可以随时对其进行处理和重新创建。 Activiti Cloud团队致力于缩短启动时间，并确保将前期操作保持在最低限度。

### 8.支持服务 ###

_Activiti Cloud_ 服务将受限资源的概念用于安全性，电子邮件，数据源，存储和其他资源。 它还使用服务注册表来抽象其他服务的位置以及它们之间的通信方式。

### 9.环境均势 ###

_Activiti Cloud_ 为用户和实施者提供了类似于生产的环境，以生产其特定于域的人工制品，这些人工制品随后可以发布到生产环境中。 _Activiti Cloud_ 依靠行业标准来确保尽可能轻松，真实地处理用于开发，测试/质量保证和生产的多个环境。 我们依靠诸如 _HELM_ 和 _GitOps_ 实践之类的技术来确保可以在不同集群之间轻松再现这些环境。

### 10.管理流程 ###

在 _Activiti Cloud_ 中，最初与流程引擎捆绑在一起的管理流程正在外部重构，以重用基础架构提供的服务。 在大多数开源BPM项目中，管理流程是主要问题之一。 因此，我们正在努力使每个流程都脱离_Process Engine_ 范围。

### 11.端口绑定 ###

所有 _Activiti Cloud_ 服务都是 _Spring Boot_ 应用程序，它们允许环境变量配置其名称（用于自动发现）及其端口（将在这些端口上运行）以用作其他应用程序和服务的后备服务。 我们所有的服务都在同一端口上运行，它们利用 _Kubernetes_ 网络进行映射和绑定。

### 12. 无状态进程 ###

_Activiti Cloud_ 服务在设计上是无状态的，并且如果它们需要存储状态，它们将绑定到数据源以进行存储。 缓存机制（例如Redis）将在需要时用于共享状态。

### 13.并发 ###

运行时捆绑包的设计具有扩展流程执行的想法-如果创建一个类型的流程实例的请求太多或与一组特定的流程实例的交互过多，则可以根据需要创建新的运行时捆绑包实例以处理负载 。 无需额外配置即可在_Activiti Cloud_ 中实现此可伸缩性-这是平台的固有功能。

### 14.遥测 ###

_Activiti Cloud_ 服务通过标准的 _Spring Boot_（千分尺）执行器和运行状况指示器提供遥测功能，但它还提供业务级别的遥测功能，从而发出暴露运行时操作的标准化事件集。 所有这些信息都可以用于数据仓库，报告和预测目的。

### 15.认证与授权 ###

_Activiti Cloud_ 依靠 _SSO_ 和 _IDM_ 来提供所有服务。 RBAC（基于角色的访问控制）是我们所有服务的固有组成部分。 _Activiti Cloud_ 组件将为特定于域的扩展提供挂钩点，以构建更复杂的用例。

## 角色 ##

重要的是要了解我们使用不同工具集所针对的角色。 因此，下图显示了我们将定位的主要角色，以及他们将用来执行工作的功能。

_Activiti Cloud_ 将首先针对 _Activiti_ 开发人员和 _Activiti DevOps_，后者负责设置基础架构并确保可将_Activiti_ 应用程序部署到云提供商。

### Activiti Cloud 开发者 ###

_Activiti Cloud_ 开发人员负责构建系统到系统的连接器，这些连接器将用作与现有内部/外部系统的集成点。 Process Engine（在我们的Runtime Bundle中）将不了解这些连接器，并且集成将通过Events进行。 这些云连接器将在运行时发现，它们将使用基础结构提供故障转移和回退机制。 开发人员将负责测试这些连接器，打包和发布它们（Maven / Docker）。 除了所需的连接器之外，开发人员还负责测试业务流程定义。 经过测试后，开发人员将负责发布这些运行时包和连接器。

### Activiti Cloud DevOps ###

_Activiti Cloud DevOps_ 负责获取已发布的运行时捆绑包和云连接器，并将其部署到我们的基础架构实例中。 DevOps将只能访问并能够获取开发人员已经测试并发布到特定存储库（Maven/Docker）的发行版本。

### 工具和集成 ###

对于开发人员和DevOps角色而言，_Activiti Cloud_ 团队一直在寻找可简化日常工作的工具。 我们发现以下工具非常有用：

* __Jenkins X__ ：CI/CD完全适合Kubernetes
* __[JHipster](https://www.jhipster.tech/)__ ：Spring Cloud和Angular/React生成器
* __HELM__ ：Kubernetes Package Manager，我们将其用于在K8s中部署Activiti Cloud应用程序。
* __[Spring Cloud Kubernetes](http://github.com/spring-cloud/spring-cloud-kubernetes/)__：我们用它来避免基础架构的重复/重叠
* __[Istio](https://istio.io/)__ ：K8之上的Service Mesh
* __KNative__ ：在K8之上充当服务层

## 之前的版本（5.x & 6.x） ###

_Activiti Cloud_ 的核心是 _Activiti 7.x_，它是 _Activiti 6.x_ 的演进。 但是，如前所述，_Activiti 7.x_ 将带来一些我们建议做事方式的变化。 其中大部分更改都是为了简化操作，以确保该框架本身不会促进公认的做法，这些做法会给现实生活中的实施带来极大的麻烦。 所有这些更改和简化都考虑到了云环境。 如果您使用的是 _Activiti 6.x_ 或 _5.x_ ，并且运转很好，请不要担心 _Activiti 7.x_ 会为您服务。 但是，如果您正在研究微服务和云环境，并且正遭受采用BPM工具和框架的困扰，那么 _Activiti Cloud_ 将为您解决大多数此类问题。 还需要注意的是，_Activiti Cloud_ 建立在行业广泛使用的屡获殊荣的框架之上，因此，您无需雇用专门的开发人员来扩展和改进 _Activiti Cloud Services_。

Activiti 7 at是核心，将维护流程引擎，但相关服务例如：

* IDM（用户/组/成员），
* 形式，
* 历史服务
* 工作任务执行器
* 计时器
* 电子邮件/通知服务
* 等等

将不推荐使用和重构，以支持与大多数云基础架构已提供的第三方组件集成。 所有这些重构的主要目标是确保 _Process Engine_ 的引导程序和占用空间最小，并且不会与基础结构预期提供的功能和行为重叠。

_Activiti Cloud_ 由来自不同BPM供应商和行业背景的工程师构建，我们的目标是解决这些其他专有或开源项目无法解决的问题。

_Activiti Cloud_ 背后的团队非常致力于开源实践，我们将采用我们的服务并将其与我们认为社区将从中受益的其他开源项目整合。 我们还将与其他以开放协作方式共享我们兴趣的开放源代码项目并进行协作。 如果您正在另一个开源项目中工作，并且有兴趣与我们集成或合作，请与我们取得联系。

## 参考文献 ##

如果您想了解项目的未来发展方向，建议阅读以下书籍和PDF列表。 我们将这些书用作基准共享语言，以讨论我们的体系结构决策和计划。

* OMG BPMN 2.x Spec

* [Cloud Native Java](https://www.amazon.co.uk/Cloud-Native-Java-Designing-Resilient/dp/1449374646/ref=sr_1_1?ie=UTF8&qid=1531487284&sr=8-1&keywords=cloud+native+java)

* [Building Microservices](https://www.amazon.co.uk/Building-Microservices-Designing-Fine-Grained-Systems/dp/1492034029/ref=sr_1_3?s=books&ie=UTF8&qid=1531487341&sr=1-3&keywords=building+microservices)

* [Implementing Domain Driven Designs](https://www.amazon.co.uk/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577/ref=sr_1_2?s=books&ie=UTF8&qid=1531487429&sr=1-2&keywords=domain+driven+design)

* [BPMN method and style](https://www.amazon.co.uk/Bpmn-Method-Style-Implementers-Guide/dp/0982368119/ref=sr_1_1?ie=UTF8&qid=1531487366&sr=8-1&keywords=bpmn+method+and+style)

* [12 factor apps](http://12factor.net/)

* [Beyond the 12 factor app](https://content.pivotal.io/blog/beyond-the-twelve-factor-app)

* [Kubernetes in Action](https://www.amazon.co.uk/Kubernetes-Action-Marko-Luksa/dp/1617293725/ref=sr_1_1?s=books&ie=UTF8&qid=1531487409&sr=1-1&keywords=kubernetes+in+action)

## Cloud Native BPMN支持 ##

BPMN规范描述了业务流程定义中允许的大量构造（bpmn元素）。 虽然在 _Activiti 5.x_ 和 _6.x_ 中都由流程引擎来支持，但是在 _Activiti Cloud（Activiti Core 7.x）_ 中，由于我们现在正在处理分布式且高度可扩展的基础结构，因此所支持的元素更少。 计时器，信号和消息等元素现在涉及与基础架构和其他服务的交互，以使其正常工作。 因此，_Activiti Cloud_ 的第一个版本选择了这些元素的子集来构建坚实的基础，该基础可以确保分布式环境中一组组件之间的执行能够按预期运行，并在出现问题时可以进行监视和跟踪 。

作为此计划的一部分，我们定义了一致性集以验证由BPMN元素的不同组合组成的不同用例。 这些性能测试使用新的 _Java API_ 和 _Cloud Native API_ 来验证我们所有组件的正确行为。 您可以在此处找到[符合性方案](https://activiti.gitbook.io/activiti-7-developers-guide/overview/comformance)。

7.1.x发行培训中支持的BPMN元素的列表为：

* Start / End Events
* SequenceFlows (conditional, default)
* Service Task
* User Task (assignee, candidateUsers, candidateGroups)
* Gateways: Parallel, Exclusive, Inclusive
* Call Activity
* Signal Intermediate Catch Event, Signal Intermediate Throw Event, Signal Boundary Event
* Embedded Subprocess

这些BPMN元素当前在 _Activiti Modeler_ 应用程序中可用，随着我们为其余每个元素增加更多的支持，测试和一致性，我们将启用更多的BPMN元素。

其余的BPMN元素将在将来的版本中涵盖。 我们重视社区的反馈，我们将根据社区认为首先应得到支持的优先级进行排序。 您可以在此处找到路线图和为支持这些元素而创建的问题。 如果找不到您要查找的内容，请创建一个新的期刊，我们将对它们进行相应分类。 如果发现描述您要使用的BPMN元素的问题，请在github中将其投票（通过添加反应或评论），以便我们可以根据此反馈确定优先级。

## BPMN  ##

本部分的目的是逐步描述 _Activiti Core_ 和 _Activiti Cloud_ 版本需要涵盖的方案。 这些测试是自动化的，以确保在将来的版本中，我们不会引入可能破坏其中某些/全部情况的回归分析。

本文档分为不同部分，分别针对本地和分布式环境的执行的不同方面。 这意味着测试应涵盖执行中涉及的每个不同服务中如何保持执行和状态。

这些场景还需要定义发送的数据类型/有效负载以及从消费者角度出发的预期输出（这意味着试图通过可用的API访问状态）。

为了对什么是有效的什么不是有一个敏感的认识，需要涵盖不同的方面：

* 本地 VS 分布式
* 用户 VS 管理员API
* 安全策略执行
* 基本，中等和高级/复杂方案
* 数据处理 & 持久性
* 性能（可以单独分析）

基于这些维度，我们将按以下顺序介绍不同的组合：

* 使用本地USER API（ACTIVITI_USER角色API）的基本方案：
  - 基本方案单独测试BPMN元素的执行。 我们应该测试快乐和不快乐的路径，以了解在执行过程中出现问题时会产生哪些错误。
  - [Set #0: Process Information]()
  - [Set #1: Service Task]()
  - [Set #2: User Task]()
  - [Set #3: User Task Assignments]()
  - [Set #4: Gateways]()
  - [Set #5: Call Activity]()
  - [Set #6: Signals]()
* 使用分布式USER API的基本方案
* 使用本地USER API和数据处理的基本方案
* 在数据处理中使用分布式USER API的基本方案
* 使用本地ADMIN API的基本方案
* 使用分布式ADMIN API的基本方案

这些测试的代码可以在这里找到：

* Activiti Core Conformance Scenarios
* Activiti Cloud Conformance Scenarios

这些方案用于确定我们的发布。 这意味着仅在这些一致性测试为绿色时才发布Activiti依赖关系和Activiti云依赖关系。

### BPMN Comformance Set 0 ###

这些场景测试了基本流程的构造，例如BPMN的开始/结束事件，并且应该测试这些元素生成的事件。 它还可以在没有配置的情况下测试基本任务的行为。 您可以在此处找到这些性能测试的源代码。

* 流程信息
  - 该测试的源代码可以在[这里](https://github.com/Activiti/Activiti/blob/develop/activiti-spring-conformance-tests/activiti-spring-conformance-set0/src/test/java/org/activiti/spring/conformance/set0/ConformanceBasicProcessInformationTest.java)找到。
  - 启动后检查流程实例是否已完成（状态）
  - 检查流程实例信息是否正确传播
    - businessKey
    - Name
  - 开始流程操作
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
  - 开始流程操作
    - 添加一个流程变量
    - PROCESS_CREATED
    - VARIABLE_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 通用任务流程
  - 该测试的源代码可以在[这里]()找到
  - 启动后检查流程实例是否已完成（状态），BPMN通用任务没有任何相关行为，因此引擎会自动完成该任务
  - 检查流程实例信息是否正确传播
    - businessKey
    - Name
  - 开始流程操作
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 流程实例删除
  - 该测试的源代码可以在[这里]()找到
  - 以等待/安全状态（例如UserTask）启动流程实例
  - 我们只需要检查流程事件，任务事件应该在Set 2中进行验证
    - 开始流程操作
      - PROCESS_CREATED
      - PROCESS_STARTED
      - ACTIVITY_STARTED
      - ACTIVITY_COMPLETED
      - SEQUENCE_FLOW_TAKEN
      - ACTIVITY_STARTED
    - 删除流程操作
      - ACTIVITY_CANCELLED
      - PROCESS_CANCELLED
  - 删除后，我们将无法找到流程实例
* 流程实例挂起/恢复
  - 该测试的源代码可以在[这里]()找到
  - 以等待/安全状态（例如UserTask）启动流程实例
  - 开始流程操作
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
  - 挂起流程操作
    - PROCESS_SUSPENDED
  - 恢复流程操作
    - PROCESS_RESUMED

### BPMN Comformance Set 1 ###

这些场景涵盖了BPMN Service任务的使用以及由包括此类任务的流程生成的事件。 为了涵盖可能由外部（第三方集成）引起的副作用，这些方案涵盖并验证了过程变量的变化。 这些测试的源代码可以在[这里]()找到。

* 具有实现服务任务
  - 由于连接器是同步操作，因此请检查进程启动以及启动后的状态是否已完成
  - 检查连接器逻辑是否已执行
  - 检查是否发出以下事件
  - 开始流程操作
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 具有实现和变量的服务任务
  - 由于连接器是同步操作，因此请检查进程启动以及启动后的状态是否已完成
  - 检查连接器逻辑是否已执行以及变量是否已修改
  - 检查是否发出以下事件
  - 开始流程操作
    - PROCESS_CREATED
    - VARIABLE_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - VARIABLE_UPDATED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
  - 检查VARIABLE_UPDATED事件是否包含修改后的值
* 没有实现的服务任务

### BPMN Comformance Set 2 ###

此业务情景检查用户任务和基本分配属性。 它还检查任务运行时生成的事件。 任务操作用于与任务资源进行交互。 这些测试的源代码可以在[这里]()找到。

* 具有用户分配的用户任务
  - 我们应该能够启动该过程，并检查启动后的状态是否为RUNNING
  - 该流程实例有一个具体的受让人：“ user1”
  - 我们应该为分配的用户查询任务，并检查是否只有一项任务
  - 我们应该检查User2没有任何任务
  - 我们应该能够使用User1通过id获取任务
  - 开始流程操作：
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - TASK_CREATED
    - TASK_ASSIGNED
    - TASK_UPDATED
  - User1应该能够完成任务
  - 任务完成操作
    - TASK_COMPLETED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 候选用户的用户任务
  - 我们应该能够启动该过程，并检查启动后的状态是否为RUNNING
  - 该流程实例没有具体的受让人
  - 我们应该查询候选用户的任务，并检查是否只有一项任务
  - 我们应该检查User2没有任何任务
  - 我们应该能够使用User1通过id获取任务
  - 我们应该检查User2是否无法通过ID获取任务
  - 在设置受让人之前，User1或User2都不应该能够完成任务（找不到错误User2，没有受让人的情况下User1无法完成）
  - User1应该可以声明任务
  - 索偿后，应确定受让人
  - 如果User1是受让人，则他/她应该能够完成任务
  - 我们应检查释放后受让人是否已被移走
  - 开始流程操作：
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - TASK_CREATED
  - 领取任务后，我们应该得到以下事件
  - 领取任务操作：
    - TASK_ASSIGNED
    - TASK_UPDATED
  - 释放任务后，我们应该得到以下事件
  - 下达任务操作：
    - TASK_COMPLETED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 带有候选用户声明和发布任务的用户任务
  - 我们应该能够启动该过程，并检查启动后的状态是否为RUNNING
  - 开始流程操作
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - TASK_CREATED
  - 该流程实例没有具体的受让人
  - 我们应该查询候选用户的任务，并检查是否只有一项任务
  - 我们应该检查User2没有任何任务
  - User1应该可以声明任务
  - 任务状态应为ASSIGNED
  - User1应该可以释放任务
  - 任务状态应为CREATED
  - 领取任务操作
    - TASK_ASSIGNED
    - TASK_UPDATED
  - 下达任务操作
    - TASK_ASSIGNED
    - TASK_UPDATED
* 候选组的用户任务
  - 我们应该能够启动该过程，并检查启动后的状态是否为RUNNING
  - 该流程实例没有具体的受让人
  - 我们应该查询候选组Group1，User1和User3的任务获得一项任务
  - 我们应该检查User2没有任何任务
  - 我们应该能够使用User1和User3通过id获取任务
  - 我们应该检查User2是否无法通过ID获取任务
  - 在设置受让人之前，User1或User2都不应该能够完成任务
  - User1应该声明任务
  - User3将不再看到该任务，因为它已分配给User1
  - **开始流程操作***
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - TASK_CREATED
  - **领取任务操作**
    - TASK_ASSIGNED
    - TASK_UPDATED
  - **下达任务操作**
    - TASK_COMPLETED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 具有候选组声明和释放任务的用户任务
  - 我们应该能够启动该过程，并检查启动后的状态是否为RUNNING
  - **开始流程操作**
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - TASK_CREATED
  - 该流程实例没有具体的受让人
  - 我们应该查询候选用户的任务，并检查是否只有一项任务
  - 我们应该检查User2没有任何任务
  - 我们应该能够使用User1通过id获取任务
  - 我们应该检查User2是否无法通过ID获取任务
  - 在设置受让人之前，User1或User2都不应该能够完成任务
  - **领取任务操作**
    - TASK_ASSIGNED
    - TASK_UPDATED
  - **下达任务操作**
    - ASK_ASSIGNED
    - TASK_UPDATED
  - 发布后，User3和User1应该能够再次看到任务
* 没有分配的用户任务
  - 我们应该能够启动该过程，并检查启动后的状态是否为RUNNING
  - User1或User2都不能看到任何任务，因为该任务没有受让人，候选人用户或候选人组
  - 具有**ACTIVITI_ADMIN**角色的用户应该能够看到任务，但稍后会在Admins上进行更多介绍
* 受让人删除的用户任务
  - 进程创建任务，用户对它执行删除
  - 用户是任务的受让人，因此无需声明
  - 在属于某个进程的任务上调用delete会产生ActivitiException：“由于该任务是正在运行的进程的一部分，因此无法删除该任务”
* 候选删除的用户任务
  - 进程创建任务，用户对它执行删除
  - 用户是任务的候选人
  - 在属于进程的任务上调用delete应该会产生ActivitiException
* 用户任务详细信息待定
* 用户任务，带数据/变量待定

### BPMN Comformance Set 3 ###

当提供了具有不同配置的用户任务时，此方案将更深入地进行分配并检查API是否返回了正确的数据。

* 将用户任务1分配给用户1，然后将用户任务2分配给候选组1
  - 用户1应该看到已分配的用户任务
  - 用户1应该能够完成他/她的任务
  - 用户1和用户3应该在候选位置看到下一个任务
  - 用户2永远不会看到任何任务
* 带有候选组1的用户任务1，然后带有候选组2的用户任务2
  - 用户1应该只看到任务1
* 具有任务组1的用户任务1，用户1声明并添加了候选用户2
  - 用户1应该可以声明任务
  - 用户1应该能够添加候选用户2
  - 用户1应释放任务
  - 用户2应该能够将任务视为候选者
* 具有任务组1的用户任务1，用户1声明并添加了候选组2
  - 用户1应该可以声明任务
  - 用户1应该能够添加候选用户2
  - 用户1应释放任务
  - 用户2应该能够将任务视为候选者



### BPMN Comformance Set 4 ###

* 排他网关，具有分配给用户1和用户2的两个用户任务
  - 由于一个简单的表达式，用户1应该看到在独占网关之后创建的任务
* 表达式错误的独占网关
  - 它应该在部署时失败
  - 现在，我们在执行网关后看到一个异常。 由失败的第一个序列流报告异常。
* 具有分配给用户1和用户2的两个用户任务的并行网关
  - 用户1应该看到在并行网关之后分配的任务
  - 用户2应该看到在并行网关之后分配的任务
  - 用户3应该看不到任何任务
* 具有分配给组1和组2的两个用户任务的并行网关
  - 用户1应该看到在并行网关之后创建的任务
  - 用户2应该看到在并行网关之后创建的任务
  - 用户3应该将两个任务都视为候选者



### BPMN Comformance Set 5 ###

* 子流程的父流程包含一个带有受让人User1的用户任务
* 带有包含服务任务及其实现的子流程的父流程



### BPMN Comformance Set 6 ###

此方案涵盖了不同组合中的信号事件。 这些测试的源代码可以在[这里]()找到。

* 流程引发信号事件
  - 开始流程操作：
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 带有信号中间捕获事件的过程
  - 在开始此过程之后，我们应该有以下事件（过程正在等待匹配信号）：
  - 开始流程操作
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
  - **信号操作**
    - 发送匹配信号后，我们应该发生以下事件：
    - SIGNAL_RECEIVED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED
* 信号边界事件处理
  - 启动过程后，我们应该发生以下事件
  - **开始流程操作**
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - TASK_CREATED
  - 发送与边界信号事件匹配的信号后，我们应该有以下事件：
  - **信号操作**
    - SIGNAL_RECEIVED
    - TASK_CANCELLED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - TASK_CREATED
* 信号开始事件
  - 发送与信号开始事件匹配的信号时，我们应该有以下事件：
  - **信号操作**
    - SIGNAL_RECEIVED
    - PROCESS_CREATED
    - PROCESS_STARTED
    - ACTIVITY_COMPLETED
    - SEQUENCE_FLOW_TAKEN
    - ACTIVITY_STARTED
    - ACTIVITY_COMPLETED
    - PROCESS_COMPLETED

# 入门 #

欢迎使用本教程，了解如何开始使用Activiti。 有两种部署选项，您可以尝试两种选择，也可以选择最适合您的需求的一种。 享受您的Activiti动手课程，现在该练习了！

**Activiti Cloud入门**：这是3个选项。

* Docker Compose（本地安装）
* 带有 Jenkins-X 的自动化 CI/CD
  - on AWS EKS
  - on GCP GKE
* 使用 Helm
  - on AWS EKS
  - on GCP GKE

Activiti Core入门：学习如何在Spring Boot应用程序中使用新的Java Runtime API。 这种Spring启动方法是将Activiti Core用作Java应用程序内部的库。

### Activiti Cloud 入门 ###

#### Activiti Cloud入门 ####

Activiti Cloud是一组从头开始设计的Cloud Native组件，可在分布式环境中使用。 我们选择了Kubernetes作为我们的主要部署基础架构，并且我们将Spring Cloud / Spring Boot与Docker一起用于这些组件的容器化。

我们经历了一段非常宝贵的旅程，遇到了非常热心的开发人员，社区以及希望利用这些技术（和业务自动化解决方案）来缩短上市时间并提高云中业务敏捷性的现有和潜在客户。 我们还为这些社区做出了贡献，确保我们消耗的开源项目获得了宝贵的反馈和贡献。

Activiti Cloud包括5个基本构建基块：

* Activiti Cloud Runtime Bundle
* Activiti Cloud Query
* Activiti Cloud Audit
* Activiti Cloud Connectors
* Activiti Cloud Notifications Service (GraphQL)

这些构建模块是Spring Boot Starters，可以附加到任何Spring Boot（2.x）应用程序。 这些构建模块通过提供Cloud Native功能的Spring Cloud功能进行了增强。

通过使用这些组件，您可以创建具有以下功能的Activiti Cloud应用程序：

* 可以根据需求独立缩放
* 可以完全隔离的周期进行管理
* 可以独立升级和维护
* 可以使用正确的工具提供特定领域的功能

在本教程中，我们想展示如何通过在Kubernetes中部署这些构建基块的示例集来入门。 我们强烈建议您使用真实的Kubernetes集群，例如GKE，PKS或EKS。 我们已经在AWS中测试了此博客文章的内容（使用Kops，PKS，GKE以及Jenkins X）

让我们开始使用Kubernetes，HELM和Activiti Cloud。

#### 先决条件 ####

将事物部署到Kubernetes的最快，最简单的方法是使用HELM图表。 如官方文档中所述，HELM是：“_一种简化安装和管理Kubernetes应用程序的工具。 可以把它想成Kubernetes的apt / yum / homebrew。_”

作为Activiti Cloud的一部分，我们创建了一组层次结构的HELM图表，可用于部署多个组件，其中一些与基础架构相关（例如SSO和Gateway），以及一些特定于应用程序的组件，例如Runtime Bundle，Audit Service，Query Service和 云连接器。

在本[快速入门](https://github.com/Activiti/activiti-cloud-full-chart/tree/master/charts/activiti-cloud-full-example)中，我们将更具体地关注 和 [Activiti Cloud Query](https://github.com/Activiti/activiti-cloud-query/tree/master/charts/activiti-cloud-query)  

父图表的通用部分位于[https://github.com/Activiti/activiti-cloud-common-chart/tree/master/charts/common](https://github.com/Activiti/activiti-cloud-common-chart/tree/master/charts/common) 

所有图表档案已移至[https://github.com/Activiti/activiti-cloud-helm-charts ](https://github.com/Activiti/activiti-cloud-helm-charts )，通用图表是位于以下位置的所有图表的基本图表[https://github.com/Activiti/activiti-cloud-common-chart](https://github.com/Activiti/activiti-cloud-common-chart)。位于组件文件夹中的组件的图表，例如：

Runtime - [https://github.com/Activiti/example-runtime-bundle/tree/master/charts/runtime-bundle](https://github.com/Activiti/example-runtime-bundle/tree/master/charts/runtime-bundle)

Example cloud connector - [https://github.com/Activiti/example-cloud-connector/tree/master/charts/activiti-cloud-connector](https://github.com/Activiti/example-cloud-connector/tree/master/charts/activiti-cloud-connector)

Audit - [https://github.com/Activiti/activiti-cloud-audit/tree/master/charts/activiti-cloud-audit](https://github.com/Activiti/activiti-cloud-audit/tree/master/charts/activiti-cloud-audit)

Query - [https://github.com/Activiti/activiti-cloud-query/tree/master/charts/activiti-cloud-query](https://github.com/Activiti/activiti-cloud-query/tree/master/charts/activiti-cloud-query)

此“ Activiti Cloud完整示例”部署了以下组件：

需要注意的重要一件事是，每个Activiti Cloud组件都可以独立使用。 此示例旨在显示大规模部署方案。 您可以从一个Runtime Bundle（提供流程和任务运行时）开始，但是如果您想扩大规模，则需要知道您的目标，并且此图表准确地显示了这一点。

#### 安装Kubectl和HELM ####

* Kubectl : [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* HELM: [https://github.com/helm/helm/releases/tag/v2.16.1](https://github.com/helm/helm/releases/tag/v2.16.1) 请使用HELM版本2。

在下一部分中，我们将向您展示如何使用Amazon Web Services EKS或Google Cloud Platform GKE创建Kubernetes集群。 我们让您决定最适合您的云平台。 您还可以使用例如Docker Desktop在本地计算机上部署Activiti Cloud full示例。 我们建议您使用云基础架构以获得更快，更流畅的体验，但是如果您需要本地安装，则可以在[此处](https://community.alfresco.com/community/bpm/blog/2018/12/10/getting-started-with-activiti-7-beta#jive_content_id_Deploying_and_Running_a_Business_Process)查看我们的博客文章系列。

#### 步骤1和2：创建K8集群并进行配置 ####

[选项 A:  使用 Amazon EKS ](https://activiti.gitbook.io/activiti-7-developers-guide/getting-started/getting-started-activiti-cloud/amazon-eks)

[选项 B:  使用 Google Cloud - GKE](https://activiti.gitbook.io/activiti-7-developers-guide/getting-started/getting-started-activiti-cloud/google-cloud-gke)

#### 步骤3：部署Activiti Cloud Full示例 ####

第一步是运行以下命令将Activiti Cloud HELM图表注册到HELM中：

```bash
helm repo add activiti-cloud-helm-charts https://activiti.github.io/activiti-cloud-helm-charts
helm repo update
```

可以自定义Activiti Cloud完整示例图表以打开和关闭不同的功能，但是需要提供一个强制性参数，该参数是此安装将使用的外部域名：

##### 1-a）为AWS配置部署 #####

> 对于此步骤，您需要一个公共域名。 如果您没有，请使用Route 53注册一个新的公共域名。

转到AWS管理控制台，然后打开Route 53控制台。 转到“托管区域”，然后选择一个公共托管区域并创建一个新的记录集。 使用“ *”字符将其命名，以创建通配符。 在“别名目标”中，选择我们之前部署的Ingress控制器（ELB）的DNS名称。

##### 1-b）为GCP配置部署 #####

对于GCP，请使用“ <EXTERNAL-IP> .nip.io”部署Activiti Helm图表。 在我们的案例中：35.194.42.164.nip.io

##### 2) 部署HELM图 ####

解析域名后，请通过使用公用域名运行Helm install命令来设置Helm图表来设置global.gateway.domain密钥。 在我们的情况下，将字符串“ REPLACEME”替换为上一步中的域。

```bash
helm install --name example activiti-cloud-helm-charts/activiti-cloud-full-example --version 7.1.0-M7 --set global.gateway.domain=REPLACEME --set activiti-cloud-identity.alfresco-identity-service.keycloak.postgresql.persistence.existingClaim=""
# 对于AWS，我们使用：
global.gateway.domain=raphaelallegre.com
# 对于GCP，我们使用：
global.gateway.domain=35.194.42.164.nip.io
# Here is the example result for AWS:
NOTES:
               _   _       _ _   _    _____ _                 _ 
     /\       | | (_)     (_) | (_)  / ____| |               | |
    /  \   ___| |_ ___   ___| |_ _  | |    | | ___  _   _  __| |
   / /\ \ / __| __| \ \ / / | __| | | |    | |/ _ \| | | |/ _` |
  / ____ \ (__| |_| |\ V /| | |_| | | |____| | (_) | |_| | (_| |
 /_/    \_\___|\__|_| \_/ |_|\__|_|  \_____|_|\___/ \__,_|\__,_|
 Version: 7.1.0-SNAPSHOT

Thank you for installing activiti-cloud-full-example-7.1.0-M4

Your release is named example.

To learn more about the release, try:

  $ helm status example
  $ helm get example

Get the application URLs:

Activiti Gateway         : http://gateway.default.alfrescodemo.co.uk/
Activiti Identity        : http://identity.default.alfrescodemo.co.uk/auth
Activiti Modeler         : http://gateway.default.alfrescodemo.co.uk/modeling
Activiti Runtime Bundle  : http://gateway.default.alfrescodemo.co.uk/rb
Activiti Cloud Connector : http://gateway.default.alfrescodemo.co.uk/example-cloud-connector
Activiti Query           : http://gateway.default.alfrescodemo.co.uk/query
Activiti Audit           : http://gateway.default.alfrescodemo.co.uk/audit
Notifications GraphiQL   : http://gateway.default.alfrescodemo.co.uk/notifications/graphiql
Notifications WebSockets : http://gateway.default.alfrescodemo.co.uk/notifications/ws/graphql
Notifications Graphql    : http://gateway.default.alfrescodemo.co.uk/notifications/graphql

To see deployment status, try:

  $ kubectl get pods -n default
raphaels-mbp-1:development raphaelallegre$ 
```

以下是BPMN 2建模应用程序。 默认用户：建模者/密码。

有关BPMN建模应用程序的更多信息，请查看以下博客文章。

#### 步骤4：使用部署的服务 ####

如果尚未安装，请在计算机上安装Postman客户端。

然后，使用以下命令从Activiti Cloud Examples存储库下载Activiti Cloud Postman集合：

```bash
curl -o Activiti_v7_REST_API.postman_collection.json https://raw.githubusercontent.com/Activiti/activiti-cloud-examples/develop/Activiti%20v7%20REST%20API.postman_collection.json
```

使用导入按钮在Postman中导入集合。

在调用任何服务之前，您将需要在Postman中创建一个新的环境。 您可以转到“管理环境”图标（cog）

然后“添加”新环境并为其添加名称。 现在，您需要为环境配置变量：“ gateway”，“ idm”和“ realm”

对于网关，您需要复制与您的Ingress相关联的url，对于idm来说也是如此，即使用Keycloak的SSO和IDM。 对于领域，输入“ activiti”：

单击保存或更新，然后就可以开始使用该环境了。 确保在右侧的下拉列表中选择环境：

如上一个屏幕截图所示，如果您进入keycloak目录并选择“ getKeycloakToken testuser”，您将获得令牌，该令牌将用于验证其他请求。 请注意，此令牌是时间敏感的，它将自动失效，因此，如果您开始收到未经授权的错误，则可能需要再次获取它。

获得用户令牌后，便可以与所有用户端点进行交互。 例如，您可以创建一个请求，以查看在示例运行时捆绑包中部署了哪些流程定义：

现在，您还可以从我们的SimpleProcess启动一个新的流程实例：

您可以检查审核服务是否包含与刚启动的流程实例关联的事件。

并且查询服务已经包含有关流程执行的信息：

现在，您准备开始使用这些服务来自动化自己的业务流程。

最后，您可以通过将浏览器指向以下位置来访问所有Swagger服务文档：

我们所有的服务都使用SpringFox生成此文档并为其提供UI。

#### 总结 ####

在本教程中，我们了解了如何创建Kubernetes集群（使用GKE或EKS）以及如何使用Activiti Cloud HELM图表部署Activiti Cloud应用程序。 如果您不熟悉Kubernetes，Docker和GKE或AWS，这可能看起来像很多新信息，我们的任务是简化这些入门指南中涵盖的所有步骤。 因此，我们建议您检出Jenkins X项目，该项目大大简化了有关为项目创建集群和配置基本基础结构的前两部分。

作为Activiti Cloud计划的一部分，我们确保遵循Kubernetes，Docker和Spring Cloud社区的最佳实践，并且我们将通过修复和反馈做出贡献，以使该技术最适合Cloud Native应用程序。

如果您对本教程有任何疑问或反馈，请随时通过专用的Gitter渠道与Activiti团队联系。

### Amazon EKS ###

#### 步骤1：创建一个Kubernetes集群 ####

##### 1）为Amazon EKS安装 aws-iam-authenticator #####

Amazon EKS集群需要Kubernetes的AWS IAM身份验证器才能对Kubernetes集群进行IAM身份验证。 使用go get安装aws-iam-authenticator二进制文件：

```bash
go get -u -v github.com/kubernetes-sigs/aws-iam-authenticator/cmd/aws-iam-authenticator
```

> *Note: 使用 Go 1.7 或更高版本*

将 `$HOME/go/bin` 添加到PATH环境变量中：

* macOS

  ```bash
  export PATH=$HOME/go/bin:$PATH && echo 'export PATH=$HOME/go/bin:$PATH' >> ~/.bash_profile
  ```

* Linux

  ```bash
   export PATH=$HOME/go/bin:$PATH && echo 'export PATH=$HOME/go/bin:$PATH' >> ~/.bashrc
  ```

运行以下命令以测试aws-iam-authenticator二进制文件是否有效：

```bash
aws-iam-authenticator help
```

##### 2) 安装AWS CLI #####

要安装 aws cli，请查看用户指南：[https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

安装后，使用以下命令检查您的AWS CLI版本：

```bash
aws --version
```

输出示例：

```bash
aws-cli/1.16.87 Python/3.7.2 Darwin/18.2.0 botocore/1.12.77
```

> Note: 系统的 Python 版本必须是 Python 3, 或 Python 2.7.9 或更高版本. 否则，您收到与Amazon EKS的AWS CLI调用的主机名不匹配的错误。

使用以下命令将您的AWS CLI配置为与您的AWS账户进行交互：

```bash
aws configure
AWS Access Key ID [None]: <your-access-key-ID>
AWS Secret Access Key [None]:<your-secet-access-key>
Default region name [None]: <your-region>
Default output format [None]: json
```

##### 3) 创建一个EKS集群 #####

### Activiti Core 入门 ###

#### Activiti Core Runtime API 入门 ####

创建新API的明确目的是满足以下要求：

* 为我们的云方法提供一条清晰的道路
* 隔离内部和外部API以提供向后兼容
* 通过遵循单责任方法为模块化提供未来的途径
* 减少以前版本的API的混乱情况
* 将安全和身份管理纳入头等公民
* 您希望依靠流行框架提供的约定，从而缩短常见用例的价值实现时间
* 提供基础服务的替代实现
* 在尊重既定合同的同时使社区创新

我们尚未弃用旧的API，因此您仍然可以自由使用它，但是我们强烈建议您使用新的API以获得长期支持。

该API处于Beta测试阶段，这意味着我们可能会在GA发布之前对其进行更改和完善。 我们将非常感谢社区用户提供的所有反馈，如果您希望参与该项目，请与我们联系。

是时候让我们看几个示例项目了。

#### TaskRuntime API ####

如果要构建业务应用程序，那么为组织中的用户和组创建任务可能会很方便。

TaskRuntime API在这里可以为您提供帮助。

您可以从GitHub克隆此示例：[https://github.com/Activiti/activiti-examples](https://github.com/Activiti/activiti-examples)

该部分的代码可以在“ activiti-api-basic-task-example” maven模块内找到。

如果您在Spring Boot 2应用程序中运行，则只需添加activiti-spring-boot-starter依赖项和一个数据库驱动程序，即可将H2用于内存中存储。

```xml
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

我们建议使用我们的BOM（材料清单）

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.activiti.dependencies</groupId>
            <artifactId>activiti-dependencies</artifactId>
            <version>7.1.0.M5</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后，您将可以使用TaskRuntime

```java
@Autowired
private TaskRuntime taskRuntime;
```

将bean注入应用程序后，您应该能够创建任务并与任务交互。

```java
public interface TaskRuntime {
  TaskRuntimeConfiguration configuration();
  Task task(String taskId);
  Page tasks(Pageable pageable);
  Page tasks(Pageable pageable, GetTasksPayload payload);
  Task create(CreateTaskPayload payload);
  Task claim(ClaimTaskPayload payload);
  Task release(ReleaseTaskPayload payload);
  Task complete(CompleteTaskPayload payload);
  Task update(UpdateTaskPayload payload);
  Task delete(DeleteTaskPayload payload);
  ...
}
```

例如，您可以通过执行以下操作来创建任务：

```java
taskRuntime.create(
            TaskPayloadBuilder.create()
                .withName("First Team Task")
                .withDescription("This is something really important")
                .withGroup("activitiTeam")
                .withPriority(10)
           .build());
```

只有属于activitiTeam的用户和所有者（当前登录的用户）才能看到此任务。

您可能已经注意到，您可以使用TaskPayloadBuilder来参数化将以一种流畅的方式发送到TaskRuntime的信息。

为了处理安全性，角色和组，我们依赖于Spring Security模块。 因为我们在Spring Boot应用程序内部，所以我们可以使用UserDetailsService来配置可用用户及其各自的组和角色。 我们目前正在@Configuration类中执行此操作：[https://github.com/Activiti/activiti-examples/blob/master/activiti-api-basic-task-example/src/main/java/org/activiti/examples/DemoApplicationConfiguration.java#L26](https://github.com/Activiti/activiti-examples/blob/master/activiti-api-basic-task-example/src/main/java/org/activiti/examples/DemoApplicationConfiguration.java#L26)

这里需要注意的重要一点是，为了以用户身份与TaskRuntime API进行交互，您需要具有以下角色：ACTIVITI_USER（授权机构：ROLE_ACTIVITI_USER）。

与REST端点进行交互时，授权机制将设置当前登录的用户，但是出于示例的原因，我们使用的实用程序类允许我们在上下文中设置手动选择的用户。 请注意，除非您正在尝试尝试并且想要在不经过REST端点的情况下更改用户，否则切勿这样做。 查看“网络”示例，以查看根本不需要该实用工具类的更多真实场景。

该示例中要强调的最后一件事是任务事件侦听器的注册：

```java
@Bean
public TaskRuntimeEventListener taskAssignedListener() {
  return taskAssigned
           -> logger.info(
                 ">>> Task Assigned: '"
                + taskAssigned.getEntity().getName()
                +"' We can send a notification to the assignee: "
                + taskAssigned.getEntity().getAssignee());
}
```

您可以根据需要注册任意多个TaskRuntimeEventListeners。 当服务触发运行时事件时，这将使您的应用程序得到通知。

#### ProcessRuntime API ####

以类似的方式，如果要开始使用ProcessRuntime API，则需要包括与以前相同的依赖项。 我们的目标是在将来提供更大的灵活性和单独的运行时，但是目前，相同的Spring Boot Starter同时提供TaskRuntime和ProcessRuntime API。

该部分中的代码可以在“ activiti-api-basic-process-example” maven模块内找到。

```java
public interface ProcessRuntime {
  ProcessRuntimeConfiguration configuration();
  ProcessDefinition processDefinition(String processDefinitionId);
  Page processDefinitions(Pageable pageable);
  Page processDefinitions(Pageable pageable,
              GetProcessDefinitionsPayload payload);
  ProcessInstance start(StartProcessPayload payload);
  Page processInstances(Pageable pageable);
  Page processInstances(Pageable pageable,
              GetProcessInstancesPayload payload);
  ProcessInstance processInstance(String processInstanceId);
  ProcessInstance suspend(SuspendProcessPayload payload);
  ProcessInstance resume(ResumeProcessPayload payload);
  ProcessInstance delete(DeleteProcessPayload payload);
  void signal(SignalPayload payload);
  ...
}
```

与TaskRuntime API相似，为了与ProcessRuntime API进行交互，当前登录的用户必须具有“ ACTIVITI_USER”角色。

首先，让我们自动连接ProcessRuntime：

```java
@Autowired
private ProcessRuntime processRuntime;

@Autowired
private SecurityUtil securityUtil;
```

与以前一样，我们需要SecurityUtil帮助器来定义与API交互的用户。

现在我们可以开始与ProcessRuntime进行交互了：

```java
Page processDefinitionPage = processRuntime
                                .processDefinitions(Pageable.of(0, 10));
logger.info("> Available Process definitions: " +
                  processDefinitionPage.getTotalItems());
for (ProcessDefinition pd : processDefinitionPage.getContent()) {
  logger.info("\t > Process definition: " + pd);
}
```

流程定义需要放在 /src/main/resources/processes/ 中。 对于此示例，我们定义了以下过程：

我们正在使用Spring Scheduling功能来每秒启动一个过程，从数组中拾取随机值以进行处理：

```java
@Scheduled(initialDelay = 1000, fixedDelay = 1000)
public void processText() {
  securityUtil.logInAs("system");
  String content = pickRandomString();
  SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yy HH:mm:ss");
  logger.info("> Processing content: " + content
                    + " at " + formatter.format(new Date()));
  ProcessInstance processInstance = processRuntime
                  .start(ProcessPayloadBuilder
                       .start()
                       .withProcessDefinitionKey("categorizeProcess")
                       .withProcessInstanceName("Processing Content: " + content)
                       .withVariable("content", content)
                       .build());
  logger.info(">>> Created Process Instance: " + processInstance);
}
```

与以前一样，我们使用ProcessPayloadBuilder来流畅地设置要启动哪个过程以及哪个过程变量的参数。

现在，如果我们回顾流程定义，您会发现3个服务任务。 为了提供这些服务任务的实现，您需要定义连接器：

```java
@Bean
public Connector processTextConnector() {
  return integrationContext -> {
      Map inBoundVariables = integrationContext.getInBoundVariables();
      String contentToProcess = (String) inBoundVariables.get("content")
     // Logic Here to decide if content is approved or not
     if (contentToProcess.contains("activiti")) {
        logger.info("> Approving content: " + contentToProcess);
        integrationContext.addOutBoundVariable("approved",true);
     } else {
        logger.info("> Discarding content: " + contentToProcess);
        integrationContext.addOutBoundVariable("approved",false);
     }
    return integrationContext;
  };
}
```

这些连接器使用Bean名称（在本示例中为“ processTextConnector”）自动连接到ProcessRuntime。 此bean名称是从我们的流程定义中的serviceTask元素的实现属性中选取的：

```xml
<bpmn:serviceTask id="Task_1ylvdew" name="Process Content" implementation="processTextConnector">
```

这个新的Connector接口是JavaDelegates的自然演变，新版本的Activiti Core将通过将它们包装在Connector实现中来尝试重用JavaDelagates：

```java
public interface Connector {
  IntegrationContext execute(IntegrationContext integrationContext);
}
```

连接器接收带有过程变量的IntegrationContext，并返回修改后的IntegrationContext，其结果需要映射回过程变量。

在前面的示例中，连接器实现正在接收“内容”变量，并基于内容处理逻辑添加“批准”变量。

在这些连接器内部，您可能会包括系统到系统的调用，例如REST调用和基于消息的交互。 这些交互趋于变得越来越复杂，由于这个原因，我们将在以后的教程中看到如何从ProcessRuntime上下文外部运行中提取这些连接器（云连接器），从而将外部交互的责任分离出来。 ProcessRuntime范围。

查看Maven模块activiti-api-spring-integration-example以获取更高级的示例，该示例使用Spring Integrations基于文件轮询器启动进程。

#### 全部实例 ####

您可以找到使用ProcessRuntime和TaskRuntime API来自动执行以下过程的示例：

该部分的代码可以在“ activiti-api-basic-full-example” maven模块内找到。

作为仅ProcessRuntime的示例，它还对一些输入内容进行了分类，但是在这种情况下，该过程依赖于人类演员来决定是否批准该内容。 与之前一样，我们有一个计划任务，该任务每5秒创建一个新的流程实例，并且模拟用户检查是否有可用的任务要处理。

将UserTask创建给一组潜在所有者，在本示例中为“ activitiTeam”组。 但是在这种情况下，我们不会像第一个示例那样手动创建任务。 每次启动流程时，流程实例都会为我们创建任务。

```xml
<bpmn:userTask id="Task_1ylvdew" name="Process Content">
  <bpmn:incoming>SequenceFlow_09xowo4</bpmn:incoming>
  <bpmn:outgoing>SequenceFlow_1jzbgkj</bpmn:outgoing>
  <bpmn:potentialOwner>
    <bpmn:resourceAssignmentExpression>
      <bpmn:formalExpression>activitiTeam</bpmn:formalExpression>
    </bpmn:resourceAssignmentExpression>
  </bpmn:potentialOwner>
</bpmn:userTask>
```

属于该组的用户将可以声明任务并继续工作。

我们鼓励您运行这些示例并进行实验，如果有疑问或发现问题，请与我们联系。

#### 总结 ####

在本教程中，我们已经了解了如何从新的Activiti Core Beta项目开始使用新的ProcessRuntime和TaskRuntime API。

我们建议您检查Activiti Examples存储库，以获取更多示例：[https://github.com/Activiti/activiti-examples](https://github.com/Activiti/activiti-examples)

帮助我们编写更多此类示例可能是社区做出的很好的初期贡献。 如果您有兴趣，请与我们联系，我们非常乐意为您提供指导。

即将有更多博客文章介绍Runtime Admin API，以及如何将这些示例修改为可以在我们的新Activiti Cloud方法中执行。

## 组件 ##

Activiti Cloud提供了一组基本的构建基块，可以将其分为3个独立的组：

* [Activiti Cloud Infrastructure]()

* [Activiti Cloud Applications]()

* [Activiti Cloud Modeler]()

我们定义了基础架构应提供的一组服务，这意味着在不同的环境中，这些组件可以由底层基础架构提供的服务代替。 其他组件将依靠这些基础设施服务来工作，这意味着必须提供一组清晰的功能，才能使Activiti Cloud应用程序与所有设计的功能一起使用。

Activiti Cloud应用程序是动态的，可以在运行时在现有基础架构之上进行配置。 可以使用我们提供的构件来构成Activiti Cloud应用程序，以适应各种情况，我们提供了一些有关如何构成这些应用程序的示例，但是Activiti Cloud设计支持不同的配置以支持大规模部署。

最后，Activiti Cloud Modeler提供了一个环境，您可以在其中生成业务资产。 这些将包括所有业务模型定义，例如业务流程，决策表，连接器定义（系统到系统交互的接口）等。

### Spring Cloud ###

从Activiti框架的角度来看，我们依赖于Spring Boot / Spring Cloud / Spring Cloud Kubernetes的3个关键方面，这些方面使我们的组件（例如Process Runtime）能够与其余基础架构很好地集成。

首先，我们所有的REST端点都依赖Spring Boot HATEOAS，以确保它们一致，并且需要维护的代码最少。 我们还为JSON有效负载提供了一种替代格式，以遵循Alfresco API准则。 Spring Boot是创建微服务的首选武器。 我们依靠Spring Boot提供经过测试的基本框架集，以创建REST端点，并通过依赖注入来引导我们的服务。 我们在很大程度上依赖于Spring Boot Starter概念，在这里我们使用@EnableAutoConfiguration来创建运行时所需的所有bean。 这些自动配置还帮助我们提供配置点，以参数化所提供服务的行为。

其次，我们非常依赖Spring Cloud Stream来确保我们可以异步方式使用和发出事件。 这是确保独立组件可以协作而不会紧密耦合的关键推动力。 Spring Cloud Streams支持Binders的概念，这些概念使我们可以使用底层Message Broker的不同实现，例如：RabbitMQ，Kafka，Amazon SQS等。

最后，我们利用一些服务（例如服务注册表，用于动态服务注册，发现和路由，分布式配置服务等的网关）之上的抽象层。为了利用Kubernetes内部的这些功能，我们使用了Spring Cloud Kubernetes项目。 在Spring Cloud发布培训中。 通过使用Spring Cloud Kubernetes，我们可以将我们的服务集成到Kubernetes本机服务中，从而实现服务发现，配置，秘密等功能。

我们利用所有组件来确保我们在Kubernetes内部使用Docker等技术在分布式环境中良好运行。 我们还确保Activiti Cloud不会与此类技术提供的任何功能重叠，以免引起任何摩擦。

### Activiti云基础架构 ###

Activiti Cloud旨在在Kubernetes内部运行。 这意味着Activiti Cloud将与Kubernetes本机服务（例如Kubernetes服务注册表，用于配置的配置映射，机密，作业/ CronJobs等）进行交互。

从基础架构的角度来看，我们将在Kubernetes内部部署以下服务集：

* **网关** : 从客户端（用户界面，其他应用程序）角度来看，公开单个入口点与所有服务进行交互是很常见的。 网关组件提供了该单一入口点，以访问所有已启动并正在运行的服务。 网关配置为注册默认情况下部署网关的名称空间中所有可用的服务，但是我们建议使用应用程序服务操作员将应用程序公开为逻辑单元。
* **身份管理/单点登录** : 为所有正在运行的服务提供单点登录功能，因此我们可以传播JWT以进行授权和身份验证。 该服务还提供了身份管理功能，使我们能够同步来自LPAD或AD服务器的用户。
* **分布式日志记录/跟踪/监视** : 

