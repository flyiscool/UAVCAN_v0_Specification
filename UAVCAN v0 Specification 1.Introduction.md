## UAVCAN v0 Specification 1.Introduction

------------
## Contents
### Introduction
 - Core design goals
 - Specification update and approval process
 - Referenced sources
------------------------

## Introduction（ 简介）
This section covers the basic concepts that govern development and maintenance of the specification.
本节介绍了在开发和维护中规范说明的一些基本概念。

The actual specification is contained in the following sections.
实际的规范说明在后续章节中。

The reader should have a solid understanding of the main concepts and operating principles of the CAN bus.
读者在阅读之前应该对 CAN 总线的主要概念和工作原理有个扎实的理解。



### Core design goals（核心设计目标）
The core design goals listed below help explain the basic UAVCAN concepts and the motivation behind them.
下面列出了核心设计目标帮助解释 UAVCAN 的基本概念和背后的动机。

 - __Democratic network__ - There should be no master node. All nodes in the network should have the same communication rights so that there is no single point of failure.
 - __去中心化网络__ - 网络中没有主节点。所有的节点享有同样的通信权限避免出现单个节点引起的通信失败。
 <br>
 
 - __Nodes can exchange long payloads__ - Nodes must be provided with a simple way to exchange large data structures that cannot fit into a single CAN frame (such as GNSS solutions, 3D vectors, etc.). UAVCAN should perform automatic transfer decomposition and reassembly at the protocol level, hiding the related complexity from the application.
 - __节点间可以互传长载荷__ -  对于超过一个 CAN 帧长度的大型数据结构（如 GNSS 解决方案，3D 向量等），节点必须提供一个简单的方式来进行互传。UAVCAN 应该在协议层封装完成对大数据结构的传输分解和重组过程，以此避免应用程序出现相关的复杂操作。
<br>

 - __Support for redundant interfaces and redundant nodes__ - This is a common requirement for safety-concerned applications.
 - __支持冗余接口和冗余节点__ - 这是一个安全相关应用的一般需求。
<br>

 - __High throughput, low latency communication__ - Applications that are dependent on high-frequency, hard real-time control loops, require a low-latency, high-throughput communication method.
 - __高吞吐量，低延时通信__ - 依赖于高频、硬实时控制循环的程序，需要低延时、高吞吐量的通信方法。
 <br>
 
 - __Simple logic, low computational requirements__ - UAVCAN targets a wide variety of embedded systems, from high-performance embedded on-board computers for intensive data processing (e.g., a high-performance Linux-powered machine) to extremely resource-constrained microcontrollers. The latter imposes severe restrictions on the amount of logic needed to implement the protocol.
 - __逻辑简单，计算量小__ - UAVCAN 的目标应用是各种各样的嵌入式系统，从用于密集数据处理的高性能嵌入式车载计算机(例如，高性能的 linux 驱动的机器)到资源极其有限的微控制器。后者对实现协议所需的逻辑数量有着严格的限制。
 <br>

 - __Common high-level functions should be clearly defined__ - UAVCAN defines standard services and messages for common high-level functions, such as network discovery, node configuration, node firmware update, node status monitoring (which naturally grows into a vehicle-wide health monitoring), network-wide time synchronization, dynamic node ID allocation (a.k.a. plug-and-play), etc.
 - __常用的高级功能应该被清晰定义__ - UAVCAN 为常见的高级功能定义了标准服务和消息，如网络发现、节点配置、节点固件更新、节点状态监控（或者扩展成为整机的状态监控）、全网时间同步，动态节点 ID 分配（即插即用）等。
<br>

 - __Open specification and reference implementations__ - The UAVCAN specification is open and freely available; the reference implementations are distributed under the terms of the MIT License.
 - __开源的规范和参考实现__ - UAVCAN 规范是开源而且免费的；参考实现的发布遵循“MIT License”。
<br>

### Specification update and approval process（规范的更新和审批流程）
The UAVCAN development team is charged with advancing the specification based on input from adopters. This feedback is gathered via the mailing list, which is open to everyone.

UAVCAN 的开发团队负责根据使用者的输入来推进规范。反馈通过邮件列表收集，邮件列表对所有人开放。

The set of standard data definitions is one of the cornerstone concepts of the specification (see data structure description language (DSDL)). Within the same major version, the specification can be extended only in the following ways:

标准数据集的定义是规范的基础概念之一（请参考数据结构描述语言（DSDL）)。在同一个主版本内，规范只能以一下方式新型扩展：

 - A new data type can be added, possibly with default data type ID, as long as the default data type ID doesn’t conflict with one of the existing data types.
  - 可以添加一个新的数据类型，可能使用默认数据类型 ID，只要默认数据类型ID和已有数据类型不冲突。
<br/>

 - An existing data type can be modified, as long as the modification doesn’t break backward compatibility.
 - 只要不破坏向后兼容性，就可以修改现有的数据类型。
<br/>

 - An existing data type can be declared deprecated.
 - 已有数据类型可以被声明放弃使用。
 <br/>
 
  - Once declared deprecated, the data type will be maintained for at least two more years. After this period its default data type ID may be reused for an incompatible data type.
  - 一旦宣布放弃使用，已有数据类型会被保留2年以上。这个时间过后他默认的数据类型ID可以被用于不兼容的数据类型。
 <br/>
 
  - Deprecation will be announced via the mailing list, and indicated in the form of a comment within the DSDL definition.
  - 弃用将通过邮件列表的形式宣布，并在DSDL的定义中以注释的形式表示。
  <br/>
 
Link to the repository containing the set of default DSDL definitions can be found on the contacts page.
可以在联系人页面上找到包含缺省 DSDL 定义集的存储库链接。


### Referenced sources（参考资源）
The UAVCAN specification contains references to the following sources:
UAVCAN规范包含对以下来源的引用:

 - CiA 801 - Application note - Automatic bit rate detection.
 - CiA 103 - Intrinsically safe capable physical layer.
 - CiA 303 - Recommendation - Part 1: Cabling and connector pin assignment.
 - IEEE 754 - Standard for binary floating-point arithmetic.
 - ISO 11898-1 - Controller area network (CAN) - Part 1: Data link layer and physical signaling.
 - ISO 11898-2 - Controller area network (CAN) - Part 2: High-speed medium access unit.
 - ISO/IEC 10646:2014 - Universal Coded Character Set (UCS).
 - ISO/IEC 14882:2014(E) - Programming Language C++.
 - “Implementing a Distributed High-Resolution Real-Time Clock using the CAN-Bus”, M. Gergeleit and H. Streich.
 - “In Search of an Understandable Consensus Algorithm (Extended Version)”, Diego Ongaro and John Ousterhout.
