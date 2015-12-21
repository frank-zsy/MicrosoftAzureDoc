# API设计指南

该指南中的某些部分依然存在争议并可能修改，欢迎大家的反馈。

## 概述

许多基于网络的现代解决方案都是用Web服务器提供的功能为远程客户端应用提供服务。由Web服务暴露的操作组成了[API][APIWiki]接口。该指南主要描述一些在设计API时应该考虑的问题。

人们设计实现了许多不同的实现机制、数据格式、传输协议等，都旨在将实现Web服务器的技术和使用它们的客户端程序抽离开来，这些都是为了降低服务器与客户端之间的依赖性。其中，如[SOAP][SOAPWiki]等协议功能非常强大，但在一些常用场景下显得较为庞大笨重。目前最典型的API服务器均基于[HTTP协议][HTTPWiki]（HTTP也是许多SOAP底层实现使用的传输协议）。

使用SOAP的一个问题就是由Web服务器暴露出来的是一组设计者认为有用的接口，但这些接口可能并不能满足所有需要。例如，一个电商系统可能提供了一组典型的面向RPC的CRUD操作，如用于创建订单的CreateOrder、用于利用ID获取订单的GetOrder、用于删除订单的DeleteOrder、用于修改订单内容的ModifyOrder等。但如果想要获取所有的订单便需要增加如GetAllOrders这样的接口，或获取某个特定用户所有订单需要增加如GetOrdersForCustomer接口并传入用户ID，又或应用需要找出某商品相关的所有订单，则可能需要提供一个GetOrdersForProduct接口。这些接口均面向某特定的业务场景，且在业务需求变化时并不易扩展。

## [REST][RESTWiki]简介

Roy Fielding在其2000年的论文中提出了另一种构建Web服务器接口的架构方式——REST。REST是构建基于超媒体分布式系统的软件架构风格。使用REST的主要优势是其基于开放的标准且其实现并不与客户端或任何特定的实现技术相关。例如，一个REST服务器可能使用Microsoft ASP.NET实现API，而客户端则可以使用其他任何具有HTTP请求和解析工具的语言来实现。

REST模式使用导航方案在网络上表示业务对象和服务（即资源），并使用经典的HTTP协议传输访问资源的请求。客户端程序向一个特定的资源URI提交请求，并由HTTP动词（通常是GET、POST、PUT或DELETE）来指定对此资源要进行的操作，而HTTP的body部分则包含了要进行该操作所需的数据。需要重点理解的是REST提供的是无状态请求模型。HTTP请求应该是完全独立并可能以任意顺序发出，故在请求间尝试保持状态信息是不可行的。保存信息的唯一位置就是资源本身，且每一个请求都应该是[原子操作][AtomicOperationWiki]。事实上，REST模式实现了一个有限状态机，一个请求会将某资源从一个定义好的非暂时状态转换到另一个状态。

> **注意**：REST模式中请求的无状态性使得系统在遵循这些原则时可以获得高可扩展性。因为发起一系列请求的客户端应用无需与处理请求的服务器保持任何耦合关系。
