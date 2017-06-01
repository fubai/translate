## IPFS配置文件

可以通过一个JSON格式的文本文件配置IPFS，这个文件在`~/.ipfs/config`。


### 地址

配置文件存储了几种不同的地址类型，它们全都是multiaddr寻址格式。一起来看看这几个地址类型都是啥意思吧。

```
"Addresses": {
    "Swarm": [
      "/ip4/0.0.0.0/tcp/4001"
    ],
    "API": "/ip4/127.0.0.1/tcp/5001",
    "Gateway": "/ip4/127.0.0.1/tcp/8080"
  }
```

#### Swarm

Swarm地址是一组供IPFS本地后台进程监听的，用于和其它对等节点建立连接的地址。要确保这些地址能从其它电脑上拨通，而且没有防火墙阻塞地址的端口。

#### API

API地址用于后台程序对外提供HTTP的API。这些API用于从命令行控制后台程序，或者通过curl来体验。请确保API地址不能从本机之外的地方访问，不然潜在的恶意攻击者就可以给本机的IPFS后台程序发送命令了。

#### Gateway

Gateway地址用于后台程序对外提供网关接口。网关能用于通过IPFS查看文件，并提供静态的Web内容。它的端口可以被也可以不被其它机器访问，这全部取决于你。网关地址是可选的，如果把它设置为空，网关服务器就不会启动。

### Mounts

Mounts配置项用于为IPFS和IPNS虚拟文件系统指定默认的挂载点，如果`ipfs mount`命令没有指定其它目录的话，就会用Mounts配置项里的内容。挂载时这些文件夹需要存在，并且用户要有权限通过fuse挂载到它们。

### Bootstrap

Bootstrap数组配置了后台程序启动时要连接的IPFS节点的列表，默认值是“IPFS阳光网络”节点，这是一组分布在全国(美国?)各地的VPS服务器。

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/config/readme.md)
