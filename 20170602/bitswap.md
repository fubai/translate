bitswap
=======

![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

Authors:

- David Dias
- Jeromy Johnson
- Juan Benet

* * *

# 抽象

Bitswap是ipfs的数据交易模型，它管理着IPFS网络上节点之间的块请求与发送。Bitswap有两个主要的工作，第一个是为网络上的客户端获取其所请求的块，第二个是审慎地把拥有的块发送给其它想要这些块的节点。


# 介绍

Bitswap是IPFS主要的块交换协议。它处理着IPFS用户，人类或一个应用程序产生的请求，为其从网络上获取数据块。它和其它IPFS节点上的Bitswap代理交互，交换(获取 + 提供)其所需要的块。

Bitswap是一个基于消息的协议，而不是响应-应答。消息里会包含需要列表(wantlist)或块。在收到wantlist时，IPFS节点应该考虑发送被要的块，如果它有的话。在收到块时，节点应该发出一个叫做“Cancel”的通知，以表明它不再需要该块了。在协议层，bitswap是非常简单的。

总之Bitswap是一个简单的协议，然而要让它运行的块，占用的内存少的话，有几个非常重要的实现细节必须弄对。我们把这些细节写在这个文档里，这样其它实现者就能学习这些从bitswap的几轮迭代里收集起来的知识了。

# Bitswap流

Bitswap里有两个主要的流程：向其它节点请求块和为其它节点提供块。

## 请求块

客户端要请求的新块是被“需要管理器”处理的，它会为每个被要的新块(或一组块)调用“WantBlocks”方法。“需要管理器”给每个连接节点的消息队列发新消息以确保这些节点知道了我们需要哪些新块。消息队列在有工作可做时会循环执行如下工作：

1. 确保自己和自己的节点有连接
2. 接收要待发送的消息
3. 发送

循环在执行到第1步或第3步时，如果有新消息被添加了，这些消息就会被关联成一个，这是为了避免要去维护一个真的队列，最后发送这些消息。客户端接收块并为它发送“Cancel”消息时也是同样的过程。

## 提供块

在内部，当我们收到一个有wantlist的消息时，它会被发送给决策引擎以被决策，我们有的wantlist里的块会被放到“节点请求队列”里。任何我们处理地被其它节点需要的块都会有一个任务在“节点请求队列”里被创建。

“节点请求队列”是一个优先级队列，会通过一些度量指标把可用的任务排序，现在这个指标非常简单，旨在解决每一个其它节点的任务。以后会实现更高级的决策逻辑。

任务工作者把要做的任务从队列里拉取下来，检索要发送的块，然后发送出去。任务工作者的数量受一个常量因子限制。

# 传输格式

["bitswap消息"之protobuf](https://github.com/ipfs/go-ipfs/blob/master/exchange/bitswap/message/pb/message.proto)的流:

```
message Message {
  message Wantlist {
    message Entry {
      optional string block = 1; // 块的键
      optional int32 priority = 2; // 优先级(normalized)。默认是1
      optional bool cancel = 3;  // 是否这撤销了一个实体
    }

    repeated Entry entries = 1; // wantlist实体集合
    optional bool full = 2;     // 是否是完整的wantlist。默认是false
  }

  optional Wantlist wantlist = 1;
  repeated bytes blocks = 2;
}
```

# 实现细节

另外，请务必阅读 - https://github.com/ipfs/go-ipfs/tree/master/exchange/bitswap#go-ipfs-implementation

实现建议:

- 维护一组“在线伙伴”节点
- 协议监听者接受来自伙伴节点的流以接收消息
- 协议发送者打开去向伙伴节点的流以发送消息
- 分离出一个决策引擎是为了选择把哪些块发送给哪些伙伴节点，以及在什么时候发。(这有点小复杂，但分离了这个引擎之后，整体变得超简单了)

发送者：

1. 打开bitswap流
2. 发送一条或多条bitswap消息
3. 关闭bitswap流

监听者：

1. 接受一个bitswap流
2. 接受一个或多个bitswap消息
3. 关闭bitswap流

事件：

bitswap.addedBlock(block)
- 看看是否有节点要这个块，然后发给它

bitswap.getBlock(key, cb)
- 添加到wantlist
- 也许发送wantlist的更新给节点

bitswap.cancelGet(key)
- 可以发送对wantlist的cancel了

bitswap.receivedMessage(msg)
- 处理wantlist的变化
- 处理块

bitswap.peerConnected(peer)
- 把节点添加到节点集合 + (也许)给节点集合发送wantlist

bitswap.peerDisconnected(peer)
- 把节点从节点集合中移除

小问题:
- bitswap的客户端可能调用"getBlock"后再调用"cancelBlock"
- 伙伴节点可能会像发送垃圾邮件一样发送wantlist
- 仅每个节点规范优先级

模块:
- bitswap-decision-engine
- bitswap-message
- bitswap-net
- bitswap-wantlist

注意：

```
var bs = new BlockService(repo, bitswap)
bs.getBlock(multihash, (err, block) => {
  // 1) 尝试从本地仓库获取
  // 2) 如果没有 -> 问bitswap要
    // 2.1) bitswap在收到块后调用cb()，成功后把块写道本地仓库里repo as well. 
})
```

# 实现

- [Go](https://github.com/ipfs/go-ipfs/tree/master/exchange/bitswap)