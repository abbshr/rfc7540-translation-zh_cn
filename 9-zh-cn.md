# 9. 其他HTTP要求/注意事项

> This section outlines attributes of the HTTP protocol that improve   interoperability, reduce exposure to known security vulnerabilities,   or reduce the potential for implementation variation.

这部分概述了`HTTP`协议的属性，包括提高互通性、 减少暴露已知的安全漏洞或者减少执行变异的可能。

## 9.1 连接管理

> HTTP/2 connections are persistent.  For best performance, it is   expected that clients will not close connections until it is   determined that no further communication with a server is necessary   (for example, when a user navigates away from a particular web page)   or until the server closes the connection.

`HTTP/2`的连接是持久性的。 为了获得最佳的性能，在确定与服务器没有进一步的连接必要之前客户端将不会关闭连接（例如：当用户导航到其他特定的网页时这个连接就会关闭），或者直到服务器端关闭连接之前。

> Clients SHOULD NOT open more than one HTTP/2 connection to a given   host and port pair, where the host is derived from a URI, a selected   alternative service [ALT-SVC], or a configured proxy.

客户端不应该给一个给定的主机和端口号打开多个`HTTP/2`连接，其中这个主机来源于一个选定替代服务([参见：ALT-SVG](https://tools.ietf.org/html/rfc7540#ref-ALT-SVC))或者配置代理的统一资源标识符(URI)。

> A client can create additional connections as replacements, either to   replace connections that are near to exhausting the available stream   identifier space (Section 5.1.1), to refresh the keying material for   a TLS connection, or to replace connections that have encountered   errors (Section 5.4.1).

客户端可以创建额外的连接作为替代，或者取代快要耗尽的可用流标识符空间（[参见：Section 5.1.1](http://tools.ietf.org/html/rfc7540#section-5.1.1))，或者为`TLS`连接刷新密钥，或者替换遇到的错误连接（[参见: Section 5.4.1](http://tools.ietf.org/html/rfc7540#section-5.4.1)）。

> A client MAY open multiple connections to the same IP address and TCP   port using different Server Name Indication [TLS-EXT] values or to   provide different TLS client certificates but SHOULD avoid creating   multiple connections with the same configuration.

一个客户端可以针对相同的`IP`地址和`TCP`端口号用不同的服务器端名称标识符或者用不同的`TCL`客户端证书创建多个连接，但是应该避免用相同的配置创建多个连接。

> Servers are encouraged to maintain open connections for as long as   possible but are permitted to terminate idle connections if   necessary.  When either endpoint chooses to close the transport-layer   TCP connection, the terminating endpoint SHOULD first send a GOAWAY   (Section 6.8) frame so that both endpoints can reliably determine   whether previously sent frames have been processed and gracefully   complete or terminate any necessary remaining tasks.

服务器端被鼓励尽可能长时间的保持打开的连接，但是在必要的情况下要关闭空余的连接。当任意一个端口选择关闭传输层的`TCP`连接时，决定关闭的终端应该首先发送一个超时帧，这样两个终端都能可靠的确定之前发送的帧是否已经被处理、“优雅”的完成或者终止任何剩余的必要任务。

### 9.1.1 连接复用

> Connections that are made to an origin server, either directly or   through a tunnel created using the CONNECT method (Section 8.3), MAY   be reused for requests with multiple different URI authority   components.  A connection can be reused as long as the origin server   is authoritative (Section 10.1).  For TCP connections without TLS,   this depends on the host having resolved to the same IP address.

连接到源服务器的连接，无论是直接或者通过由`CONNECT`方法([参见： Section 8.3](http://tools.ietf.org/html/rfc7540#section-8.3))建立的隧道连接,都可能重复使用于多个不同`URL`权限请求组件。只要源服务器是权威的，连接就可以被复用（[参见： Section 10.1](http://tools.ietf.org/html/rfc7540#section-10.1)）。对于没有`TLS`的`TCP`连接，这取决于对同一个`IP`地址已经解析的主机端。

> For "https" resources, connection reuse additionally depends on   having a certificate that is valid for the host in the URI.  The   certificate presented by the server MUST satisfy any checks that the   client would perform when forming a new TLS connection for the host   in the URI.

对于`HTTP`资源，连接复用还额外取决于是否拥有一个对于`URL`中的主机已经验证过的证书。该证书由服务器提供并且必须满足这种形式的检查，即在`URL`主机中形成一个新的`TLS`连接时客户端将执行任何检查。

> An origin server might offer a certificate with multiple   "subjectAltName" attributes or names with wildcards, one of which is   valid for the authority in the URI.  For example, a certificate with   a "subjectAltName" of "*.example.com" might permit the use of the   same connection for requests to URIs starting with   "https://a.example.com/" and "https://b.example.com/".

源服务器可能提供一个拥有多个`subjectAltName`属性或者通配符的证书，其中之一是有效授权的`URL`。例如：一个带有`*.example.com`的`subjectAltName`证书将允许`a.example.com`和`b.example.com`使用同一个连接。

>  In some deployments, reusing a connection for multiple origins can   result in requests being directed to the wrong origin server.  For   example, TLS termination might be performed by a middlebox that uses   the TLS Server Name Indication (SNI) [TLS-EXT] extension to select an   origin server.  This means that it is possible for clients to send   confidential information to servers that might not be the intended   target for the request, even though the server is otherwise   authoritative.   
> 
> A server that does not wish clients to reuse connections can indicate   that it is not authoritative for a request by sending a 421   (Misdirected Request) status code in response to the request (see   Section 9.1.2).   
> 
> A client that is configured to use a proxy over HTTP/2 directs   requests to that proxy through a single connection.  That is, all   requests sent via a proxy reuse the connection to the proxy.

### 9.1.2 421（误导请求）状态码

> The 421 (Misdirected Request) status code indicates that the request   was directed at a server that is not able to produce a response.   This can be sent by a server that is not configured to produce   responses for the combination of scheme and authority that are   included in the request URI.

421（误导请求）状态码标识的是一个不能够产生响应的服务器请求。这个状态码可能会在服务器没有为产生经过计划和认证的响应进行相关的配置的时候产生。

> Clients receiving a 421 (Misdirected Request) response from a server   MAY retry the request -- whether the request method is idempotent or   not -- over a different connection.  This is possible if a connection   is reused (Section 9.1.1) or if an alternative service is selected   [ALT-SVC].

客户端接收到服务器端发送的421(误导请求)响应可以通过不同的连接重试请求——不管这个请求方式是不是幂等的。这个连接可能是复用的（参见：[Section 9.1.1](https://tools.ietf.org/html/rfc7540#section-9.1.1)），或者可选的服务被选择([参见 ALT-SVC](https://tools.ietf.org/html/rfc7540#ref-ALT-SVC))。

> This status code MUST NOT be generated by proxies.

这个状态码绝对不能由代理端产生。

> A 421 response is cacheable by default, i.e., unless otherwise   indicated by the method definition or explicit cache controls (see   Section 4.2.2 of [RFC7234]).

421的响应缓存是默认的。除非被定义的方法或者显式缓存控制另有说明

## 9.2 使用TLS功能

> Implementations of HTTP/2 MUST use TLS version 1.2 [TLS12] or higher   for HTTP/2 over TLS.  The general TLS usage guidance in [TLSBCP]   SHOULD be followed, with some additional restrictions that are   specific to HTTP/2.

`HTTP/2`的实现必须使用`TLS 1.2`([参见： TLS12](https://tools.ietf.org/html/rfc7540#ref-TLS12))或者更高的版本。通用的`TLS`指导（[参见： TLSBCP](https://tools.ietf.org/html/rfc7540#ref-TLSBCP)）应该遵循，同时还需加上`HTTP/2`的一些额外限制条件。

>   The TLS implementation MUST support the Server Name Indication (SNI)   [TLS-EXT] extension to TLS.  HTTP/2 clients MUST indicate the target   domain name when negotiating TLS.

`TLS`的实现必须支持服务器名称指示(SNI)的`TLS`扩展([参见：TLS-EXT](https://tools.ietf.org/html/rfc7540#ref-TLS-EXT))。`HTTP/2`客户端在协商`TLS`的时候必须注明目标域名。

> Deployments of HTTP/2 that negotiate TLS 1.3 or higher need only   support and use the SNI extension; deployments of TLS 1.2 are subject   to the requirements in the following sections.  Implementations are   encouraged to provide defaults that comply, but it is recognized that   deployments are ultimately responsible for compliance.

`HTTP/2`的部署中在协商`TLS1.3`或者更高的版本时只需要支持和使用服务器域名指示（SNI）扩展。`TLS1.2`的部署需要遵循下面章节的要求。在实现过程中鼓励提供符合要求的默认值，部署的最终负责合规性。

### 9.2.1 TLS 1.2功能

> This section describes restrictions on the TLS 1.2 feature set that   can be used with HTTP/2.  Due to deployment limitations, it might not   be possible to fail TLS negotiation when these restrictions are not   met.  An endpoint MAY immediately terminate an HTTP/2 connection that   does not meet these TLS requirements with a connection error   (Section 5.4.1) of type INADEQUATE_SECURITY.

本节描述了`HTTP/2`使用的`TLS 1.2`功能集限制。由于部署的限制，当一些限制没有遇到的时候`TLS`可能无法断开协商。终端必须立即终止一个不符合`TLS`要求的`HTTP/2`连接，并且把他们当做INADEQUATE_SECURITY处理（[参见： Section5.4.1](https://tools.ietf.org/html/rfc7540#section-5.4.1)）。

> A deployment of HTTP/2 over TLS 1.2 MUST disable compression.  TLS   compression can lead to the exposure of information that would not   otherwise be revealed [RFC3749].  Generic compression is unnecessary   since HTTP/2 provides compression features that are more aware of   context and therefore likely to be more appropriate for use for   performance, security, or other reasons.

通过`TLS 1.2`进行的`HTTP/2`部署必须禁止压缩。`TLS`的压缩可能导致信息的暴露（[参见：RFC3749](https://tools.ietf.org/html/rfc3749)）。通用的压缩是不必要的，因为`HTTP/2`提供的压缩功能能够更好的联系上下文，因此可能更加的符合性能要求，安全或者其他方面。

> A deployment of HTTP/2 over TLS 1.2 MUST disable renegotiation.  An   endpoint MUST treat a TLS renegotiation as a connection error   (Section 5.4.1) of type PROTOCOL_ERROR.  Note that disabling renegotiation can result in long-lived connections becoming unusable   due to limits on the number of messages the underlying cipher suite   can encipher.

通过`TLS 1.2`进行的`HTTP/2`部署必须禁止重新协商。终端必须将`TLS`协商定义为连接错误，类型为`PROROCOL_ERROR`（[参见 Section 5.4.1](https://tools.ietf.org/html/rfc7540#section-5.4.1)）。务必注意由于密码套件可以加密消息的次数限制，禁止重新协商可能会导致长期连接编程不可用。

> An endpoint MAY use renegotiation to provide confidentiality   protection for client credentials offered in the handshake, but any   renegotiation MUST occur prior to sending the connection preface.  A   server SHOULD request a client certificate if it sees a renegotiation   request immediately after establishing a connection.

终端可以使用协商来对客户端握手提供机密性的保护，但是任何协商必须在发送连接前言之前。在建立连接后，服务器如果看到一个协商请求后会马上协商请求客户端证书。

> This effectively prevents the use of renegotiation in response to a   request for a specific protected resource.  A future specification   might provide a way to support this use case.  Alternatively, a   server might use an error (Section 5.4) of type HTTP_1_1_REQUIRED to   request the client use a protocol that supports renegotiation.

这有效的防止了使用协商相应来请求一个特定的受保护资源。未来的规范可能会提供一种方式来支持这种需求。可替代地，服务器可以使用类型为`HTTP_1_1_REQUIRED`的错误（[参见： Section 5.4](https://tools.ietf.org/html/rfc7540#section-5.4)）来请求客户端使用支持重新协商的协议。

> Implementations MUST support ephemeral key exchange sizes of at least   2048 bits for cipher suites that use ephemeral finite field Diffie-   Hellman (DHE) [TLS12] and 224 bits for cipher suites that use   ephemeral elliptic curve Diffie-Hellman (ECDHE) [RFC4492].  Clients   MUST accept DHE sizes of up to 4096 bits.  Endpoints MAY treat   negotiation of key sizes smaller than the lower limits as a   connection error (Section 5.4.1) of type INADEQUATE_SECURITY.

实现必须在最少支持2048位短暂密码套件（DHE）——使用短暂的迪菲－赫尔曼密钥交换([参见：TLS12](https://tools.ietf.org/html/rfc7540#ref-TLS12))和244位的密码套件（ecdhe）——使用短暂的椭圆曲线迪菲－赫尔曼密钥交换（[参见：RFC4492](https://tools.ietf.org/html/rfc4492)）中等支持密钥交换的加密套件中使用。客户端必须接受高达4096的`DHE`尺寸。终端可以把小于密钥最小尺寸的协商看作为类型为`INADEQUATE_SECURITY`连接错误（[参见 Section 5.4.1](https://tools.ietf.org/html/rfc7540#section-5.4.1)）

### 9.2.2 TLS 1.2加密套件

> A deployment of HTTP/2 over TLS 1.2 SHOULD NOT use any of the cipher   suites that are listed in the cipher suite black list (Appendix A).

通过`TLS 1.2`进行的`HTTP/2`部署不应该使用任何流出在加密套件黑名单中的密码套件。

> Endpoints MAY choose to generate a connection error (Section 5.4.1)   of type INADEQUATE_SECURITY if one of the cipher suites from the   black list is negotiated.  A deployment that chooses to use a black-   listed cipher suite risks triggering a connection error unless the   set of potential peers is known to accept that cipher suite.

如果黑名单列表中的密码套件已经进行了协商，终端可以选择生成一个类型为`INADEQUATE_SECURITY`的连接错误。除非对等端接收这个密码套件，否则部署中选择使用黑名单中的密码套件会触发连接错误。

> Implementations MUST NOT generate this error in reaction to the   negotiation of a cipher suite that is not on the black list.   Consequently, when clients offer a cipher suite that is not on the   black list, they have to be prepared to use that cipher suite with   HTTP/2.

黑名单包括密码套件，`TLS 1.2`是强制性的，这就意外着`TLS 1.2`的部署可能使用密码套件的非相交集。为了避免这个问题使得`TLS`的握手失败，通过`TLS 1.2`部署的`HTTP/2`必须支`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`([参见：TLS-ECDHE](https://tools.ietf.org/html/rfc7540#ref-TLS-ECDHE))与P-256椭圆曲线(参见： [FIPS186](https://tools.ietf.org/html/rfc7540#ref-FIPS186))。

> Note that clients might advertise support of cipher suites that are   on the black list in order to allow for connection to servers that do   not support HTTP/2.  This allows servers to select HTTP/1.1 with a cipher suite that is on the HTTP/2 black list.  However, this can   result in HTTP/2 being negotiated with a black-listed cipher suite if   the application protocol and cipher suite are independently selected.

需要注意的是为了允许连接不支持HTTP/2的服务器，客户端上可能宣传支持黑名单上的密码套件。这使得服务器用`HTTP/2`黑名单上的密码套件使用`HTTP/1.1`。然而，如果应用协议和加密套件是独立选择的，这可能会导致`HTTP/2`与黑名单上的密码套件进行协商。