# multiformats

> Self-describing values for Future-proofing -- 自描述的值，面向未来

计算中的每个选择都有权衡，这包括格式，算法，编码等等。而且即使有大量的规划，也可能导致决策不能适用于未来的变化，或者解决方案不再是最优的。所以让系统能进化和成长是非常重要的。

multiformats项目是一组在今天面向未来的协议。为了面向未来，这组协议通过“自描述”来增强格式的值。这带来了互通性和协议灵活性，避免了被锁死在某一个格式上。

协议的“自描述”部分有几个规定：

* 它们必须和值在一起，而不是在值外面(比如在上下文中)。
* 它们必须提升可扩展性以避免使用者被锁死在某个值上
* 它们必须小巧，有二进制包的表示方式
* 它们必须要有人类可读的表示方式

现在有如下几个multiformat协议：

* multihash - 自描述哈希
* multiaddr - 自描述网络地址
* multibase - 自描述base编码
* multicodec - 自描述序列化
* multistream - 自描述流网络协议
* multigram (WIP) - 自描述包网络协议

## multihash

multihash是一种用于区分不同哈希函数输出的协议，主要关注点是大小和编码。它对于那些要让自己的哈希的使用能面向未来的应用程序来说是有用的，而且它还能允许多个哈希函数共存。

multihash在一个依赖[加密安全哈希函数](https://en.wikipedia.org/wiki/Cryptographic_hash_function)的系统中显得特别重要。攻击[可能会破解](https://en.wikipedia.org/wiki/Hash_function_security_summary)安全哈希函数的加密属性。[密码的破解](https://en.wikipedia.org/wiki/Cryptanalysis)在一个大型工具生态系统里是特别痛苦的，因为这些工具可能已经在哈希值上做了假定，比如依赖某个哈希函数或摘要大小，于是对哈希函数的升级就变成了噩梦，因为这些做了假定的工具必须被升级到新的哈希函数和新的哈希摘要长度。

>
有多少程序假设了git的哈希就是sha1哈希？
有多少脚本假设了哈希摘要就是160位？
有多少工具在这些值变化时会被破坏？
有多少程序在这些值变化时会静默失败？

这就是multihash发光的地方，它就是被设计为用于升级的。

### multihash格式

multihash遵循TLV(type-length-value -- 类型-长度-值)模式。

* `<哈希函数类型>`是一个[无符号整数变量](https://github.com/multiformats/unsigned-varint)，用于辨别不同的哈希函数。有一个可配置的[默认列表](https://github.com/multiformats/multihash/blob/master/hashtable.csv)记录了不同哈希函数的代码。
* `<摘要长度>`是一个[无符号整数变量](https://github.com/multiformats/unsigned-varint)，是摘要的字节长度。
* `<摘要值>`是哈希函数计算出来的结果。

所以一个multihash是这样的：

`<可变的哈希函数代码><可变的摘要字节大小><哈希函数摘要>`

比如：

`122041dd7b6443542e75701aa98a0c235951a28a0d851b11564d20022ab11d2589a8`

其中前两位是哈希函数类型，`12`代表`sha2-256`；第3和第4位是摘要的字节长度`20`；剩下的就是摘要。

## multiaddr

multiaddr是一种为不同网络协议编码地址的格式。它对于那些要让自己的地址的使用能面向未来的应用程序来说是有用的，而且它还能允许多个传输协议和地址共存。

现在互联网的网络寻址体系不是自描述的。以下形式的地址需要太多释义和特定场景的上下文。它们是建立在一些假设之上的，这也导致应用程序要做这些假设，于是产生了许多“这种地址”特定的代码。网络地址和它们的协议“锈死”在了其应用的位置，不能被未来的协议替代，因为寻址方式阻碍了这种替换。

比如，请思考：

```
127.0.0.1:9090   # ip4。这是TCP，UDP，还是其它什么协议?
[::1]:3217       # ip6。这是TCP，UDP，还是其它什么协议?

http://127.0.0.1/baz.jpg
http://foo.com/bar/baz.jpg
//foo.com:1234
 # 用了DNS，解析成ip4或ip6
 # 但后面确实用了tcp。或者也许是quic... >.<
 # 这些默认假设了TCP端口 :80。
```

### multiaddr格式

multiaddr值是一个递归的`(TLV)+`(类型-长度-值+)编码。有两种形式：

* 人类可读 - 用于展示给用户(UTF-8)。
* 二进制 - 用于存储，在线路上传播，以及其它格式的原生格式。

#### 人类可读版：

格式为：`(/<协议字符串代码>/<地址值>)+`。

* `<协议字符串代码>` - 网络协议的字符串代码，有一个可配置的[默认代码列表](https://github.com/multiformats/multiaddr/blob/master/protocols.csv)。
* `<地址值>` - 是网络地址的原生字符串值。
* `+` - 加号代表`重复`，一个协议可以包含其它协议。

例如：`/ip4/127.0.0.1/udp/4023/quic`。有的协议可能只有代码，比如`/quic`；也有有地址值的，比如`/ip4/127.0.0.1`。

#### 二进制版：

格式为：`(<协议代码><地址值长度><地址值>)+`。

* `<协议代码>` - 网络协议的整数代码，有一个可配置的[默认代码列表](https://github.com/multiformats/multiaddr/blob/master/protocols.csv)。
* `<地址值长度>` - 无符号整数，地址值的字节长度，如果一个协议没有地址值或地址值的长度是确定的，会抛弃该长度值。
* `<地址值>` - 是网络地址的值，如果协议没有地址值，会将其抛弃。
* `+` - 加号代表`重复`，一个协议可以包含其它协议。

## multibase

multibase是一个用于区分base编码和其它简单字符串编码的协议，并且用于确保和程序接口的完全兼容能力。它是这个问题的答案：

> 给定数据d，将其编码为字符串s，怎样说明d的base编码呢？

multibase解决的最大问题是：一个程序用了multibase后就可以采用任意编码的输入和输出了。因为这些值都是自描述的，这能让其它程序也知道某个值用什么编码。

### multibase格式

multibase的格式为：`<base编码的代码><被编码的数据>`。

`<base编码的代码>`请参考[这里](https://github.com/multiformats/multibase/blob/master/multibase.csv)。

## multicodec

`multicodec`是一种自描述的*multiformat*，它通过一点自描述信息包装了其它格式。multicodec的标识符是一个表明了其后数据的代码，这意味每个multicodec的最高位代码是保留的，以用于表明其后的代码是什么类型的。

multicodec的格式如下：

`<multicodec-varint><encoded-data>`

请参考multicodec的[代码列表](https://github.com/multiformats/multicodec/blob/master/table.csv)。