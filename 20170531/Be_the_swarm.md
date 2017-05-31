## 玩玩网络

IPFS是关于网络！它有一组有用的命令用于观察网络。

看看你被直接连接到了谁：

```sh
ipfs swarm peers
```

获取整个网络的列表：

```sh
ipfs diag net
```

手动连接到指定的节点。如果下面示例的节点不能工作，可以试试`ipfs swarm peers`输出的节点。

```sh
ipfs swarm connect /ip4/104.236.176.52/tcp/4001/ipfs/QmSoLnSGccFuZQJzRadHn95W2CrSFmZuTdDWP8HXaHca9z
```

在网络上搜索一个给定的节点：

```sh
ipfs dht findpeer QmSoLnSGccFuZQJzRadHn95W2CrSFmZuTdDWP8HXaHca9z
```

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/network/readme.md)