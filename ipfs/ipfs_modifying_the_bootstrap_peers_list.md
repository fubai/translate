## 引导

IPFS引导列表是IPFS网络里的一些对等节点的列表，IPFS的后台程序会用这个列表从网络上了解更多的节点。IPFS默认附带了一个可信的节点列表，不过可以随意修改这个列表以满足自己的需要。自定义引导列表最常见的用途是创建一个私有的IPFS网络。

首先，咱们把引导列表打印出来：

```
> ipfs bootstrap list
/ip4/104.131.131.82/tcp/4001/ipfs/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
/ip4/104.236.151.122/tcp/4001/ipfs/QmSoLju6m7xTh3DuokvT3886QRYqxAzb1kShaanJgW36yx
/ip4/104.236.176.52/tcp/4001/ipfs/QmSoLnSGccFuZQJzRadHn95W2CrSFmZuTdDWP8HXaHca9z
/ip4/104.236.179.241/tcp/4001/ipfs/QmSoLpPVmHKQ4XTPdz8tjDFgdeRFkpV8JgYq8JVJ69RrZm
/ip4/104.236.76.40/tcp/4001/ipfs/QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64
/ip4/128.199.219.111/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
/ip4/162.243.248.213/tcp/4001/ipfs/QmSoLueR4xBeUbY9WZ9xGUUxunbKWcrNFTDAadQJmocnWm
/ip4/178.62.158.247/tcp/4001/ipfs/QmSoLer265NRgSp2LA3dPaeykiS1J6DifTC88f5uVQKNAd
/ip4/178.62.61.185/tcp/4001/ipfs/QmSoLMeWqB7YGVLJN3pNLQpmmEk35v6wYtsMGLzSr5QBU3
```

上面打印的就是IPFS默认引导节点的地址 -- 这些节点是IPFS开发团队运行的。上面列出的地址完全是以[multiaddr](https://github.com/jbenet/multiaddr)格式解析指定的，所以每个协议都很清楚。以这种方式能让节点很清楚的知道在哪能找到引导节点 -- 它们的位置很明确。

别修改这个列表，除非你明白这么做意味着啥。引导过程是分布式系统里的一个重要的故障安全点：恶意的引导节点能只告诉你同样恶意的节点。所以建议保留IPFS开发团队提供的默认列表，或者 -- 在设置私有IPFS网络的场景下 -- 用一些你能控制的节点的列表。不要向这个列表里添加你不信任的节点。

向引导列表里添加一个新节点：

```
> ipfs bootstrap add /ip4/25.196.147.100/tcp/4001/ipfs/QmaMqSwWShsPg2RbredZtoneFjXhim7AQkqbLxib45Lx4S
```

从引导列表里删除一个节点：

```
> ipfs bootstrap rm /ip4/128.199.219.111/tcp/4001/ipfs/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu
```

假设要为新引导列表创建个备份，把`ipfs bootstrap list`的输出重定向到一个文件里就可以了：

```
> ipfs bootstrap list > save
```

如果要从零开始，可以先一下子删除掉整个引导列表：

```
> ipfs bootstrap rm --all
```

有了一个空的列表就可以重新存储默认的引导列表了：

```
> ipfs bootstrap add --default
``` 

再次清空引导列表，然后重新存储 -- 通过管道符把上面备份的列表文件输入给`ipfs bootstrap add`：

```
> ipfs bootstrap rm --all
> cat save | ipfs bootstrap add
```


By [jbenet](http://github.com/jbenet) and [insanity54](http://github.com/insanity54)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/bootstrap/readme.md)