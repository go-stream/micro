微服务的设计指南：

	1、职责单一原则（Single Responsibility Principle）：把某一个微服务的功能聚焦在特定业务或者有限的范围内会有助于敏捷开发和服务的发布。

	2、设计阶段就需要把业务范围进行界定。

	3、需要关心微服务的业务范围，而不是服务的数量和规模尽量小。数量和规模需要依照业务功能而定。

	4、于SOA不同，某个微服务的功能、操作和消息协议尽量简单。

	5、项目初期把服务的范围制定相对宽泛，随着深入，进一步重构服务，细分微服务是个很好的做法。

	

微服务消息

	同步消息 – REST, Thrift 

		REST是微服务中默认的同步消息方式，它提供了基于HTTP协议和资源API风格的简单消息格式，多数微服务都采用这种方式（每个功能代表了一个资源和对应的操作）。

	异步消息 – AMQP, STOMP, MQTT

	消息格式 – JSON, XML, Thrift, ProtoBuf, Avro 	

		微服务采用简单的文本协议JSON和XML，基于HTTP的资源API风格。如果需要二进制，通过用到Thrift, ProtoBuf, Avro。

	服务约定 – 定义接口 – Swagger, RAML, Thrift IDL

		

微服务集成 \(服务间通信\)

	点对点方式 – 直接调用服务

	API-网关方式

		所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能个。通常，网关也是提供REST/HTTP的访问API。服务端通过API-GW注册和管理服务。

		1、有能力为微服务接口提供网关层次的抽象。比如：微服务的接口可以各种各样，在网关层，可以对外暴露统一的规范接口。

		2、轻量的消息路由、格式转换。

		3、统一控制安全、监控、限流等非业务功能。	4、每个微服务会变得更加轻量，非业务功能个都在网关层统一处理，微服务只需要关注业务逻辑

		目前，API网关方式应该是微服务架构中应用最广泛的设计模式。

	

	消息代理方式

		异步的场景下，通过队列和订阅主题，实现消息的发布和订阅。

		通常异步的生产者/消费者模式，通过AMQP、MQTT等异步消息规范。

	

数据的去中心化	

	微服务方式，多个服务之间的设计相互独立，数据也应该相互独立（比如，某个微服务的数据库结构定义方式改变，可能会中断其它服务）。因此，每个微服务都应该有自己的数据库。	其它微服务不能直接访问。

	1、每个微服务有自己私有的数据库持久化业务数据

	2、每个微服务只能访问自己的数据库，而不能访问其它服务的数据库	3、某些业务场景下，需要在一个事务中更新多个数据库。这种情况也不能直接访问其它微服务的数据库，而是通过对于微服务进行操作。

		

治理（构建方案）去中心化

	在微服务架构中，不同的微服务之间相互独立，并且基于不同的平台和技术。因此，没有必要为服务的设计和开发定义一个通用的标准。

	总结微服务的治理去中心化如下：

	1、微服务架构，在设计时不需要集中考虑治理。

	2、每个微服务可以有独立的设计、执行决策。

	3、微服务架构着重培养通用/可重用的服务。

	4、运行时的治理，比如安全级别保证（SLA），限制，监控，安全和服务发现，可以在API网关层处理。	

		

服务注册和发现

	服务注册

		微服务在启动时向注册中心注册自己的信息，关闭时注销。其它使用者能够通过注册中心找到可用的微服务和相关信息。

	服务发现	

		客户端发现 

			客户端或者API网关通过查询服务注册中心或者服务的位置信息。

			客户端/API网关必须调用服务注册中心组件，实现服务发现的逻辑。

		服务端发现 	

			客户端/API网关把请求发送到已知位置信息的组件（比如负载均衡器）。

			组件去访问注册中心，找到微服务的位置信息。	

		

部署

	1、能够独立于其他微服务发布或者取消发布

	2、微服务可以水平扩展（某一个服务比其他的请求量大）

	3、快速构建和发布

	4、微服务之间的功能不相互影响

	5、Docker能够快速部署微服务，包括关键几点：

	（一个运行在linux上并且开源的应用，能够协助开发和运维把应用运行在容器中）

		把微服务打包成Docker镜像

		启动容器实例

		改变实例的数量达到扩容需求

	相对于传统的虚拟机模式，利用docker容器构建、发布、启动微服务将会变得十分快捷。

	通过Kubernetes能够进一步扩展Docker的能力，能够从单个linux主机扩展到linux集群，

	支持多主机，管理容器位置，服务发现，多实例。都是微服务需求的重要特性。

	因此，利用Kubernetes管理微服务和容器的发布，是一个非常有力的方案。	

		

安全

	OAuth2-是一个访问委托协议。

		需要获得权限的客户端，向授权服务申请一个访问令牌。

		访问令牌没有任何用户/客户端的信息，仅仅是一个给授权服务器使用的用户引用信息。

		因此，这个“引用的令牌”也没有安全问题。

	OpenID类似于OAuth，

		不过除了访问令牌以外，授权服务器还会颁发一个ID令牌，包含用户信息。

		通常由授权服务器以JWT（JSON Web Token）的方式实现。

		通过这种方式确保客户和服务器端的互信。

		JWT令牌是一种“有内容的令牌”，包含用户的身份信息，在公共环境中使用不安全。	

	现微服务安全的关键几步：

		1、所有的授权由授权服务器，通过OAuth和OpenID方式实现，确保用户能访问到正确的数据。

		2、采用API网关方式，所有的客户端请求有唯一入口。

		3、客户端通过授权服务器获得访问令牌，把令牌发送到API网关。

		4、令牌在网关的处理 – API网关得到令牌后，发送到授权服务器获得JWT。

		5、网关把JWT和请求一起发送到微服务中。

		JWT包含必要的用户信息，如果每个微服务都能够解析JWT，那么你的系统中每个服务都能处理身份相关的业务。在每个微服务中，可以有一个处理JWT的轻量级的组件。	



	（https://mp.weixin.qq.com/s/x0CZpovseOuofTA\_lw0HvA）

	正如 David Borsos 所建议的一种方案，在微服务架构下，我们更倾向于将 Oauth 和 JWT 结合使用，Oauth 一般用于第三方接入的场景，管理对外的权限，所以比较适合和 API 网关结合，针对于外部的访问进行鉴权（当然，底层 Token 标准采用 JWT 也是可以的）。JWT 更加轻巧，在微服务之间进行访问鉴权已然足够，并且可以避免在流转过程中和身份认证服务打交道。

	（https://segmentfault.com/a/1190000006785852）

	

事务

	跨多个微服务的分布式事务支持非常复杂，

	微服务的设计思路是尽量避免多个服务之间的事务操作。

容错设计		

线路中断		

	

http://blog.nsfocus.net/token-authentication/



