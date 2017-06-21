## Git，能更分散

你曾经有没有对自己说过：“少年，我的Git服务器不够分散啊”或者“我想有一个简单的方式能提供一个世界范围的Git仓库服务”。别光想了，这就有个解决方案！

本文将讨论怎样用IPFS网络提供一个Git仓库服务。最后的结果是一个可`git clone`的IPFS地址！

开始前要先选择一个待托管的Git仓库，然后bare克隆它：

```
$ git clone --bare git@myhost.io/myrepo
```

这里给对Git不是很熟的朋友解释下，一个bare仓库没有工作区，可以被用作服务器，和普通Git仓库的格式有点不同。

现在，把这个仓库设置到待克隆状态：

```
$ cd myrepo
$ git update-server-info
```

下面是可选的去解包所有的Git对象：

```
$ cp objects/pack/*.pack .
$ git unpack-objects < ./*.pack
$ rm ./*.pack
```

解包是把Git的大打包文件分割成所有独立对象的小文件。这样做可以让IPFS在你添加了多个版本的该Git仓库时去除重复对象。

上面的事做完后就准备好了一个可以对外服务的仓库了。剩下的是把这个仓库添加到IPFS里：

```
$ pwd
/code/myrepo
$ ipfs add -r .
...
...
...
added QmX679gmfyaRkKMvPA4WGNWXj9PtpvKWGPgtXaF18etC95 .
```

现在就剩下克隆这个仓库了：

```
$ cd /tmp
$ git clone http://localhost:8080/ipfs/QmX679gmfyaRkKMvPA4WGNWXj9PtpvKWGPgtXaF18etC95 myrepo
```

注意：确保用你自己得到的哈希

现在你可能会问：“好吧，多好的一个Git仓库啊，我都不能改”。好吧，让我告诉你一个NB的用例！我倾向于用Go编程，有些同学不知道Go所import的路径，其实是可以用版本控制的，比如：

```go
import (
	"github.com/whyrusleeping/mycoollibrary"
)
```

Go的import是一个很nice的特性，它解决了很多问题，但经常有问题的是，我用了某些人的库，然后他们把库里的API给改了，我的代码就出错了。按咱们上面做的，可以把引用的库克隆下来，然后添加到IPFS里，那么导入的路径就变成了：

```go
import (
	mylib "gateway.ipfs.io/ipfs/QmX679gmfyaRkKMvPA4WGNWXj9PtpvKWGPgtXaF18etC95"
)
```

这样就保证了每次都有相同的代码了！

注意：由于Go不允许使用localhost做为import的路径，所以要用公共的HTTP网关。这样就没了安全保证，因为可能有中间人攻击，导致你会收到有问题的代码。不过可以使用域名把请求重定向到本机以避免这个问题。

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/git/readme.md)
