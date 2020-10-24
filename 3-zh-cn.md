## 3. Starting HTTP/2 / 开始 HTTP/2
> An HTTP/2 connection is an application-layer protocol running on top of a TCP connection ([TCP]). The client is the TCP connection initiator.

HTTP/2 是一个运行在 TCP 之上的应用层协议。客户端是 TCP 连接的发起者。

> HTTP/2 uses the same "http" and "https" URI schemes used by HTTP/1.1. HTTP/2 shares the same default port numbers: 80 for "http" URIs and 443 for "https" URIs. As a result, implementations processing requests for target resource URIs like http://example.org/foo or https://example.com/bar are required to first discover whether the upstream server (the immediate peer to which the client wishes to establish a connection) supports HTTP/2.

HTTP/2 使用和 HTTP/1.1 一样的 "http" 和 "https" 的 URL 模式。同时 HTTP/2 和 HTTP/1.1 也共享了相同的默认端口号："http" 的 80 端口，"https" 的 443 端口。因此，实现者在处理类似 http://example.org/foo 或 https://example.com/bar 这样 URI 的目标资源请求之前，需要首先确定上游服务端（即当前客户端希望直接与之建立连接的对端）是否支持 HTTP/2。

> The means by which support for HTTP/2 is determined is different for "http" and "https" URIs. Discovery for "http" URIs is described in Section 3.2. Discovery for "https" URIs is described in Section 3.3.

对于 "http" 和 "https" 两种 URI，检测是否支持 HTTP/2 的方法是不同的。检测 "http" 的 URI 在 3.2 节中描述。检测 "https" 的 URI 在 3.3 节中描述。

### 3.1 HTTP/2 Version Identification / HTTP/2 版本标识
> The protocol defined in this document has two identifiers.
>
> * The string "h2" identifies the protocol where HTTP/2 uses Transport Layer Security (TLS) [TLS12]. This identifier is used in the TLS application-layer protocol negotiation (ALPN) extension [TLS-ALPN] field and in any place where HTTP/2 over TLS is identified.
>
> 	The "h2" string is serialized into an ALPN protocol identifier as the two-octet sequence: 0x68, 0x32.
> * The string "h2c" identifies the protocol where HTTP/2 is run over cleartext TCP. This identifier is used in the HTTP/1.1 Upgrade header field and in any place where HTTP/2 over TCP is identified.
>
>  The "h2c" string is reserved from the ALPN identifier space but describes a protocol that does not use TLS.

本文档定义的协议有两个标识符：

* 字符串 "h2" 标示使用了 TLS(Transport Layer Security)*[TLS12]* 的 HTTP/2 协议。该标识符用在 TLS-ALPN(application-layer protocol negotiation)*[TLS-ALPN]* 的扩展字段，以及其他需要标示运行于 TLS 之上 HTTP/2 的地方。

  "h2" 字符串序列化成 ALPN 协议标识符时是两个字节的序列：0x68，0x32。

* 字符串 "h2c" 标示运行在明文 TCP 之上的 HTTP/2 协议。该标识符用在 HTTP/1.1 的 Upgrade 首部字段，以及其他需要标示运行于 TCP 之上 HTTP/2 的地方。

  "h2c" 字符串保留在 ALPN 标识符空间，但是实际上标示了一个不使用 TLS 的协议。

> Negotiating "h2" or "h2c" implies the use of the transport, security, framing, and message semantics described in this document.

协商 "h2" 或 "h2c" 需要用到本文档里描述的传输，安全，成帧和消息语义等概念。

### 3.2 Starting HTTP/2 for "http" URIs / 为 "http" URI 启用 HTTP/2 协议
> A client that makes a request for an "http" URI without prior knowledge about support for HTTP/2 on the next hop uses the HTTP Upgrade mechanism (Section 6.7 of [RFC7230]). The client does so by making an HTTP/1.1 request that includes an Upgrade header field with the "h2c" token. Such an HTTP/1.1 request MUST include exactly one HTTP2-Settings (Section 3.2.1) header field.
>
> For example:
>
> ```
> GET / HTTP/1.1
> Host: server.example.com
> Connection: Upgrade, HTTP2-Settings
> Upgrade: h2c
> HTTP2-Settings: <base64url encoding of HTTP/2 SETTINGS payload>
> ```

客户端在发起一个 "http" URI 的请求时，如果事先不知道下一跳是否支持 HTTP/2 ，需要使用 HTTP Upgrade 机制(*[RFC7230]*的6.7节)。客户端首先发起一个 HTTP/1.1 请求，其中包含值为 "h2c" 的 Upgrade 首部字段。该请求还必须包含一个 HTTP2-Settings (3.2.1节)首部字段。

例如：

```
GET / HTTP/1.1
Host: server.example.com
Connection: Upgrade, HTTP2-Settings
Upgrade: h2c
HTTP2-Settings: <base64url encoding of HTTP/2 SETTINGS payload>
```

> Requests that contain a payload body MUST be sent in their entirety before the client can send HTTP/2 frames. This means that a large request can block the use of the connection until it is completely sent.

在客户端能发送 HTTP/2 帧之前，包含有效载荷的请求必须被全部发送。这意味着一个大尺寸的请求会独占连接, 直到它全部发送完毕。

> If concurrency of an initial request with subsequent requests is important, an OPTIONS request can be used to perform the upgrade to HTTP/2, at the cost of an additional round trip.

对于一个具有后续请求的初始请求来说，如果它并发性十分重要，那么可以先发送一个 OPTIONS 类型的请求, 将协议升级到 HTTP/2, 但这样做会带来一个额外的往返开销。

> A server that does not support HTTP/2 can respond to the request as though the Upgrade header field were absent:
>
> ```
> HTTP/1.1 200 OK
> Content-Length: 243
> Content-Type: text/html
> ...
> ```

不支持 HTTP/2 的服务端响应请求时，可以认为 Upgrade 首部字段不存在：

```
HTTP/1.1 200 OK
Content-Length: 243
Content-Type: text/html
...
```

> A server MUST ignore an "h2" token in an Upgrade header field. Presence of a token with "h2" implies HTTP/2 over TLS, which is instead negotiated as described in Section 3.3.

服务端必须忽略值为 "h2" 的 Upgrade 首部字段。"h2" 标示使用 TLS 的 HTTP/2，其协商过程在 3.3 节中描述。

> A server that supports HTTP/2 accepts the upgrade with a 101 (Switching Protocols) response. After the empty line that terminates the 101 response, the server can begin sending HTTP/2 frames. These frames MUST include a response to the request that initiated the upgrade.
>
> For example:
>
> ```
> HTTP/1.1 101 Switching Protocols
> Connection: Upgrade
> Upgrade: h2c
>
> [ HTTP/2 connection ...
> ```

支持 HTTP/2 的服务器响应状态码 101 (Switching Protocols) 表示接受升级协议的请求。在结束 101 响应的空行之后，服务端可以开始发送 HTTP/2 帧。这些帧必须包含一个对初始升级请求的响应。

例如：

```
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: h2c

[ HTTP/2 connection ...
```

> The first HTTP/2 frame sent by the server MUST be a server connection preface (Section 3.5) consisting of a SETTINGS frame (Section 6.5). Upon receiving the 101 response, the client MUST send a connection preface (Section 3.5), which includes a SETTINGS frame.

服务端发送的第一个 HTTP/2 帧必须是由一个 SETTINGS 帧(6.5节)组成的服务端连接前奏(3.5节)。客户端收到 101 响应之后，也必须发送一个包含 SETTINGS 帧的连接前奏。

> The HTTP/1.1 request that is sent prior to upgrade is assigned a stream identifier of 1 (see Section 5.1.1) with default priority values (Section 5.3.5). Stream 1 is implicitly "half-closed" from the client toward the server (see Section 5.1), since the request is completed as an HTTP/1.1 request. After commencing the HTTP/2 connection, stream 1 is used for the response.

升级之前发送的 HTTP/1.1 请求被分配了一个有着默认优先级值(5.3.5节)的流标识符 1(5.1.1节)。流 1 暗示从客户端到服务端(5.1节)是半关闭的，因为该请求作为HTTP/1.1 请求已经完成了。HTTP/2 连接开始后，流 1 用于响应。

#### 3.2.1 HTTP2-Settings Header Field / HTTP2-Settings 首部字段
> A request that upgrades from HTTP/1.1 to HTTP/2 MUST include exactly one HTTP2-Settings header field. The HTTP2-Settings header field is a connection-specific header field that includes parameters that govern the HTTP/2 connection, provided in anticipation of the server accepting the request to upgrade.
>
> ```
> HTTP2-Settings    = token68
> ```

从 HTTP/1.1 升级到 HTTP/2 的请求必须包含且只包含一个 HTTP2-Settings 首部字段。HTTP2-Settings 是一个连接相关的首部字段，它提供了用于管理 HTTP/2 连接的参数（前提是服务端接受了升级请求）。

```
HTTP2-Settings    = token68
```

> A server MUST NOT upgrade the connection to HTTP/2 if this header field is not present or if more than one is present. A server MUST NOT send this header field.

如果该首部字段没有出现，或者出现了不止一个，那么服务端一定不要把连接升级到 HTTP/2。服务端一定不要发送该首部字段。

> The content of the HTTP2-Settings header field is the payload of a SETTINGS frame (Section 6.5), encoded as a base64url string (that is, the URL- and filename-safe Base64 encoding described in Section 5 of [RFC4648], with any trailing '=' characters omitted). The ABNF [RFC5234] production for token68 is defined in Section 2.1 of [RFC7235].

HTTP2-Settings 首部字段的值是 SETTINGS 帧(6.5节)的有效载荷，被编码成了 base64url 串(即，*[RFC4648]*第 5 节描述的 URL 和文件名安全 Base64 编码，忽略任何结尾'='字符)。ABNF*[RFC5234]*产生 token68 在*[RFC7235]*2.1节有定义。

> Since the upgrade is only intended to apply to the immediate connection, a client sending the HTTP2-Settings header field MUST also send HTTP2-Settings as a connection option in the Connection header field to prevent it from being forwarded (see Section 6.1 of [RFC7230]).

因为升级操作只适用于相邻端点的直连，发送 HTTP2-Settings 首部字段的客户端也必须在发送的 Connection 首部字段值里加上 HTTP2-Settings ，防止它被转发(参见*[RFC7230]*的 6.1 节)。

> A server decodes and interprets these values as it would any other SETTINGS frame. Explicit acknowledgement of these settings (Section 6.5.3) is not necessary, since a 101 response serves as implicit acknowledgement. Providing these values in the upgrade request gives a client an opportunity to provide parameters prior to receiving any frames from the server.

就像对其他的 SETTINGS 帧那样，服务端对这些值进行解码和解释。对这些设置(6.5.3节)进行显式的确认是没有必要的，因为 101 响应本身就相当于隐式的确认。升级请求中提供这些值，让客户端有机会在收到服务端的帧之前就设置一些参数。

### 3.3 Starting HTTP/2 for "https" URIs / 为 "https" URI 启用 HTTP/2 协议
> A client that makes a request to an "https" URI uses TLS [TLS12] with the application-layer protocol negotiation (ALPN) extension [TLS-ALPN].

客户端对 "https" URI 发起请求时使用 TLS-ALPN(带有应用层协议协商扩展 (ALPN) 的 TLS [TLS12])。

> HTTP/2 over TLS uses the "h2" protocol identifier. The "h2c" protocol identifier MUST NOT be sent by a client or selected by a server; the "h2c" protocol identifier describes a protocol that does not use TLS.

运行在 TLS 之上的 HTTP/2 使用"h2"协议标识符。此时，客户端不能发送 "h2c" 协议标识符，服务端也不能选择 "h2c" 协议标识符；"h2c" 协议标识符表示 HTTP/2 不使用 TLS。

> Once TLS negotiation is complete, both the client and the server MUST send a connection preface (Section 3.5).

一旦 TLS 协商完成，客户端和服务端都必须发送一个连接序言(3.5节)。

### 3.4 Starting HTTP/2 with Prior Knowledge / 先验知识下启用 HTTP/2
> A client can learn that a particular server supports HTTP/2 by other means. For example, [ALT-SVC] describes a mechanism for advertising this capability.

客户端可以通过其他方式了解服务端是否支持 HTTP/2。例如 *[ALT-SVC]* 就描述了一种广播服务端能力的机制。

> A client MUST send the connection preface (Section 3.5) and then MAY immediately send HTTP/2 frames to such a server; servers can identify these connections by the presence of the connection preface. This only affects the establishment of HTTP/2 connections over cleartext TCP; implementations that support HTTP/2 over TLS MUST use protocol negotiation in TLS [TLS-ALPN].

客户端必须先向这种服务端发送连接序言(3.5)，然后可以立即发送 HTTP/2 帧。服务端能通过连接序言识别出这种连接。这只影响基于明文 TCP 的 HTTP/2 连接。基于 TLS 的 HTTP/2 实现必须使用 TLS 中的协议协商*[TLS-ALPN]*。

> Likewise, the server MUST send a connection preface (Section 3.5).

同样，服务端也必须发送一个连接序言(3.5节)。

> Without additional information, prior support for HTTP/2 is not a strong signal that a given server will support HTTP/2 for future connections. For example, it is possible for server configurations to change, for configurations to differ between instances in clustered servers, or for network conditions to change.

在没有额外的参考信息的情况下，某个服务端先前支持 HTTP/2 并不能表明它在以后的连接中仍会支持 HTTP/2。例如，服务端配置有可能改变了，或者集群中不同服务器配置有差异，或者网络状况改变了。

### 3.5 HTTP/2 Connection Preface / HTTP/2 连接序言
> In HTTP/2, each endpoint is required to send a connection preface as a final confirmation of the protocol in use and to establish the initial settings for the HTTP/2 connection. The client and server each send a different connection preface.

在 HTTP/2 连接中，要求两端都要发送一个连接序言，作为对所使用协议的最终确认，并确定 HTTP/2 连接的初始设置。客户端和服务端各自发送不同的连接序言。

> The client connection preface starts with a sequence of 24 octets, which in hex notation is:
>
> ```
> 0x505249202a20485454502f322e300d0a0d0a534d0d0a0d0a
> ```
> That is, the connection preface starts with the string PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n). This sequence MUST be followed by a SETTINGS frame (Section 6.5), which MAY be empty. The client sends the client connection preface immediately upon receipt of a 101 (Switching Protocols) response (indicating a successful upgrade) or as the first application data octets of a TLS connection. If starting an HTTP/2 connection with prior knowledge of server support for the protocol, the client connection preface is sent upon connection establishment.

客户端连接序言以一个24字节的序列开始，用十六进制表示为：

```
0x505249202a20485454502f322e300d0a0d0a534d0d0a0d0a
```

连接序言以字符串 "PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n" 开始。这个序列后面必须跟一个可以为空的 SETTINGS 帧(6.5节)。客户端一收到 101(Switching Protocols) 响应（表示成功升级）后，就发送客户端连接序言，或者作为 TLS 连接的第一批应用数据字节。如果在预先知道服务端支持 HTTP/2 的情况下启用 HTTP/2 连接，客户端的连接序言在连接建立时发送。

> Note: The client connection preface is selected so that a large proportion of HTTP/1.1 or HTTP/1.0 servers and intermediaries do not attempt to process further frames. Note that this does not address the concerns raised in [TALKING].

注意：客户端连接前奏是专门挑选的，目的是为了让大部分 HTTP/1.1 或 HTTP/1.0 服务器和中介不会试图处理后面的帧，但这并没有解决在 *[TALKING]* 中提出的问题。

> The server connection preface consists of a potentially empty SETTINGS frame (Section 6.5) that MUST be the first frame the server sends in the HTTP/2 connection.

服务端连接前奏包含一个可能为空的 SETTINGS 帧(6.5节)，它必须是服务端在 HTTP/2 连接中发送的第一个帧。

> The SETTINGS frames received from a peer as part of the connection preface MUST be acknowledged (see Section 6.5.3) after sending the connection preface.

在发送完本端的连接前奏之后，必须对来自对端的作为连接前奏一部分的 SETTINGS 帧进行确认。

> To avoid unnecessary latency, clients are permitted to send additional frames to the server immediately after sending the client connection preface, without waiting to receive the server connection preface. It is important to note, however, that the server connection preface SETTINGS frame might include parameters that necessarily alter how a client is expected to communicate with the server. Upon receiving the SETTINGS frame, the client is expected to honor any parameters established. In some configurations, it is possible for the server to transmit SETTINGS before the client sends additional frames, providing an opportunity to avoid this issue.

为了避免不必要的时延，允许客户端发送完连接前奏后就立即向服务端发送其他的帧，而不必等待服务端的连接前奏。不过需要注意的是，服务端连接前奏的 SETTINGS 帧可能包含一些期望客户端如何与服务端进行通信所必须修改的参数。在收到这些 SETTINGS 帧以后，客户端应当遵守所有设置的参数。在某些配置中，服务端是可以在客户端发送额外的帧之前传送 SETTINGS 帧的，这样就避免了之前所说的问题。

> Clients and servers MUST treat an invalid connection preface as a connection error (Section 5.4.1) of type PROTOCOL_ERROR. A GOAWAY frame (Section 6.8) MAY be omitted in this case, since an invalid preface indicates that the peer is not using HTTP/2.

客户端和服务端都必须将无效的连接前奏处理为连接错误(5.4.1节)，错误类型为 PROTOCOL_ERROR。在这种情况下，可以不发送 GOAWAY 帧(6.8节)，因为无效的连接前奏表示对端并没有使用 HTTP/2。


