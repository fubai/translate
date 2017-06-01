## 制作自己的IPFS服务

IPFS有几个默认运行的默认服务，比如dht，bitswap和诊断服务。这些服务只是在IPFS的节点主机上注册一个处理程序，在其中监听新的连接。`corenet`包为此功能提供了一个非常干净的接口。下面尝试构建一个简单的示例服务来试试吧！

先从构建服务主机开始：

```
package main

import (
	"fmt"

	core "github.com/ipfs/go-ipfs/core"
	corenet "github.com/ipfs/go-ipfs/core/corenet"
	fsrepo "github.com/ipfs/go-ipfs/repo/fsrepo"

	"code.google.com/p/go.net/context"
)
```

不用导入太多的包，现在唯一需要的就是main函数了：

设置ipfs节点。

```
func main() {
	// ipfs节点基础设置
	r, err := fsrepo.Open("~/.ipfs")
	if err != nil {
		panic(err)
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	cfg := &core.BuildCfg{
		Repo:   r,
		Online: true,
	}

	nd, err := core.NewNode(ctx, cfg)

	if err != nil {
		panic(err)
	}
```

上面是一段基础的模版代码，以用户`~/.ipfs`目录下的config文件初始化了一个默认ipfs节点。

下面开始构建服务。

```
	list, err := corenet.Listen(nd, "/app/whyrusleeping")
	if err != nil {
		panic(err)
	}

	fmt.Printf("I am peer: %s\n", nd.Identity.Pretty())

	for {
		con, err := list.Accept()
		if err != nil {
			fmt.Println(err)
			return
		}

		defer con.Close()

		fmt.Fprintln(con, "Hello! This is whyrusleepings awesome ipfs service")
		fmt.Printf("Connection from: %s\n", con.Conn().RemotePeer())
	}
}
```

这就是在ipfs上写一个服务所全部需要的了。当客户端连接时，给它发送一个问候，打印它的节点ID，然后关闭会话。这是个简单的服务，当然你可以为处理连接而写一些真正的逻辑。

现在需要一个客户端连向我们：

```
package main

import (
	"fmt"
	"io"
	"os"

	core "github.com/ipfs/go-ipfs/core"
	corenet "github.com/ipfs/go-ipfs/core/corenet"
	peer "github.com/ipfs/go-ipfs/p2p/peer"
	fsrepo "github.com/ipfs/go-ipfs/repo/fsrepo"

	"golang.org/x/net/context"
)

func main() {
	if len(os.Args) < 2 {
		fmt.Println("Please give a peer ID as an argument")
		return
	}
	target, err := peer.IDB58Decode(os.Args[1])
	if err != nil {
		panic(err)
	}

	// ipfs节点基础设置
	r, err := fsrepo.Open("~/.ipfs")
	if err != nil {
		panic(err)
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	cfg := &core.BuildCfg{
		Repo:   r,
		Online: true,
	}

	nd, err := core.NewNode(ctx, cfg)

	if err != nil {
		panic(err)
	}

	fmt.Printf("I am peer %s dialing %s\n", nd.Identity, target)

	con, err := corenet.Dial(nd, target, "/app/whyrusleeping")
	if err != nil {
		fmt.Println(err)
		return
	}

	io.Copy(os.Stdout, con)
}
```

这个客户端会设置它自己的ipfs节点(注意：这是相当昂贵的，通常不会只为单个连接启动一个实例)，并且请求我们上面创建的服务。

在一台电脑上运行下面的命令来试试：

```
$ ipfs init # 做过了就不用再做了
$ go run host.go
```

应该会打印出节点的ID，拷贝这个ID并在第二台电脑上用：

```
$ ipfs init # 做过了就不用再做了
$ go run client.go <peerID>
```

这应该会打印出`Hello! This is whyrusleepings awesome ipfs service`

你现在可能会问自己：“为什么要用这个？它比`net`包好哪儿了？”。下面就是它的优势：

1. 请求的目标是peerID，而不是IP地址。
2. 可以利用内置在我们网络包里的NAT打洞功能。
3. 替换掉端口号，获得了一个更有意义的协议ID字符串。

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/api/service/readme.md)
