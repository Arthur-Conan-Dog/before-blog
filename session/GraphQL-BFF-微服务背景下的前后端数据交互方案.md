### [GraphQL-BFF：微服务背景下的前后端数据交互方案](https://mp.weixin.qq.com/s/oZsF4cHdInJXJ3S2Ov9g3A)

#### 关于 GraphQL 容易误解的几点

1. GraphQL 不一定要操作数据库

GraphQL 只是关于 schema 和 resolver 的一一对应和调用，它并未对数据的获取方式和来源等做任何假设。

2. GraphQL 与RESTful API并不对立

可以在 GraphQL Schema 的 Resolver 函数里，调用 RESTful API 去获取数据。当然，也可以调用 RPC 或者 ORM 等方式，从别的数据接口或者数据库里获取数据。因此，实现一个 GraphQL 服务，并不需要挑战当前整个后端体系。它具有高度灵活的适配能力，可以低侵入性的嵌入当前系统中。

3. GraphQL 不一定是一个后端服务

GraphQL 作为一门语言，它并没有限制在后端场景，甚至可以用在纯前端，去实现 State Management 状态管理。Relay 等框架，即包含了用在前端的 graphql。

4. GraphQL 不一定需要 Schema

GraphQL Type System 是一个静态的类型系统。我们可以称之为静态类型 GraphQL。此外，社区还有一种[动态类型的 GraphQL 实践](https://github.com/apollographql/apollo-client/tree/master/packages/graphql-anywhere)。

在某些场景下，动态类型的 GraphQL 有一定的便利性。不过，它同时丧失了 GraphQL 的部分精髓。不管是静态类型的 GraphQL 还是动态类型的 GraphQL，都是既可以运行在服务端，也可以运行在前端。

5. GraphQL 不一定返回 JSON 数据格式

graphql-anywhere 里的例子。实现了一个 gqlToReact 的 Resolver，可以把一个 graphql 查询转换为 ReactElement 结构。不过这种做法，目前仅仅停留在能力演示阶段。其妙用还有待社区去挖掘和探索。

#### GraphQL 的几种使用模式

1. RESTful-like

把 RESTful API 服务，替换成 GraphQL 实现。之前有多少 RESTful 服务，重构后就有多少 GraphQL 服务。之前发两个请求，现在还是需要发两个请求，并没有发挥组合资源在一个 query 的作用。

2. GraphQL as an API Gateway 模式

在 GraphQL 这个中间层里，我们将各个微服务，按照它们的数据关联，整合成一个基于 GraphQL Schema 的数据关系网络。前端通过 GraphQL 查询语句，同时发起对多个微服务的数据的获取、筛选、裁剪等行为。

2.1 转发数据请求，聚合数据结果的 GraphQL => GraphQL as BFF

2.2 包含大量真实的数据操作和处理的 GraphQL => 一个疑问：BFF里也可能有数据操作呀？

3. GraphQL as a Backend Framework

前端的调用方式，还是 RESTful API，在 RESTful 服务内部，它自己向自己发起了 GraphQL 查询。

适用：PC 端和移动端对于同一份数据的消费方式差异很大，需要两个RESTful API，因此很多公司会分出两个BFF，分别给两个平台使用。可以把 PC-BFF 和 Mobile-BFF 整合成一个 GraphQL-BFF 服务。前端与BFF间可能还通过RESTful交互，但BFF内部可以通过 GraphQL 查询自己，省去了分两个 service 的开销。支持优雅降级。

#### 关于架构设计

Apollo-GraphQL 的重心是前文所说的第一类 API Gateway 角色的 GraphQL 服务。

GraphQL-BFF 的开发模式，应该跟 service 的领域模型，有一一对应的关系。然后通过某种形式，多个 services 自然整合到一起。每个 GraphQL-Service 应该可以按照模块化的方式编写，跟其它 GraphQL-Service 组合起来后，构建出更大的 GraphQL-Server。

resolver 也需要解耦。可以利用如 koa 框架的中间件插件机制。

需要解决 mock 问题。文章里用到了 faker 这个 npm 包。
