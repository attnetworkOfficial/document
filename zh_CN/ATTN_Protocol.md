# ATTNProto - ATT节点协议

介绍ATT服务节点的协议规范以及通行细节。

## 概览

一个去中心化的消息传递网络。加密算法为信息的私密传递提供可靠支持；区块链技术为参与数据传输的节点提供代币奖励。

### 1. 介绍

目前的主流社交聊天服务，用户必须使用手机、邮箱等个人账户信息进行注册，且聊天的消息需要经过服务商提供的服务器，存在泄漏用户隐私的风险。ATT旨在建立一个去中心化的数据传网络，利用加密技术，用户只需持有私钥，就可以与其它用户协商通信密钥。在去中心化的消息网络中，完成一对一甚至一对多的加密消息传递。

### 2. 协议描述


> 高级接口组件

节点应该遵循一套基本的RPC接口规范，向其它节点提供服务。

有如下几种类型的消息

    1. RPC 调用 (客户端 -> 服务端)
    2. RPC 应答 (服务端 -> 客户端)
    3. 消息接收确认
    4. 消息状态查询

> ATTNProto 传输

节点之间传输的消息都通过特定的方法进行加密。

首先，节点间通过ATT秘钥协商机制建立会话。

> 底层传输协议

底层传输可以通过现有的多种协议实现:

    TCP
    Websocket
    HTTP
    UDP

使用相同私钥的节点加入网络时，这些节点均为有效节点。使用 UDP 协议时，节点收到的消息可能来自不同IP的有效节点。

> 总结

使用 OSI 网络7层协议作为类比:

    L7 (应用层): High-level RPC API
    L6 (表示层): 
    L5 (会话层): ATTNProto session
    L4 (传输层): ATTN protocol
    L3 (网络层): IP
    L2 (数据链路层): MAC/LLC
    L1 (物理层): IEEE 802.3, IEEE 802.11, etc…
