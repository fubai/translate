## 固定

固定是IPFS里的一个非常重要的概念。IPFS在语义上试图让用户感觉每一个对象都是本地的，也就是没有“从远程服务器上获取这个文件给我”这种语义，而只是`ipfs cat`或`ipfs get`这种不管对象定位在哪都表现的一样的方式。尽管这很nice，但有时候还是需要能控制周围所保留的东西。固定是一种机制，它能告诉IPFS始终把一个给定的对象保留在本地。IPFS的缓存机制相当激进，被执行了任何一个IPFS操作后的对象，只会被缓存一小段时间，这些对象可能会被定期的垃圾回收处理掉。要阻止垃圾回收，简单的pin你所关心对象的哈希就行了。而使用`ipfs add`添加的对象默认就会被递归的固定在本地。

```
echo "ipfs rocks" > foo
ipfs add foo
ipfs pin ls --type=all
ipfs pin rm <foo hash>
ipfs pin rm -r <foo hash>
ipfs pin ls --type=all
```

你可能已经注意到了，第一个`ipfs pin rm`命令没有工作，它应该输出了`Error: <foo hash> is pinned recursively`(译者调用`ipfs pin rm`时并没有出现作者所说的情况，调用`ipfs pin rm --help`发现参数r现在的默认值为true)。IPFS的世界里有三种类型的固定：直接固定，只固定一个块，不固定其它和这个块有关的块；递归固定，固定给定的块，以及它所有的子块；间接固定，一个给定块的父块被递归的固定了，导致了该块被间接的固定。

一个被固定的对象不会被垃圾回收，如果你不信可以试下：

```
ipfs add foo
ipfs repo gc
ipfs cat <foo hash>
```

但是如果foo被以某种方式取消固定后...

```
ipfs pin rm -r <foo hash>
ipfs repo gc
ipfs cat <foo hash>
```

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/pinning/readme.md)