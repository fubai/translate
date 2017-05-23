# 星际命名系统

ipns能给永久不可变的ipfs添加少量的可变性。它能在由peerID(ipfs节点公钥的哈希)组成的命名空间中为一个ipfs节点的哈希添加一个引用。设置的命令非常简单。

首先，要有待发布的内容：

```
$ echo 'Let\'s have some mutable fun!' | ipfs add
```

记下上面命令输出的哈希，然后把这个哈希发布到网络上：

```
$ ipfs name publish <that hash>
Published to <your peer ID>: <that hash>
```

现在，测试下是否对了，有几种不同的方式：

```
$ ipfs name resolve <your peer ID>
<that hash>
```

如果上面这行命令所运行的机器，和最前面两行命令所运行的机器是同一台，那应该会立即返回结果，因为第二条命令会在本地缓存ipns实体。可以到另一台运行着ipfs的机器上试下，看看效果。

另外一种方式是在网关上查看：

```
http://gateway.ipfs.io/ipns/<your peer ID>
```

还有一种方式：

```
ipfs cat /ipns/<your peer ID>
```

OK，有趣的东西来了：来做些改变

```
$ echo 'Look! Things have changed!' | ipfs add
```

下一步，拿着上面输出的哈希，然后...

```
$ ipfs name publish <the new hash>
Published to <your peer ID>: <the new hash>
```

瞧啊！现在，如果你重新解析那个ipns实体，就能看见新的对象了。

Congratulations！你刚刚成功的发布解析了一个ipns实体！现在，有几点要注意；第一，现在的每个ipfs节点都只能发布一个ipns实体。不过这很快就会改变。第二，修改一个ipns实体会"break links"，因为任何一个引用了ipns实体的东西可能不再指向它原本期望的内容。没办法解决这个问题(你懂的，可变性嘛)，因此，在需要确保永久性时，要谨慎的使用ipns链接。将来我们可能把ipns实体改成以git提交链的方式工作，链上连续的ipns实体会以时间倒序指向其它的值。

作者：[whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/ipns/readme.md)