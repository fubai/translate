## 播放视频

IPFS可用于存储播放视频。假设添加一个视频：

```
ipfs add -q sintel.mp4 | tail -n1
```

拿着上面输出的哈希，可以用几种不同的方式播放视频：

以命令行的方式：

```
ipfs cat $vidhash | mplayer -vo xv -
```

以本地网关的方式：

```
mplayer http://localhost:8080/ipfs/$vidhash

# 或者用Chrome标签页打开它(火狐也行)

chromium http://localhost:8080/ipfs/$vidhash
```

(注意: 网关的方式适用于大多数视频播放器和浏览器)

By [whyrusleeping](http://github.com/whyrusleeping)

[原文链接](https://ipfs.io/ipfs/QmNZiPk974vDsPmQii3YbrMKfi12KTSNM7XMiYyiea4VYZ/example#/ipfs/QmRFTtbyEp3UaT67ByYW299Suw7HKKnWK6NJMdNFzDjYdX/videos/readme.md)