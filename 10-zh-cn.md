## 10. Security Considerations
## 10. 安全性考虑

### 10.1. Server Authority
### 10.1. 服务器授权

> HTTP/2 relies on the HTTP/1.1 definition of authority for determining
   whether a server is authoritative in providing a given response (see
   [RFC7230], Section 9.1).  This relies on local name resolution for
   the "http" URI scheme and the authenticated server identity for the
   "https" scheme (see [RFC2818], Section 3).

HTTP/2根据HTTP/1.1中授权的定义来判断在给定响应中一个服务器是否已授权. 这由http URI的本地名字解析以及https中已授权的服务器的身份决定.

### 10.2. Cross-Protocol Attacks
### 10.2. 跨协议攻击

> In a cross-protocol attack, an attacker causes a client to initiate a
   transaction in one protocol toward a server that understands a
   different protocol.  An attacker might be able to cause the
   transaction to appear as a valid transaction in the second protocol.
   In combination with the capabilities of the web context, this can be
   used to interact with poorly protected servers in private networks.

在跨协议攻击下, 攻击者让一个客户端与服务器之间开始一个事务, 然而客户端使用了服务器不理解的协议进行交流. 攻击者可以构造这个事务, 使其在服务其使用的协议中是合法的. 结合web环境下的功能, 就可以在私有网络下与防护能力弱的服务器进行交互.

> Completing a TLS handshake with an ALPN identifier for HTTP/2 can be
   considered sufficient protection against cross-protocol attacks.
   ALPN provides a positive indication that a server is willing to
   proceed with HTTP/2, which prevents attacks on other TLS-based
   protocols.

在HTTP/2中用一个ALPN标记进行TLS握手会提供足够的保护以免遭受跨协议攻击. ALPN提供了一个正向指标表示服务器愿意使用HTTP/2, 这防止了对其他基于TLS的协议上的攻击.

> The encryption in TLS makes it difficult for attackers to control the
   data that could be used in a cross-protocol attack on a cleartext
   protocol.

TLS中的加密措施使得跨协议攻击者难以控制一个明文协议上的数据.

> The cleartext version of HTTP/2 has minimal protection against cross-
   protocol attacks.  The connection preface (Section 3.5) contains a
   string that is designed to confuse HTTP/1.1 servers, but no special
   protection is offered for other protocols.  A server that is willing
   to ignore parts of an HTTP/1.1 request containing an Upgrade header
   field in addition to the client connection preface could be exposed
   to a cross-protocol attack.

HTTP/2的明文版本具有抵御跨协议攻击的最小保护. 在3.5节的序言中有一段字符串用于混淆HTTP/1.1服务器, 但是对其他协议并没有提供特别的保护. 如果服务器希望忽略HTTP/1.1请求中包含除了客户端连接序言的upgrade头部字段的那部分, 可能会受跨协议攻击影响.

### 10.3. Intermediary Encapsulation Attacks
### 10.3. 伪装中介攻击

> The HTTP/2 header field encoding allows the expression of names that
   are not valid field names in the Internet Message Syntax used by
   HTTP/1.1.  Requests or responses containing invalid header field
   names MUST be treated as malformed (Section 8.1.2.6).  An
   intermediary therefore cannot translate an HTTP/2 request or response
   containing an invalid field name into an HTTP/1.1 message.

HTTP/2头部字段编码允许那些在HTTP/1.1所使用的互联网消息语法格式中并不合法的字段名.
对于那些包含非法头部字段名的请求或者响应必须视为格式错误(见第8.1.2.6小节描述).
因此中介者不可以把包含非法字段名的HTTP/2请求或响应转换成HTTP/1.1兼容的报文.

> Similarly, HTTP/2 allows header field values that are not valid.
   While most of the values that can be encoded will not alter header
   field parsing, carriage return (CR, ASCII 0xd), line feed (LF, ASCII
   0xa), and the zero character (NUL, ASCII 0x0) might be exploited by
   an attacker if they are translated verbatim.  Any request or response
   that contains a character not permitted in a header field value MUST
   be treated as malformed (Section 8.1.2.6).  Valid characters are
   defined by the "field-content" ABNF rule in Section 3.2 of [RFC7230].

同样, HTTP/2允许非法的头部字段值.
然而大多数可以被编码的值并不会影响头部字段的解析, 如果把他们一字不差进行解析,
那么像回车(CR, ASCII 0xd), 换行(LF, ASCII 0xa), 以及零字符(NUL, ASCII 0x0)就有可能会被攻击者利用.
如果请求或响应的头部字段值包含了非法的字符, 必须视之为格式错误(见第8.1.2.6小节描述).
"field-content" ABNF规则(RFC7230的第3.2节给出了描述)中定义了什么是**合法的字符**.

### 10.4. Cacheability of Pushed Responses
### 10.4. 推送响应的缓存能力

> Pushed responses do not have an explicit request from the client; the
   request is provided by the server in the PUSH_PROMISE frame.

推送响应并不会对应一个来自客户端的显式请求. 这个请求包含在服务器的PUSH_PROMISE帧中.

> Caching responses that are pushed is possible based on the guidance
   provided by the origin server in the Cache-Control header field.
   However, this can cause issues if a single server hosts more than one
   tenant.  For example, a server might offer multiple users each a
   small portion of its URI space.

可以依据原服务器在`Cache-Control`头部字段提供的指示来缓存推送的响应. 尽管如此, 在一个服务器上部署多个tenant时可能会出问题. 比如,
服务器可能把它的URI空间分配给多个用户各一小部分.

> Where multiple tenants share space on the same server, that server
   MUST ensure that tenants are not able to push representations of
   resources that they do not have authority over.  Failure to enforce
   this would allow a tenant to provide a representation that would be
   served out of cache, overriding the actual representation that the
   authoritative tenant provides.

多个tenant共享同一个服务器的空间的情况下, 服务器必须确保tenant不能推送他们无权管辖的资源. 如果不强制这么做, tenant将有可能提供缓存范围外的资源, 覆盖权威tenant提供的实际资源.

> Pushed responses for which an origin server is not authoritative (see
   Section 10.1) MUST NOT be used or cached.

对于不具权威性(见10.1节描述)的原服务器, 推送响应是不准被使用或缓存的.

### 10.5. Denial-of-Service Considerations
### 10.5. 对于拒绝服务攻击的考虑

> An HTTP/2 connection can demand a greater commitment of resources to
   operate than an HTTP/1.1 connection.  The use of header compression
   and flow control depend on a commitment of resources for storing a
   greater amount of state.  Settings for these features ensure that
   memory commitments for these features are strictly bounded.

相比HTTP/1.1连接, HTTP/2连接可以要求更大的资源操作能力.
头部压缩和流控在使用上都依赖于资源的承诺, 以存储更多的状态.
对于这些特性的设置严格限制了内存的使用.

> The number of PUSH_PROMISE frames is not constrained in the same
   fashion.  A client that accepts server push SHOULD limit the number
   of streams it allows to be in the "reserved (remote)" state.  An
   excessive number of server push streams can be treated as a stream
   error (Section 5.4.2) of type ENHANCE_YOUR_CALM.

PUSH_PROMISE帧的数量限制方式并不相同. 接受了服务器推送的客户端应该限制其"reserved (remote)"状态中允许使用的流的数量. 服务器推送流数量过多的话可能被视为一个类型为ENHANCE_YOUR_CALM的流错误(见5.4.2小节描述).

> Processing capacity cannot be guarded as effectively as state
   capacity.

`操作`不像`状态`那样可以有效进行管理.

> The SETTINGS frame can be abused to cause a peer to expend additional
   processing time.  This might be done by pointlessly changing SETTINGS
   parameters, setting multiple undefined parameters, or changing the
   same setting multiple times in the same frame.  WINDOW_UPDATE or
   PRIORITY frames can be abused to cause an unnecessary waste of
   resources.

SETTINGS帧的滥用可能会导致一个对等端增加额外的处理时间. 比如无意义的更改SETTINGS帧的参数, 设置多个未定义的参数, 或在同一个帧中多次修改同一个参数.
WINDOW_UPDATE或PRIORITY帧的滥用可能会导致资源的不必要浪费.

> Large numbers of small or empty frames can be abused to cause a peer
   to expend time processing frame headers.  Note, however, that some
   uses are entirely legitimate, such as the sending of an empty DATA or
   CONTINUATION frame at the end of a stream.

巨大数量的小型帧或空帧的滥用可能会导致一个对等端增加处理帧头部的时间. 注意, 尽管如此一些情况下这种使用是完全合理的, 比如在流结束时发送一个空的DATA帧或者CONTINUATION帧.

> Header compression also offers some opportunities to waste processing
   resources; see Section 7 of [COMPRESSION] for more details on
   potential abuses.

头部压缩过程也暴露了浪费处理资源的一些可乘之机. 详情参见[COMPRESSION]中的第七节描述的潜在滥用情况.

> Limits in SETTINGS parameters cannot be reduced instantaneously,
   which leaves an endpoint exposed to behavior from a peer that could
   exceed the new limits.  In particular, immediately after establishing
   a connection, limits set by a server are not known to clients and
   could be exceeded without being an obvious protocol violation.

SETTINGS帧中对参数的限制不会立即生效, 这让一个从对等端暴露自己行为的端系统能够超过新设置的最大限度.
特别是, 建立连接之后即刻起, 服务器设置的限制, 客户端无从得知, 所以除非有明显违反协议不然可能会超过这个限制.

> All these features -- i.e., SETTINGS changes, small frames, header
   compression -- have legitimate uses.  These features become a burden
   only when they are used unnecessarily or to excess.

所有这些特性, 例如, SETTINGS改变, 小型帧, 头部压缩, 都有其合理的使用场景. 也只有在不必要的使用或者滥用情况下这些特性才会成为带来负面影响.

> An endpoint that doesn't monitor this behavior exposes itself to a
   risk of denial-of-service attack.  Implementations SHOULD track the
   use of these features and set limits on their use.  An endpoint MAY
   treat activity that is suspicious as a connection error
   (Section 5.4.1) of type ENHANCE_YOUR_CALM.

一个端系统不会监视这一行为以免把自己置于DoS攻击的威胁之下.
协议的实现应该追踪这些特性的使用情况并在他们的使用上做限制. 一个端系统可以把可疑的活动视为一个类型为ENHANCE_YOUR_CALM的连接错误.
