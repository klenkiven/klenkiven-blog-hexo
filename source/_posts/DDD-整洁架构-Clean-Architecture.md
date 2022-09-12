---
title: DDD - 整洁架构/Clean Architecture
sidebar:
  - toc
date: 2022-09-12 22:51:13
tags: [DDD,架构设计,设计模式]
categories: [领域驱动设计]
cover: https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/CA-COVER.jpg
---

## 概述

整洁架构（Clean Architecture）又称为 洋葱架构（Onion Architecture），可能是名字不咋好听，外网一般叫做整洁架构（Clean Architecture）。

洋葱架构将业务逻辑和应用程序模型放在应用程序的核心。与以前将业务逻辑强依赖于数据访问和或者其他基础设施不同，这个架构依赖是**反转**的，也就是说：基础设置和细节的实现依赖于应用程序核心（Application Core）。这样，功能就通过在应用程序核心中定义抽象或者接口来被实现，同时这些抽象将会被定义在基础设施层（Infrustructure  Layer）的类型所实现。一种常见的可视化方法是使用一系列同心圆来表示这个架构，类似于洋葱。

![img](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/CA-01.png)

在这个图表中，依赖流都流向最中心的圆圈。**应用程序核心**（The Application Core）就是这个最中心的圆圈的名字。并且你可以看到，应用程序核心并没有依赖其他的应用程序层。应用程序的**实体和接口**在最中心，在他们外面仍然是应用程序核心，是**领域服务**，领域服务通常实现了最中心的实体和接口。应用程序核心外层，就是UI和基础设置层，他们依赖于应用程序核心，而**不是互相之间依赖**（必然的）。

![img](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/CA-02.png)

上面实现箭头代表了编译时的依赖，虚线箭头代表了运行时依赖。在洋葱架构中，UI层工作在应用程序核心接口定义的接口上，并且在**理想状态下UI层不应该知道任何在基础设施层实现的具体类型**。然而，这些实现类型是应用程序执行所必需的，因此它们需要存在并通过**依赖注入**连接到应用程序核心接口。（**依赖倒转原则**）

在实际上使用ASP.NET Core框架的时候，根据上面的推荐要求，可以设计为下图的形式。

![img](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/CA-03.png)

因为应用程序核心不依赖于任何的基础设施，所以非常容易为这层编写自动化测试。

![img](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/CA-04.png)

![img](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/CA-05.png)

同时UI层不包含任何的对于基础设施层的实现，这样就非让容易去替换各种各样的实现，也有助于去测试基础设施或者为了响应需求去改变应用程序的需求。ASP.NET Core自带的依赖注入让这个架构非常适合去构建一个可商用的单机应用程序。

对于单机应用，应用程序核心、基础设施和UI项目全部都是作为一个应用运行。运行时程序架构看起来就比较像下图。

![img](https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/CA-06.png)

## 在整洁架构中组织代码

在整洁架构的解决方案中，每个项目都有清晰的责任。就比如像这样，在每个项目中都有特定的类型，这样你就可以在适当的位置找到与这些类型对应的文件目录项目。

### 应用程序核心（Application Core）

在应用程序核心中有业务模型，包括实体（Entities）、服务（Servises）和接口（Interfaces）。这些接口包括操作的抽象，然后这些抽象将会在基础设施中表现为，数据访问、文件访问、网络调用等等。有时候，定义在这层的服务或者接口会需要**和UI和基础设施无依赖关系**的**非实体类型**。这些可以被定义为简单数据传输对象（DTOs）。

#### 应用程序核心类型

- Entities (需要持久化存储的实体)
- Aggregates (聚合根，实体的组合)
- Interfaces
- Domain Services
- Specifications（这个比较特殊来自于作者的框架工具）
- Custom Exceptions and Guard Clauses（这个比较特殊来自作者的框架工具）
- Domain Events and Handler

### 基础设施（Infrustructure）

基础设施项目通常包括数据访问的实现。在一个经典的ASP.NET Core Web应用程序中，这些实现包括EF的DbContext，以及所有的EF Core的 Migration 定义的对象，以及数据访问实现类。最通用的方式就是通过 [Repository 设计模式](https://deviq.com/repository-pattern/) 来抽象数据访问的实现代码。

除了数据访问实现，基础设施项目也应该包含**涉及和基础设施交互的服务**的实现。这些服务应该实现定义在应用程序核心中的接口，并且这些基础设施应该有一个应用程序核心项目的引用。

#### 基础设施的类型

- EF Core types (DbContext, Migration)
- Data access implementation types (Repositories)
- Infrastructure-specific services (for example, FileLogger or SmtpNotifier)

### UI层

ASP.NET Core MVC 程序的用户接口层是这个应用程序的**入口**。这个项目因该引用应用程序核心项目，并且**它的类型应该严格地通过应用程序核心来和基础设施做交互**。基础设施项目中，**不需要实例化的静态调用**允许被UI层调用。

#### UI层类型

- Controllers
- Custom Filters
- Custom Middleware
- Views
- ViewModels
- Startup

Startup 类或者 Program.cs 文件负责配置应用程序，并且依赖注入实现类型到接口中。执行此逻辑的位置被称为应用程序的**组合根（Composition Root）**，这个位置可以进行依赖项注入，以便于在运行时正常工作。

{% grid 注意 %}
为了在应用程序启动期间连接依赖项注入，UI层项目可能需要引用基础设施项目。可以通过使用自定义DI容器消除这种依赖性，该容器内置了对从程序集加载类型的支持。就本示例而言，最简单的方法是允许UI项目引用基础设施项目（**但开发人员应将对基础设施项目中类型的实际引用限制为应用程序的组合根**）。
{% endgrid %}

## 参考资料

- [视频资料] [SOLID Principles for C# Developers](https://app.pluralsight.com/library/courses/csharp-solid-principles/table-of-contents)
- [视频资料] [Domain-Driven Design Fundamentals](https://app.pluralsight.com/library/courses/fundamentals-domain-driven-design/table-of-contents)
- [书籍] [Architecing Modern Web Applications with ASP.NET Core And Microsoft Azure](https://aka.ms/webappebook)
- [开源项目] [ardalis/CleanArchitecture](https://github.com/ardalis/CleanArchitecture)
- [开源项目] [dotnet-architecture/eShopOnWeb](https://github.com/dotnet-architecture/eShopOnWeb)
