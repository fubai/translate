## IPFS的网站

### 一个把网站托管到IPFS上的简短指南

把静态网站托管到IPFS上是非常简单的！只需打开daemon：

```bash
$ ipfs daemon
```

然后添加网站所在的目录：

```bash
$ ls mysite
img index.html
$ ipfs add -r mysite
added QmcMN2wqoun88SVF5own7D5LUpnHwDA6ALZnVdFXhnYhAs mysite/img/spacecat.jpg
added QmS8tC5NJqajBB5qFhcA1auav14iHMnoMZJWfmr4k3EY6w mysite/img
added QmYh6HbZhHABQXrkQZ4aRRSoSa6bb9vaKoHeumWex6HRsT mysite/index.html
added QmYeAiiK1UfB8MGLRefok1N7vBTyX8hGPuMXZ4Xq1DPyt7 mysite/
```

离文件夹名字最近的哈希(倒数第二个)就是咱们想要的，现在开始就叫它`$SITE_HASH`吧。

现在就能在浏览器里打开测试`http://localhost:8080/ipfs/$SITE_HASH`了！下一步可以到其它IPFS节点上查看`http://gateway.ipfs.io/ipfs/$SITE_HASH`。酷不酷？但是这些哈希也太丑了，咱们看看能不能摆脱它们。

首先可以用一个包含了`dnslink=/ipfs/$SITE_HASH`的简单DNS TXT记录。这个纪录生效后就可以用`http://localhost:8080/ipns/your.domain`查看了，现在看起来干净多了，还能在网关上看`http://gateway.ipfs.io/ipns/your.domain`。

下面你可能会问：“挺好的，但是如果我要修改网站呢？DNS太慢了！”。好的，让我告诉你一个叫做Ipns的小东西吧(注意这个“n”)。Ipns是星际命名系统，可能你已经注意到上面的链接里有`/ipns/`而不是`/ipfs/`了。Ipns可用于IPFS网络里的可变内容，它比较易用，能让你更改网站后不必每次都更新dns纪录！那它怎么用呢？

添加网页后只需要以下操作：

```bash
$ ipfs name publish $SITE_HASH
Published to <your peer id>: /ipfs/$SITE_HASH
```

现在可以通过查看链接来测试它是否工作：`http://localhost:8080/ipns/<your peer id>`，也可以通过公共网关查看这个地址。要是确认能工作，咱们就再来隐藏哈希吧。把DNS TXT纪录修改为`dnslink=/ipns/<your peer id>`，等待DNS纪录生效后访问`http://localhost:8080/ipns/your.domain`。

这时候就有了一个在ipfs/ipns上的网站了。你可能好奇怎么才能把这个网站暴露到`http://your.domain`呢，这样才能让今天的互联网用户不用知道这些就访问它啊？这真的出奇的简单，所有要做的就是前面创建的TXT纪录，以及一个`your.domain`的A纪录。这个A纪录指向一个IP地址，这个IP地址上有一个IPFS后台程序在运行，而且监听了80端口的HTTP请求(比如`gateway.ipfs.io`)。用户浏览器会在请求头Host里发送`your.domain`，有上面的dnslink TXT纪录，所以ipfs网关能把`your.domain`辨别成一个IPNS名称，这样就把`/`替换成了`/ipns/your.domain/`。

所以如果把`your.domain`的A纪录指向`gateway.ipfs.io`的IP之后，所有人就能不用任何额外的配置通过`http://your.domain`访问托管在IPFS上的网站了。

Happy Hacking!

By [Whyrusleeping](https://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/websites/README.md)
