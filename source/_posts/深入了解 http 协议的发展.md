---
title: 深入了解 http 协议
date: 2023/2/19
updated: 2023/2/19
tags: http
categories: http
description: "详细介绍 http/1.x, http/2, http/3 的内容"
---

## [HTTP/1.1——标准化的协议](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP#http1.1——标准化的协议)

- 连接可以复用，节省了多次打开 TCP 连接加载网页文档资源的时间。
- 增加管线化技术，允许在第一个应答被完全发送之前就发送第二个请求，以降低通信延迟。
- 支持响应分块。
- 引入额外的缓存控制机制。
- 引入内容协商机制，包括语言、编码、类型等。并允许客户端和服务器之间约定以最合适的内容进行交换。
- 凭借 [`Host`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Host) 标头，能够使不同域名配置在同一个 IP 地址的服务器上。

### [HTTP 流水线](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Connection_management_in_HTTP_1.x#http_流水线)

流水线是在同一条长连接上发出连续的请求，而不用等待应答返回。这样可以避免连接延迟。理论上讲，性能还会因为两个 HTTP 请求有可能被打包到一个 TCP 消息包中而得到提升。就算 HTTP 请求不断的继续，尺寸会增加，但设置 TCP 的[最大分段大小（MSS）](https://zh.wikipedia.org/wiki/最大分段大小)选项，仍然足够包含一系列简单的请求。

并不是所有类型的 HTTP 请求都能用到流水线：只有[幂等](https://developer.mozilla.org/zh-CN/docs/Glossary/Idempotent)方式，比如 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)、[`HEAD`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD)、[`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) 和 [`DELETE`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/DELETE) 能够被安全地重试。如果有故障发生时，流水线的内容要能被轻易的重试。

> **备注：** HTTP 流水线在现代浏览器中并不是默认被启用的：
>
> - 正确的实现流水线是复杂的：传输中的资源大小、多少有效的 [RTT](https://zh.wikipedia.org/wiki/來回通訊延遲) 会被用到以及有效带宽都会直接影响到流水线提供的改善。不知道这些的话，重要的消息可能被延迟到不重要的消息后面。这个重要性的概念甚至会演变为影响到页面布局！因此 HTTP 流水线在大多数情况下带来的改善并不明显。
> - 流水线受制于[队头阻塞（HOL）](https://zh.wikipedia.org/wiki/队头阻塞)问题。.

## [HTTP/2——为了更优异的表现](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP#http2——为了更优异的表现)

- **HTTP/2 是二进制协议**而不是文本协议。不再可读，也不可无障碍的手动创建，改善的优化技术现在可被实施。
- 这是一个**多路复用协议**。并行的请求能在同一个链接中处理，移除了 HTTP/1.x 中顺序和阻塞的约束。
- **压缩了标头**。因为标头在一系列请求中常常是相似的，其移除了重复和传输重复数据的成本。
- 其允许服务器在客户端缓存中填充数据，通过一个叫服务器推送的机制来提前请求。

### Http/2 多路复用

HTTP/2 引入了一个额外的步骤：它将 HTTP/1.x 消息分成帧并嵌入到流（stream）中。数据帧和报头帧分离，这将允许报头压缩。将多个流组合，这是一个被称为*多路复用*（multiplexing）的过程，它允许更有效的底层 TCP 连接。

### Http/2 服务器推送

服务可以主动向客户端发送消息。在浏览器刚请求 `HTML` 的时候，服务端会把某些资源存在一定的关联性 `JS、CSS` 等文件等静态资源主动发给客户端，这样**客户端可以直接从本地加载这些资源**，不用再通过网络再次请求，以此来达到节省浏览器发送 request 请求的过程。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26b896162456493a8f7ec59a9e12310d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

使用服务器推送

```html
Link: </css/styles.css>; rel=preload; as=style，
</img/example.png>; rel=preload; as=image
```

可以看到服务器**initiator**中的 push 状态表示这是服务端进行主动推送。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/592e497aba0842c2ad1e59f3220c22fb~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

```
对于主动推送的文件势必会带来多余或已经浏览器已有一份的文件
```

客户端使用一个简洁的 **Cache Digest** 来告诉服务器，哪些东西已经在缓存，因此服务器也就会知道哪些是客户端所需要的。

### Http/2 缺点

- TCP 以及 TCP+TLS 建立连接的延迟（握手延迟）
- http2.0 中 TCP 的**队头阻塞依然没有彻底解决**，连接双方的有任一个数据包丢失，或任一方的网络中断，整个 TCP 连接就会暂停，丢失的数据包需要被重新传输，从而阻塞该 TCP 连接中的所有请求，反而在网络较差或不稳定情况下，使用多个连接表现更好。

## [HTTP/3——基于 QUIC 的 HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP#http3——基于_quic_的_http)

- **Real Streams**：是实际的直接数据传输速度提升；
- **改进的握手**：是连接建立速度的改进；
- **连接迁移**：减少必要的重新连接次数，从而减少会话之间的连接建立和数据丢失，从而提高速度；

[Real Streams](https://valerii-udodov.com/posts/http3/#real-streams)

因为在 TCP 中，所有数据包都在“线路”中传输，如果一个数据包变慢或丢失，则整个“线路”必须等待慢速/丢失的数据包回到其位置并继续前进。尽管 TCP 有流的概念，但它们不是真正的流，而是模拟它们的抽象。多个文件被分解成数据包，数据包被分组在小帧中，但无论如何都通过“行”中的单个流。

![TCP Stream](https://valerii-udodov.com/images/posts/http3/tcp-stream.jpeg)

QUIC 实现了 streams，这是真正的 stream。所有的数据将真正通过一个连接但是多个流在流动。例如：JS bundle 文件可能是一个流并标记另一个流

![QUIC Stream](https://valerii-udodov.com/images/posts/http3/quic-stream.jpeg)

除了通过真正并行化数据包传输来提高速度之外，还有一个恼人的问题已经解决。线路阻塞的头。

你知道，当你匆忙在你的杂货店排队时。前面有一个超级慢的人，只会减慢整条线的速度。嗯，这就是 HoL Blocking。

> 当单个（慢速）对象阻止其他/后续对象取得进展时

HTTP 的所有版本，都以某种方式在不同级别上遭受了[行头阻塞 (HoL)](https://calendar.perfplanet.com/2020/head-of-line-blocking-in-quic-and-http-3-the-details/)。在 HTTP/2 中，由于单流性质，它是 TCP HoL 阻塞。

![TCP HoL 阻塞](https://valerii-udodov.com/images/posts/http3/tcp-hol-blocking.jpeg)

QUIC 并没有真正解决 HoL 阻塞问题，而是将它移到了另一个地方。虽然它没有完全解决问题，但这种转变显着提高了性能。

![QUIC 霍尔阻塞](https://valerii-udodov.com/images/posts/http3/quic-hol-blocking.jpeg)

HoL 阻塞可能发生在流级别。因为流顺序仍然很重要，所以您不能重新排序或丢失数据包。因此，如果流中有一个慢家伙，流将无法继续，必须等待……但在这种情况下，它不会阻塞其他流。一旦烦人的数据包问题得到解决，流就会恢复。

[握手](https://valerii-udodov.com/posts/http3/#handshake)

QUIC 至少需要少一次握手。新连接只需要一次握手，恢复连接不需要握手。这使得 QUIC 1RTT 用于新连接，0RTT 用于恢复。

![握手](https://valerii-udodov.com/images/posts/http3/handshaking.png)

[安全](https://valerii-udodov.com/posts/http3/#security)

HTTP 是纯文本协议。它只是通过电线（或无线电波）发送文本。HTTPS 中的 S 代表 Secure。通过使用 HTTPS，我们指示浏览器使用 TLS 协议参与通信。TLS 协议将使用高级数学算法加密我们的数据。这样可以确保即使数据可能被第三方拦截，也没有用。现在，Internet 上的大多数 Web 资源都使用 HTTPS。
即使现代浏览器坚持使用 HTTPS，我们仍然可以避免它。

![HTTP2 与 HTTP3](https://valerii-udodov.com/images/posts/http3/http2-vs-http3.jpeg)

对于 QUIC，这是不可避免的。因为 TLS 是内置在 QUIC 中的。因此对于 HTTP/3，将不再需要`https://`和端口`433`。开箱即用的一切都将被加密。而且没有办法关闭加密。

### 连接迁移

连接迁移：当客户端切换网络时，和服务器的连接并不会断开，仍然可以正常通信，对于 TCP 协议而言，这是不可能做到的。因为 TCP 的连接基于 4 元组：源 IP、源端口、目的 IP、目的端口，只要其中 1 个发生变化，就需要重新建立连接。但 **QUIC 的连接是基于 64 位的 Connection ID**，网络切换并不会影响 Connection ID 的变化，连接在逻辑上仍然是通的

## 参考

- [Evolution of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)
- [HTTP/3. All you need to know.](HTTP/3. All you need to know.)
- [HTTP Messages](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages)
