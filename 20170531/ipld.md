# IPLD

> The "thin-waist" merkle dag format

![](https://img.shields.io/badge/status-draft-green.svg?style=flat-square)

UPDATE: 我们为处理链接重新draft了这个规范，希望能尽快重新finalize。任何不便之处敬请谅解。这是开始实现前要做的一个重要的修改。

有很多系统使用了受默克尔树和哈希链启发的数据结构(比如git，bittorrent，ipfs，tahoe-lafs，sfsro)。IPLD (行星链接化数据)定义了:

- **_merkle-links_**: 基于默克尔树结构形成图结构的核心单位。
- **_merkle-dag_**: 边缘是 _merkle-link_ 的图。
- **_merkle-paths_**: unix风格的路径，以 _命名化的merkle-link_ 在 _merkle-dag_ 之间穿行。
- **IPLD数据模型**: 一个灵活的，基于JSON的数据模型，用于表示merkle-dag。
- **IPLD序列化格式**: 可以表示IPLD对象的一些列格式，比如JSON，CBOR，CSON，YAML，Protobuf，XML，RDF等等。
- **IPLD标准格式**: 序列化格式的确定性描述，用于确保相同的 _logical_ 对象始终被序列化为 _完全相同的bit序列_ 。这对默克尔式的连接以及所有的加密应用来说至关重要。

简而言之：具有命名化的merkle-link的JSON文档能被遍历。

## 介绍

### 什么是 _merkle-link_ ?

_merkle-link_ 是两个对象之间的链接，它们以目标对象的内容算出 _密码式的哈希_ ，并以其做寻址，这些链接会嵌入到源对象里。用merkle-link以内容做寻址能：

- **密码式的完整性校验**：解析到的链接的内容可以通过哈希测试。反过来，这种机制带来了广泛，安全，不依赖信任机制的数据交换(比如git或者bittorrent)，因为其它人没法给你和哈希不匹配的内容。
- **不可变的数据结构**：有默克尔链接的数据结构是不可变的，这对分布式系统来说非常nice。这对控制版本很有用，可用于表示分布式可变状态(比如CRDT)，以及长期归档。

IPLD对象模型里用map表示一个 _merkle-link_ ，这个map只有一个键`/`，它被映射到一个"链接值"。比如：

**下面是个用json表示的链接，作为一个"链接对象"**

```js
{ "/" : "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k" }
// "/"是链接的键
// "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k"是链接的值
```

**下面是个有个链接的对象，链到`foo/baz`**

```js
{
  "foo": {
    "bar": "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k", // 不是一个链接
    "baz": {"/": "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k"} // 链接
  }
}
```

**下面这个对象有个假"链接对象"，链到`files/cat.jpg`，同时它有个真链接，链到`files/cat.jpg/link`**

```js
{
  "files": {
    "cat.jpg": { // 给的链接的属性被封装在另一个对象里
      "link": {"/": "/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k"}, // 真正的链接
      "mode": 0755,
      "owner": "jbenet"
    }
  }
}
```

在解析链接的引用时，链接自身的map会被替换成它指向的对象，除非这个链接的路径是无效的。

链接可以是一个multihash，这种情况它是被假设成`/ipfs`层次结构中的一个链接，链接还能直接是对象的绝对路径。不过现在只允许`/ipfs`层级结构。

如果一个应用程序要把只有一个`/`键的对象用作其它目的，这个应用程序就要自己负责转义IPLD对象里的`/`键，这样这个应用程序的键就不会和IPLD对象的`/`键冲突了。

### 什么是 _merkle-graph_ 或 _merkle-dag_ ？

具有merkle-link的对象会形成图(merkle-graph)，这个对象和这个图都必须是定向的，能不被计算成环，如果密码哈希函数的属性成立的话。这就是 _merkle-dag_ 。因此所有用了 _merkle来做连接_ 的图( _merkle-graph_ )必须是一个有向无环图(DAG是有向无环图的简称，所以才有了 _merkle-dag_ )。

### 什么是 _merkle-path_ ？

merkle-path是一个unix风格的路径(比如`/a/b/c/d`)，它最初通过 _merkle-link_ 解析引用，可以传递性的访问被引用的节点和其它节点。

通用的文件系统可以在IPLD的上层设计一种对象模型，能专用于文件维护并有专有的路径算法可以在这个模型上做查询。

### _merkle-paths_ 如何工作？

_merkle-path_ 是一种unix风格的路径，它的找寻过程是先通过 _merkle-link_ 解析引用，然后跟随被引用的中间对象中的 _被命名的merkle-link_ 。跟随一个名字意味着要到对象里面去查找，先找到这个 _name_ ，然后解析成关联的 _merkle-link_ 。

比如，假设我们有下面这个 _merkle-path_ ：

```
/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/c/d
```

在这里:
- `ipfs`是一个协议命名空间(让计算机能辨明要做什么)
- `QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k`是一个密码哈希。
- `a/b/c/d`是遍历路径，就像unix路径一样。

遍历路径是由`/`表明的，会在两种链接上出现:

- **对象内部遍历** 在同一个对象里遍历数据。
- **跨对象遍历** 通过解析merkle-link从一个对象遍历到另一个对象。

#### 例子

使用下面的数据集:

    > ipfs object cat --fmt=yaml QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k
    ---
    a:
      b:
        link:
          /: QmV76pUdAAukxEHt9Wp2xwyTpiCmzJCvjnMxyQBreaUeKT
        c: "d"
        foo:
          /: QmQmkZPNPoRkPd7wj2xUJe5v5DsY6MX33MFaGhZKB2pRSE

    > ipfs object cat --fmt=yaml QmV76pUdAAukxEHt9Wp2xwyTpiCmzJCvjnMxyQBreaUeKT
    ---
    c: "e"
    d:
      e: "f"
    foo:
      name: "second foo"

    > ipfs object cat --fmt=yaml QmQmkZPNPoRkPd7wj2xUJe5v5DsY6MX33MFaGhZKB2pRSE
    ---
    name: "third foo"

路径的例子:

- `/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/c` 只会遍历第一个对象，得到字符串`d`。
- `/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/link/c` 会遍历上面两个对象，得到字符串`e`
- `/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/link/d/e` 遍历上面两个对象，得到字符串`f`
- `/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/link/foo/name` 遍历第一个和第二个对象，得到字符串`second foo`
- `/ipfs/QmUmg7BZC1YP1ca66rRtWKxpXp77WgVHrnv263JtDuvs2k/a/b/foo/name` 遍历第一个和最后一个对象，得到字符串`third foo`


## 什么是IPLD数据模型？

IPLD数据模型为所有的merkle-dag定义了一个简单的基于JSON的 _结构_ ，并且标识了一组可用于编码这个结构的格式。

### 限制和期望

一些限制:
- IPLD路径必须是明确的。给定的一个路径字符串必须始终遍历到同一个对象。(例如，避免重复的链接名称)
- IPLD路径必须是通用的，且能避免对非英语人群的不友好性(比如，使用UTF-8，而不是ASCII)。
- IPLD路径必须干净的处于UNIX和Web之上(使用`/`以确保在仅支持ASCII码的系统中被转换)。
- 鉴于JSON的巨大成功，大量的系统提供了JSON接口。IPLD必须能够简单的导入导出JSON。
- JSON数据模型也很简单易用。IPLD也必须同样易用。
- 定义新的数据结构必须非常简单。在IPLD之上实验去定义新的数据结构时，不能繁琐或要求太多相关知识。
- 由于IPLD基于JSON数据模型，所以它和RDF以及以JSON-LD作为互联数据的标准是完全兼容的。
- IPLD的序列化格式(在存储和传输上)必须要快且空间高效。(不应该用JSON做为存储格式，而是用CBOR或类似的格式)
- IPLD的密码哈希必须是可升级的(使用[multihash](https://github.com/jbenet/multihash))。

一些期望拥有的:
- IPLD不应该携带遗留的错误，比如JSON缺失Integer的问题(JSON有Number类型)。
- IPLD应该是可升级的，比如，当一个更适合磁盘读写的的格式出现时，系统要能迁移到新格式上去，并且可以最小化花费的代价。
- IPLD对象也应该能把属性解析成路径，而不只是用默克尔链接。
- IPLD标准格式应该是一种能为其简单的写一个解析器的格式。
- IPLD标准格式应该能在不解析整个对象的情况就可以查找。(CBOR和Protobuf可以)。

### 格式定义

(**注:** 下面会用JSON和YML来展示格式，这里明确的用这两种格式来展示一个对象在两种不同格式上的相等性。)

从IPLD数据模型的核心说，它“是JSON”：这么说是因为(a)IPLD还是基于树结构的文档，只是多了一些私有的类型，(b)和json是一对一的映射关系，(c)用户可以通过JSON本身的方式使用IPLD。当然IPLD也“不是JSON”：(a)它改进了一些错误，(b)它有一个高效的序列化表示方式，(c)实际上没有指定一个单一在用的格式，所以可以改进已经存在的东西。

#### 基础节点

下面是IPLD对象JSON格式的例子：

```json
{
  "name": "Vannevar Bush"
}
```

假设这个对象的哈希值为`QmAAA...AAA`。记住这个对象根本没有链接，只有一个键为name的字符串值。但我们还是能“解析”这个对象的`name`键：

```sh
> ipld cat --json QmAAA...AAA
{
  "name": "Vannevar Bush"
}

> ipld cat --json QmAAA...AAA/name
"Vannevar Bush"
```

而且 -- 当然 -- 咱们也能用其他格式查看这个对象

```sh
> ipld cat --yml QmAAA...AAA
---
name: Vannevar Bush

> ipld cat --xml QmAAA...AAA
<!xml> <!-- todo -->
<node>
  <name>Vannevar Bush</name>
</node>
```

#### 节点之间的链接

在对象之间以默克尔算法形成的链接就是IPLD存在的原因。一个IPLD中的链接就是一个嵌入在另一个对象里的特殊格式的节点：

```js
{
  "title": "As We May Think",
  "author": {
    "/": "QmAAA...AAA" // 上面那个对象节点的链接
  }
}
```

假设这个对象的哈希值为`QmBBB...BBB`。这个对象节点把子路径`author`链接到`QmAAA...AAA`，也就是上面那个对象节点。所以我们现在可以：

```sh
> ipld cat --json QmBBB...BBB
{
  "title": "As We May Think",
  "author": {
    "/": "QmAAA...AAA" // 上面那个对象节点的链接
  }
}

> ipld cat --json QmBBB...BBB/author
{
  "name": "Vannevar Bush"
}

> ipld cat --yml QmBBB...BBB/author
---
name: "Vannevar Bush"

> ipld cat --json QmBBB...BBB/author/name
"Vannevar Bush"
```

#### 链接属性约定

IPLD允许用户构建复杂的数据结构，用其他和链接关联的属性。这能很有用的把其他信息和链接编码在一起，比如关系信息，或者链接需要的辅助信息。这和“链接对象约定”是 _不同_ 的，下面会讨论“链接对象约定”，在它们正确的情况是很有用的。有时候只需要在链接上添加一点数据，不必添加另一个对象。通过简单的在一个对象上嵌入真实的IPLD链接就可以做到，这个链接上会有额外的属性。

> 重要提示：不允许链接对象里直接存在链接属性，这是因为会在遍历时有歧义。请阅读规范历史里关于这些困难的一个讨论，。

举个例子，假设有一个文件系统，要指定如权限之类的元数据，或者在对象之间的链接上指明对象的拥有者。假设有一个哈希为`QmCCC...CCC`的`目录`对象，如下：

```js
{
  "foo": { // 有更多属性的链接包装器
    "link": {"/": "QmCCC...111"} // 链接
    "mode": "0755",
    "owner": "jbenet"
  },
  "cat.jpg": {
    "link": {"/": "QmCCC...222"},
    "mode": "0644",
    "owner": "jbenet"
  },
  "doge.jpg": {
    "link": {"/": "QmCCC...333"},
    "mode": "0644",
    "owner": "jbenet"
  }
}
```

或者YML格式的：

```yml
---
foo:
  link:
    /: QmCCC...111
  mode: 0755
  owner: jbenet
cat.jpg:
  link:
    /: QmCCC...222
  mode: 0644
  owner: jbenet
doge.jpg:
  link:
    /: QmCCC...333
  mode: 0644
  owner: jbenet
```

尽管在 _具体到这种数据结构_ 时，链接上有了新的属性，但还是能很好的解析链接：

```js
> ipld cat --json QmCCC...CCC/cat.jpg
{
  "data": "\u0008\u0002\u0012��\u0008����\u0000\u0010JFIF\u0000\u0001\u0001\u0001\u0000H\u0000H..."
}

> ipld cat --json QmCCC...CCC/doge.jpg
{
  "subfiles": [
    {
      "/": "QmPHPs1P3JaWi53q5qqiNauPhiTqa3S1mbszcVPHKGNWRh"
    },
    {
      "/": "QmPCuqUTNb21VDqtp5b8VsNzKEMtUsZCCVsEUBrjhERRSR"
    },
    {
      "/": "QmS7zrNSHEt5GpcaKrwdbnv1nckBreUxWnLaV4qivjaNr3"
    }
  ]
}

> ipld cat --yml QmCCC...CCC/doge.jpg
---
subfiles:
  - /: QmPHPs1P3JaWi53q5qqiNauPhiTqa3S1mbszcVPHKGNWRh
  - /: QmPCuqUTNb21VDqtp5b8VsNzKEMtUsZCCVsEUBrjhERRSR
  - /: QmS7zrNSHEt5GpcaKrwdbnv1nckBreUxWnLaV4qivjaNr3

> ipld cat --json QmCCC...CCC/doge.jpg/subfiles/1/
{
  "data": "\u0008\u0002\u0012��\u0008����\u0000\u0010JFIF\u0000\u0001\u0001\u0001\u0000H\u0000H..."
}
```

但是在获取链接时不能像获取其它属性一样舒服了，因为链接意味着 _resolve through_ 。

#### 重复的属性键

请注意，具有 _相同_ 名字的两个属性是不允许的，但实际上无法阻止(有些人就会这样做，然后提供给解析器)，所以为了安全起见，我们把用于遍历路径的值定义为，序列化内容里的 _第一个_ 入口。比如，假设有如下对象：

```json
{
  "name": "J.C.R. Licklider",
  "name": "Hans Moravec"
}
```

假设 _这_ 就是 _标准格式_ (不是json，是cbor)里的 _正确顺序_ ，它的哈希值是`QmDDD...DDD`，我们 _总会_ 得到：

```sh
> ipld cat --json QmDDD...DDD
{
  "name": "J.C.R. Licklider",
  "name": "Hans Moravec"
}
> ipld cat --json QmDDD...DDD/name
"J.C.R. Licklider"
```

#### 路径限制

Unix和Web的路径描述出现了一些重要的问题。请看[这个讨论](https://github.com/ipfs/go-ipfs/issues/1710)。为了兼容unix和web的模式和期望，IPLD明确禁止带有了某些路径零件的路径。**请注意数据本身 _也许_ 还带有这些属性(有人会这么做，有合理的用处)。所以只是 _路径解析器_ 禁止解析这样的路径。**这样的限制在典型的unix和UTF-8路径系统里也一样：

TODO:
- [ ] 列出解析路径的限制
- [ ] 展示些例子

#### JSON里的Integer

为了利用JSON的成功，IPLD _直接兼容_ 了JSON，但是IPLD需要不被JSON的错误 _阻碍_ 。这是我们能够为遵循格式的惯用选择而付出的，但是必须注意要确保总有一个定义良好的1:1的映射。

在整数这个主题上，JSON里存在多种能以字符串表示整数的格式。比如[EJSON](https://docs.meteor.com/api/ejson.html)。这些格式的使用和互相之间的转换应该是自然的 -- 也就是说，当把JSON转成CBOR时，一个EJSON整数应该能自然的被转化成一个合适的CBOR整数，而不是在map里以字符串的方式表示。

## 数据序列化格式

IPLD以[multicodec](https://github.com/jbenet/multicodec)支持了多种数据序列化格式。它们可以被以符合指定格式习惯的方式被使用，比如在`CBOR`里，可以用`CBOR`的类型标签表示merkle-link，从而避免写出完整的字符串键`@link`。我们鼓励用户把指定的格式用到极致，以格式最具意义的方式来存储和传输IPLD数据。唯一的要求**是必须有一个定义良好的，能和IPLD标准格式一对一映射的关系。**这样数据就可以从一种格式转换成另一个格式，还能再转回来，这中间不会改变原来的意思，也不会导致哈希值的变化。

### 用CBOR标签序列化

IPLD链接在CBOR里可以用标签表示，定义在[RFC 7049 章节 2.4](http://tools.ietf.org/html/rfc7049#section-2.4)。

定义了一个`<tag-link-object>`标签。这个标签后面可以跟一个文本字符串(主要类型3)，或者跟一个字节字符串(主要类型2)，和链接的目标对应。

在把IPLD的"链接对象"编码到CBOR时，使用如下算法：

- *链接的值*被提取出来。
- 如果*链接的值*是一个有效的[multiaddress](https://github.com/jbenet/multiaddr)，把链接文本转换成multiaddress的二进制字符串，再转回文本要保证产生相同文本的结果，链接被转换成一个二进制multiaddress，以字节字符串(主要类型2)被存储在CBOR格式中。
- 否则，以文本(主要类型3)存储*链接的值*。
- 所得到的编码就是一个`<tag-link-object>`，这个标签后面跟了一个用CBOR表示的*链接的值*。

在解码CBOR并转成IPLD时，每个`<tag-link-object>`的转换按如下算法：

- 被提取出来的跟在标签后面的值必须是*链接的值*
- 如果链接是一个二机制字符串，它会被解释成一个multiaddress，然后被转换成文本格式。如果不是二进制字符串，就直接使用文本字符串。
- 创建一个map，这个map只有一对键值。这个键是一个标准的IPLD*链接键*：`/`，其值是一个文本字符串，内容就是*链接的值*。

如果一个IPLD对象以上面讲述的方式包含了这些标签时，用于表示该对象编码器的multicodec头必须是`/cbor/ipld-tagsv1`，而不能仅仅只是`/cbor`。对象解读方应该能使用一个优化的读进程来使用这些标签检测链接。

**TODO:**

- [ ] register tag with IANA.


### 标准格式

为了维护默克尔算法形成的对象哈希链接的强大能力，我们必须确保有且只有一个IPLD文档的**_标准_**序列化表示。这能保证所有的应用取得一致的密码哈希。应该注意的是，尽管这个序列化表示方式是标准且唯一的，但它还是应当作为一个系统级的参数。未来的系统可能会把它改掉以进化序列化表示方式。不过我们估计这种事情十年也不会有一次。

**IPLD标准格式是 _规范化的CBOR和标签_ 。**

标准的CBOR格式必须遵循定义在[RFC 7049 章节 3.9](http://tools.ietf.org/html/rfc7049#section-3.9)中的规则，当然没有这里定义的规则。

该格式的用户不要期望键之间有指定的顺序，因为在其它非标准格式里键的顺序可能不一样。

以前遗留的标准格式是protocol buffer。

标准格式被用于决定第一次创建对象和计算它的哈希时用什么格式。一旦确定了一个IPLD对象的格式后，这个格式就必须能在所有的通信中使用，这样发送者和接受者就能通过数据检查哈希的正确性了。

比如，当在网络上发送一个用protocol buffer编码的遗留对象时，发送者必须不发送CBOR版本信息，因为多了这些信息之后，接受者将无法验证文件的有效性。

同样的方式，当接收者在存储这个对象时，必须保证这个对象的标准格式和对象一起存储起来，这样就能和其它对等网络节点分享这个对象了。

把这样的对象和其编码格式存储在一起的一个简单方式是，存储的时候带上multicodec头。

## 数据结构的例子

IPLD成为一种简单、敏捷、灵活的格式非常重要，这样才不会成为用户定义新的或导入旧的数据结构的障碍。为此，下面将展示一些数据结构的例子。

### Unix文件系统


#### 一个小文件

```js
{
  "data": "hello world",
  "size": "11"
}
```

#### 一个分块文件

把一个文件分割成多个无依赖的子文件。

```js
{
  "size": "1424119",
  "subfiles": [
    {
      "link": {"/": "QmAAA..."},
      "size": "100324"
    },
    {
      "link": {"/": "QmAA1..."},
      "size": "120345",
      "repeat": "10"
    },
    {
      "link": {"/": "QmAA1..."},
      "size": "120345"
    },
  ]
}
```

#### 一个目录

```js
{
  "foo": {
    "link": {"/": "QmCCC...111"},
    "mode": "0755",
    "owner": "jbenet"
  },
  "cat.jpg": {
    "link": {"/": "QmCCC...222"},
    "mode": "0644",
    "owner": "jbenet"
  },
  "doge.jpg": {
    "link": {"/": "QmCCC...333"},
    "mode": "0644",
    "owner": "jbenet"
  }
}
```

### git

#### git blob

```js
{
  "data": "hello world"
}
```

#### git tree

```js
{
  "foo": {
    "link": {"/": "QmCCC...111"},
    "mode": "0755"
  },
  "cat.jpg": {
    "link": {"/": "QmCCC...222"},
    "mode": "0644"
  },
  "doge.jpg": {
    "link": {"/": "QmCCC...333"},
    "mode": "0644"
  }
}
```

#### git commit

```js
{
  "tree": {"/": "e4647147e940e2fab134e7f3d8a40c2022cb36f3"},
  "parents": [
    {"/": "b7d3ead1d80086940409206f5bd1a7a858ab6c95"},
    {"/": "ba8fbf7bc07818fa2892bd1a302081214b452afb"}
  ],
  "author": {
    "name": "Juan Batiz-Benet",
    "email": "juan@benet.ai",
    "time": "1435398707 -0700"
  },
  "committer": {
    "name": "Juan Batiz-Benet",
    "email": "juan@benet.ai",
    "time": "1435398707 -0700"
  },
  "message": "Merge pull request #7 from ipfs/iprs\n\n(WIP) records + merkledag specs"
}
```

### 比特币

#### 比特币区块

```js
{
  "parent": {"/": "Qm000000002CPGAzmfdYPghgrFtYFB6pf1BqMvqfiPDam8"},
  "transactions": {"/": "QmTgzctfxxE8ZwBNGn744rL5R826EtZWzKvv2TF2dAcd9n"},
  "nonce": "UJPTFZnR2CPGAzmfdYPghgrFtYFB6pf1BqMvqfiPDam8"
}
```

#### 比特币交易

这次用了YML格式。TODO：用真实交易的例子

```yml
---
inputs:
  - input: {/: Qmes5e1x9YEku2Y4kDgT6pjf91TPGsE2nJAaAKgwnUqR82}
    amount: 100
outputs:
  - output: {/: Qmes5e1x9YEku2Y4kDgT6pjf91TPGsE2nJAaAKgwnUqR82}
    amount: 50
  - output: {/: QmbcfRVZqMNVRcarRN3JjEJCHhQBcUeqzZfa3zoWMaSrTW}
    amount: 30
  - output: {/: QmV9PkR2gXcmUgNH7s7zMg9dsk7Hy7bLS18S9SHK96m7zV}
    amount: 15
  - output: {/: QmP8r8fLUnEywGnRRUrHB28nnBKwmshMLiYeg8udzYg7TK}
    amount: 5
script: OP_VERIFY
```
