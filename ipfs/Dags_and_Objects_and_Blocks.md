## 处理块

命令`ipfs add`将为指定的文件数据创建出一个Merkle DAG，创建时遵循了[unixfs数据格式](https://github.com/ipfs/go-ipfs/blob/master/unixfs/pb/unixfs.proto)。这意味着文件会被打碎成多个块，然后用“链接节点”把它们联系在一起，形成一个类似树的结构。给定文件的“哈希”实际上就是这个DAG根(最上面)节点的哈希。对于一个给定的DAG，用`ipfs ls`就可以简单的查看其子块(经译者测试，`ipfs ls`已经不具有本文所写的功能，而本文想以`ipfs ls`体现的功能，目前实际是由`ipfs object links`所具有的)。

比如：

```
# 确保这个文件是一个大于256k的文件
ipfs add 一个大文件 
ipfs ls 其哈希
```

上面的命令会打印出如下内容：

```
> ipfs ls qms2hjwx8qejwm4nmwu7ze6ndam2sfums3x6idwz5myzbn
qmv8ndh7ageh9b24zngaextmuhj7aiuw3scc8hkczvjkww 7866189 
qmuvjja4s4cgyqyppozttssquvgcv2n2v8mae3gnkrxmol 7866189 
qmrgjmlhlddhvxuieveuuwkeci4ygx8z7ujunikzpfzjuk 7866189 
qmrolalcquyo5vu5v8bvqmgjcpzow16wukq3s3vrll2tdk 7866189 
qmwk51jygpchgwr3srdnmhyerheqd22qw3vvyamb3emhuw 5244129
```

这会展示出文件所有的直接子块，同样还有磁盘上子块的大小以及子块的子块。

### 用块做什么?

如果你感到新奇，就能从这些不同的块中获取到大量不同的信息。你可以用子块的哈希做为`ipfs cat`的输入，只查看其对应子树的数据(子块以及子块的子块的数据)。要只查看给定块的数据，而不包含其子的话，可以使用`ipfs block get`。但请注意，把`ipfs block get`用在一个中间块上，会在你的屏幕上打印出这个中间块的DAG结构的原生二进制数据。

`ipfs block stat`会告诉你给定块的准确大小(不包含其子)，`ipfs refs`会告诉你指定块所有的子。和`ipfs refs`类似，`ipfs ls`或者`ipfs object links`会给你展示所有的子以及子的大小。`ipfs refs`只返回子块的哈希，所以是一个更适合在给定对象的每个子块上运行脚本的命令。

### 块 vs 对象

在IPFS里，一个块引用一个数据单元，块的标识是它的键(哈希)。一个块可以是任何种类的数据，而且没有必要有任何关联的格式。对象遵循Merkle DAG protobuf数据格式引用了一个块，可以通过`ipfs object`命令解析操作一个对象。任何给定的哈希都可能代表一个对象，也可能代表一个块。

### 从头创建一个块

创建你自己的块很简单！把数据放到文件里然后运行`ipfs block put <你的文件>`。或者用管道的方式把文件数据提供给`ipfs block put`，比如：

```
$ echo "This is some data" | ipfs block put
QmfQ5QAjvg4GtA3wg3adpnDJug8ktA1BxurVqBD8rtgVjM
$ ipfs block get QmfQ5QAjvg4GtA3wg3adpnDJug8ktA1BxurVqBD8rtgVjM
This is some data
```

注意：当创建了你自己的块数据后，是不能用`ipfs cat`读这些数据的。这是因为没有使用unixfs数据格式来输入这些数据，要想读取原始的块，可以使用例子中的`ipfs block get`。

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/data/readme.md)
