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