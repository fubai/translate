## 快照

让我们来快速看下IPFS怎样能被用于取得基础快照。

保存一个目录：

```
$ ipfs add -r ~/code/myproject
```

记下哈希：

```
$ echo $hash `date` >> snapshots
```

或者一条命令搞定：

```
$ echo `ipfs add -q -r ~/code/myproject | tail -n1` `date` >> snapshots
```

(注意：`-q`会让输出只包含哈希，管道符后面的`tail -n1`能确保只输出最上层文件夹的哈希。)

确保挂载点目录的存在：

```
$ sudo mkdir /ipfs /ipns
$ sudo chown `whoami` /ipfs /ipns
```

你需要把`Fuse`安装到你的机器上，以保证能把IPFS`挂载`到目录上。可以在[怎样安装Fuse](https://github.com/ipfs/go-ipfs/blob/master/docs/fuse.md)中发现相关命令。

查看存活的快照：

```
$ ipfs mount
$ ls /ipfs/$hash/

# 还可以

$ cd /ipfs/$hash/
$ ls
```

通过fuse接口，在获取快照之后，你就能真的像访问文件系统里的文件一样来访问IPFS中的文件了。

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/snapshots/readme.md)