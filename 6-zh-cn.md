# 6. Frame Definitions
# 6. 帧定义

> This specification defines a number of frame types, each identified by a unique 8-bit type code. Each frame type serves a distinct purpose in the establishment and management either of the connection as a whole or of individual streams.

这份文档定义了多种帧类型，每种都由一个唯一的8位类型码标识。每种帧类型都服务于建立和管理整个连接或独立的流方面的一个不同的目的。

> The transmission of specific frame types can alter the state of a connection. If endpoints fail to maintain a synchronized view of the connection state, successful communication within the connection will no longer be possible. Therefore, it is important that endpoints have a shared comprehension of how the state is affected by the use any given frame.

特定帧类型的传输可能改变连接的状态。如果终端不能维护连接状态视图的一致性，连接内成功的通信将是不可能的。因此，终端之间，关于特定帧的使用对状态所产生的影响具有相同的理解就变得非常重要。

## 6.1 DATA

> DATA frames (type=0x0) convey arbitrary, variable-length sequences of octets associated with a stream. One or more DATA frames are used, for instance, to carry HTTP request or response payloads.

DATA帧(type=0x0)传送与一个流关联的任意的，可变长度的字节序列。一个或多个DATA帧被用于，比如，携带HTTP请求或响应载荷。

> DATA frames MAY also contain padding. Padding can be added to DATA frames to obscure the size of messages. Padding is a security feature; see Section 10.7.

DATA帧也 **可以(MAY)** 包含填充字节。填充字节可以被加进DATA帧来掩盖消息的大小。填充字节是一个安全性的功能；参见 [Section 10.7](https://http2.github.io/http2-spec/#padding)。

```
 +---------------+
 |Pad Length? (8)|
 +---------------+-----------------------------------------------+
 |                            Data (*)                         ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```
> Figure 6: DATA Frame Payload

图 6：DATA帧载荷


> The DATA frame contains the following fields:

DATA帧包含如下的字段：

> Pad Length:
> An 8-bit field containing the length of the frame padding in units of octets. This field is conditional (as signified by a "?" in the diagram) and is only present if the PADDED flag is set.

**填充长度：**

一个8位的字段，包含了以字节为单位的帧的填充的长度。这个字段是有条件的(如图中的"?"所指的)，只有在PADDED标记设置时才出现。

> Data:
> Application data. The amount of data is the remainder of the frame payload after subtracting the length of the other fields that are present.

**数据：**

应用数据。数据的大小是帧载荷减去出现的其它字段的长度剩余的大小。

> Padding:
> Padding octets that contain no application semantic value. Padding octets MUST be set to zero when sending. A receiver is not obligated to verify padding but MAY treat non-zero padding as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

**填充：**

填充字节包含了无应用语义的值。当填充时填充字节 **必须(MUST)** 被设置为0。接收者没有义务去验证填充，而 **可以(MAY)** 将非零的填充当做一个类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> The DATA frame defines the following flags:

DATA帧定义了如下的标记：

> END_STREAM (0x1):
> When set, bit 0 indicates that this frame is the last that the endpoint will send for the identified stream. Setting this flag causes the stream to enter one of the "half-closed" states or the "closed" state (Section 5.1).

**END_STREAM (0x1)：**

当设置了这个标记时，位0表示这个帧是终端将为标识的流发送的最后一帧。设置这个标记使得流进入某种"half-closed"状态或"closed"状态([Section 5.1](https://http2.github.io/http2-spec/#StreamStates))。

> PADDED (0x8):
> When set, bit 3 indicates that the Pad Length field and any padding that it describes are present.

**PADDED (0x8):**

当设置了这个标记时，位3表示上面描述的 填充长度 字段及填充存在。

> DATA frames MUST be associated with a stream. If a DATA frame is received whose stream identifier field is 0x0, the recipient MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

DATA帧 **必须(MUST)** 与一个流关联。如果收到了一个流标识符为0x0的DATA帧，接收者 **必须(MUST)** 以一个类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))来响应。

> DATA frames are subject to flow control and can only be sent when a stream is in the "open" or "half-closed (remote)" state. The entire DATA frame payload is included in flow control, including the Pad Length and Padding fields if present. If a DATA frame is received whose stream is not in "open" or "half-closed (local)" state, the recipient MUST respond with a stream error (Section 5.4.2) of type STREAM_CLOSED.

DATA帧受控于flow control，而且只能在流处于"open"或"half-closed (remote)"状态时发送。整个的DATA帧载荷被包含在flow control中，可能包括 填充长度 和填充字段。如果收到DATA帧的流不处于"open"或"half-closed (remote)"状态，则接收者 **必须(MUST)** 以一个类型为[STREAM_CLOSED](https://http2.github.io/http2-spec/#STREAM_CLOSED)的流错误([Section 5.4.2](https://http2.github.io/http2-spec/#StreamErrorHandler))来响应。

> The total number of padding octets is determined by the value of the Pad Length field. If the length of the padding is the length of the frame payload or greater, the recipient MUST treat this as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

填充字节的总大小由填充长度字段决定。如果填充的长度是帧载荷的长度或更大，则接收者 **必须(MUST)** 将这作为一个类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))来处理应。

> Note: A frame can be increased in size by one octet by including a Pad Length field with a value of zero.

注意：一个帧可以通过包含一个值为零的填充长度字段来使帧长度只增加一个字节。

## 6.2 HEADERS

> The HEADERS frame (type=0x1) is used to open a stream (Section 5.1), and additionally carries a header block fragment. HEADERS frames can be sent on a stream in the "idle", "reserved (local)", "open", or "half-closed (remote)" state.

HEADERS帧(type=0x1)用于打开一个流([Section 5.1](https://http2.github.io/http2-spec/#StreamStates))，此外还携带一个首部块片段。HEADERS帧可以在一个"idle"，"reserved (local)"，"open"，或"half-closed (remote)"状态的流上发送。

```
 +---------------+
 |Pad Length? (8)|
 +-+-------------+-----------------------------------------------+
 |E|                 Stream Dependency? (31)                     |
 +-+-------------+-----------------------------------------------+
 |  Weight? (8)  |
 +-+-------------+-----------------------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```
> Figure 7: HEADERS Frame Payload

图 7: HEADERS帧载荷

> The HEADERS frame payload has the following fields:

HEADERS帧具有如下的字段：

> Pad Length:
> An 8-bit field containing the length of the frame padding in units of octets. This field is only present if the PADDED flag is set.

**填充长度：**

一个8位的字段，包含了以字节为单位的帧的填充的长度。只有在PADDED标记设置时这个字段才出现。

> E:
> A single-bit flag indicating that the stream dependency is exclusive (see Section 5.3). This field is only present if the PRIORITY flag is set.

**E:**

一个单独的位标记，指示了流依赖是独占的(参见[Section 5.3](https://http2.github.io/http2-spec/#StreamPriority))。这个字段只有在PRIORITY标记设置时才会出现。

> Stream Dependency:
> A 31-bit stream identifier for the stream that this stream depends on (see Section 5.3). This field is only present if the PRIORITY flag is set.

流依赖：一个31位的标识了这个流依赖的流的流标识符(参见[Section 5.3](https://http2.github.io/http2-spec/#StreamPriority))。这个字段只有在PRIORITY标记设置时才会出现。

> Weight:
> An unsigned 8-bit integer representing a priority weight for the stream (see Section 5.3). Add one to the value to obtain a weight between 1 and 256. This field is only present if the PRIORITY flag is set.

**权值：**

一个无符号8位整型值，表示流的优先级权值(see [Section 5.3](https://http2.github.io/http2-spec/#StreamPriority))。这个值的范围为1到256。这个字段只有在PRIORITY标记设置时才会出现。

> Header Block Fragment:
> A header block fragment (Section 4.3).

**首部块片段：**

一个首部块片段([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))。

> Padding:
> Padding octets.

**填充：**

填充字节。

> The HEADERS frame defines the following flags:

HEADERS帧定义了如下的标记：

> END_STREAM (0x1):
> When set, bit 0 indicates that the header block (Section 4.3) is the last that the endpoint will send for the identified stream.

**END_STREAM (0x1)：**

当设置时，位0表示这个首部块([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))是终端将会为标识的流发送的最后一个块。

> A HEADERS frame carries the END_STREAM flag that signals the end of a stream. However, a HEADERS frame with the END_STREAM flag set can be followed by CONTINUATION frames on the same stream. Logically, the CONTINUATION frames are part of the HEADERS frame.

HEADERS帧携带了 END_STREAM标记，表明了流的结束。然而，在相同的流上，一个设置了END_STREAM标记的HEADERS帧后面可以跟着[CONTINUATION](https://http2.github.io/http2-spec/#CONTINUATION)帧。逻辑上来说，着[CONTINUATION](https://http2.github.io/http2-spec/#CONTINUATION)帧是HEADERS帧的一部分。

> END_HEADERS (0x4):
> When set, bit 2 indicates that this frame contains an entire header block (Section 4.3) and is not followed by any CONTINUATION frames.

**END_HEADERS (0x4):**

当设置时，位2表示这个帧包含了这个首部块([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))，而且后面没有任何的[CONTINUATION](https://http2.github.io/http2-spec/#CONTINUATION)帧。

> A HEADERS frame without the END_HEADERS flag set MUST be followed by a CONTINUATION frame for the same stream. A receiver MUST treat the receipt of any other type of frame or a frame on a different stream as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

相同的流上，一个没有设置END_HEADERS标记的HEADERS帧后面 **必须(MUST)** 跟着一个[CONTINUATION](https://http2.github.io/http2-spec/#CONTINUATION)帧。接收者 **必须(MUST)** 将 接收到其它类型的帧，或在其它流上接收到了帧，当做是类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误。

> PADDED (0x8):
> When set, bit 3 indicates that the Pad Length field and any padding that it describes are present.

**PADDED (0x8):**

当设置时，位3表示将会有填充长度和它对应的填充出现。

> PRIORITY (0x20):
> When set, bit 5 indicates that the Exclusive Flag (E), Stream Dependency, and Weight fields are present; see Section 5.3.

**PRIORITY (0x20):**

当设置时，位5指明独占标记(E)，流依赖和权值字段将出现；参见[Section 5.3](https://http2.github.io/http2-spec/#StreamPriority)。

> The payload of a HEADERS frame contains a header block fragment (Section 4.3). A header block that does not fit within a HEADERS frame is continued in a CONTINUATION frame (Section 6.10).

HEADERS帧的载荷包含一个首部块片段([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))。一个首部块无法装进一个HEADERS帧的话，将通过CONTINUATION帧来继续发送([Section 6.10](https://http2.github.io/http2-spec/#CONTINUATION))。

> HEADERS frames MUST be associated with a stream. If a HEADERS frame is received whose stream identifier field is 0x0, the recipient MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

HEADERS帧 **必须(MUST)** 与一个流关联。如果接收到了一个流标识符字段为0x0的HEADERS帧，则接收者 **必须(MUST)** 响应一个类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> The HEADERS frame changes the connection state as described in Section 4.3.

HEADERS帧如[Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock)中所述的那样改变连接的状态。

> The HEADERS frame can include padding. Padding fields and flags are identical to those defined for DATA frames (Section 6.1). Padding that exceeds the size remaining for the header block fragment MUST be treated as a PROTOCOL_ERROR.

HEADERS帧可以包含填充。填充字段和标记与DATA帧([Section 6.1](https://http2.github.io/http2-spec/#DATA))中定义的一致。填充超出了首部块片段的剩余大小 **必须(MUST)** 被当做一个 [PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)。

> Prioritization information in a HEADERS frame is logically equivalent to a separate PRIORITY frame, but inclusion in HEADERS avoids the potential for churn in stream prioritization when new streams are created. Prioritization fields in HEADERS frames subsequent to the first on a stream reprioritize the stream (Section 5.3.3).

HEADERS帧中包含的优先级信息在逻辑上等于另一个 [PRIORITY](https://http2.github.io/http2-spec/#PRIORITY)帧，但是包含在HEADERS中可以避免在创建新流时流优先级潜在的扰动。一个流中第一个之后的HEADERS帧中的优先级字段改变流的优先级([Section 5.3.3](https://http2.github.io/http2-spec/#reprioritize))。

## 6.3 PRIORITY

> The PRIORITY frame (type=0x2) specifies the sender-advised priority of a stream (Section 5.3). It can be sent in any stream state, including idle or closed streams.

PRIORITY帧 (type=0x2) 描述了给发送者建议的一个流的优先级([Section 5.3](https://http2.github.io/http2-spec/#StreamPriority))。它可以在任何流状态下发送，包括idle和closed流。

```
 +-+-------------------------------------------------------------+
 |E|                  Stream Dependency (31)                     |
 +-+-------------+-----------------------------------------------+
 |   Weight (8)  |
 +-+-------------+
```
> Figure 8: PRIORITY Frame Payload

图 8: PRIORITY帧载荷

> The payload of a PRIORITY frame contains the following fields:

PRIORITY帧包含如下的字段：

> E:
> A single-bit flag indicating that the stream dependency is exclusive (see Section 5.3).

**E:** 一个单独的位标记指明流依赖是独占的(参见[Section 5.3](https://http2.github.io/http2-spec/#StreamPriority))。

> Stream Dependency:
> A 31-bit stream identifier for the stream that this stream depends on (see Section 5.3).

**流依赖：**
一个31位的流标识符，指明了这个流依赖的流(参见 [Section 5.3](https://http2.github.io/http2-spec/#StreamPriority))。

> Weight:
> An unsigned 8-bit integer representing a priority weight for the stream (see Section 5.3). Add one to the value to obtain a weight between 1 and 256.

**权值：**
一个无符号8位整型值，表示流的优先级权值(参见 [Section 5.3](https://http2.github.io/http2-spec/#StreamPriority))。该值的取值范围为1到256。

> The PRIORITY frame does not define any flags.

PRIORITY帧不定义任何标记。

> The PRIORITY frame always identifies a stream. If a PRIORITY frame is received with a stream identifier of 0x0, the recipient MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

PRIORITY帧总是标识一个流。如果接收到了一个流标识符为0x0的PRIORITY帧，则接收者 **必须(MUST)** 响应一个类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> The PRIORITY frame can be sent on a stream in any state, though it cannot be sent between consecutive frames that comprise a single header block (Section 4.3). Note that this frame could arrive after processing or frame sending has completed, which would cause it to have no effect on the identified stream. For a stream that is in the "half-closed (remote)" or "closed" state, this frame can only affect processing of the identified stream and its dependent streams; it does not affect frame transmission on that stream.

可以在任何状态的流上发送RIORITY帧，尽管不能在包含了一个单独的首部块([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))的连续两帧之间发送。注意，这个帧可能在处理或帧发送已经完成时到达，这将不对标识的流产生影响。对于处在"half-closed (remote)"或"closed"状态的流，这个帧只影响标识的流和依赖于它的流的处理；它不影响那些流的帧传输。

> The PRIORITY frame can be sent for a stream in the "idle" or "closed" state. This allows for the reprioritization of a group of dependent streams by altering the priority of an unused or closed parent stream.

可以为处于"idle"或"closed"状态的流发送PRIORITY帧。这允许通过改变未使用或已关闭的父流的优先级来改变一组依赖流的优先级。

> A PRIORITY frame with a length other than 5 octets MUST be treated as a stream error (Section 5.4.2) of type FRAME_SIZE_ERROR.

PRIORITY帧的长度不是5个字节的话， **必须(MUST)** 被当做一个类型为[FRAME_SIZE_ERROR](https://http2.github.io/http2-spec/#FRAME_SIZE_ERROR)的流错误([Section 5.4.2](https://http2.github.io/http2-spec/#StreamErrorHandler))。

## 6.4 RST_STREAM

> The RST_STREAM frame (type=0x3) allows for immediate termination of a stream. RST_STREAM is sent to request cancellation of a stream or to indicate that an error condition has occurred.

RST_STREAM帧 (type=0x3)可以用于立即终止一个流。发送RST_STREAM来请求取消一个流，或者指明发生了一个错误状态。

```
 +---------------------------------------------------------------+
 |                        Error Code (32)                        |
 +---------------------------------------------------------------+
```
> Figure 9: RST_STREAM Frame Payload

Figure 9: RST_STREAM帧载荷

> The RST_STREAM frame contains a single unsigned, 32-bit integer identifying the error code (Section 7). The error code indicates why the stream is being terminated.

RST_STREAM帧包含了一个单独的无符号32位整型值的错误码([Section 7](https://http2.github.io/http2-spec/#ErrorCodes))。错误码指明了为什么要终止流。

> The RST_STREAM frame does not define any flags.

RST_STREAM帧不定义任何标记。

> The RST_STREAM frame fully terminates the referenced stream and causes it to enter the "closed" state. After receiving a RST_STREAM on a stream, the receiver MUST NOT send additional frames for that stream, with the exception of PRIORITY. However, after sending the RST_STREAM, the sending endpoint MUST be prepared to receive and process additional frames sent on the stream that might have been sent by the peer prior to the arrival of the RST_STREAM.

RST_STREAM帧完全终止引用的流，并使它进入"closed"状态。在一个流中收到一个RST_STREAM之后，接收者 **一定不能(MUST NOT)** 再为那个流发送额外的帧，[PRIORITY](https://http2.github.io/http2-spec/#PRIORITY)是例外。然而，在发送RST_STREAM之后，发送端 **必须(MUST)** 准备接收和处理额外的，可能由对端在RST_STREAM到达之前在那个流上发送的帧。

> RST_STREAM frames MUST be associated with a stream. If a RST_STREAM frame is received with a stream identifier of 0x0, the recipient MUST treat this as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

RST_STREAM帧 **必须(MUST)** 关联一个流。如果RST_STREAM帧的流标识符是0x0，则接收者 **必须(MUST)** 将其作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))处理。

> RST_STREAM frames MUST NOT be sent for a stream in the "idle" state. If a RST_STREAM frame identifying an idle stream is received, the recipient MUST treat this as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

RST_STREAM帧 **一定不能(MUST NOT)** 为"idle"状态的流而发送。如果接收了一个RST_STREAM帧，而它标识了一个idle流，则接收者 **必须(MUST)** 将其作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))处理。

> A RST_STREAM frame with a length other than 4 octets MUST be treated as a connection error (Section 5.4.1) of type FRAME_SIZE_ERROR.

RST_STREAM帧的长度不是4字节的话， **必须(MUST)** 被作为一个类型是[FRAME_SIZE_ERROR](https://http2.github.io/http2-spec/#FRAME_SIZE_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))处理。

## 6.5 SETTINGS

> The SETTINGS frame (type=0x4) conveys configuration parameters that affect how endpoints communicate, such as preferences and constraints on peer behavior. The SETTINGS frame is also used to acknowledge the receipt of those parameters. Individually, a SETTINGS parameter can also be referred to as a "setting".

SETTINGS帧 (type=0x4) 携带影响端点间如何通信的配置参数，比如关于对端行为的首选项和限制。SETTINGS帧也用于确认接收到了那些参数。个别地，一个SETTINGS参数也可以被称为一个"setting"。

> SETTINGS parameters are not negotiated; they describe characteristics of the sending peer, which are used by the receiving peer. Different values for the same parameter can be advertised by each peer. For example, a client might set a high initial flow-control window, whereas a server might set a lower value to conserve resources.

不协商SETTINGS参数；它们描述了发送端的特性，而由接收端使用。端点之间对于相同参数可以广告不同的值。比如，一个客户端可以设置一个较大的初始flow-control窗口，然而服务器可以设置一个小的值来保留资源。

> A SETTINGS frame MUST be sent by both endpoints at the start of a connection and MAY be sent at any other time by either endpoint over the lifetime of the connection. Implementations MUST support all of the parameters defined by this specification.

SETTINGS帧 **必须(MUST)** 在连接开始时，两端都发送，而在连接整个生命期中其它的任何时间点，其中的一个端点 **可以(MAY)** 发送。实现 **必须(MUST)** 支持这份规范定义的所有参数。

> Each parameter in a SETTINGS frame replaces any existing value for that parameter. Parameters are processed in the order in which they appear, and a receiver of a SETTINGS frame does not need to maintain any state other than the current value of its parameters. Therefore, the value of a SETTINGS parameter is the last value that is seen by a receiver.

SETTINGS帧中的每个参数替换那个参数既有的值。参数以它们出现的顺序处理，SETTINGS帧的接收者不需要维护额外的状态，除了参数的当前值。因此，一个SETTINGS参数的值是接收者收到的最后的值。

> SETTINGS parameters are acknowledged by the receiving peer. To enable this, the SETTINGS frame defines the following flag:

SETTINGS参数由接收端作确认。为了启用这一点，SETTINGS帧定义了如下的标记：

> ACK (0x1):
> When set, bit 0 indicates that this frame acknowledges receipt and application of the peer's SETTINGS frame. When this bit is set, the payload of the SETTINGS frame MUST be empty. Receipt of a SETTINGS frame with the ACK flag set and a length field value other than 0 MUST be treated as a connection error (Section 5.4.1) of type FRAME_SIZE_ERROR. For more information, see Section 6.5.3 ("Settings Synchronization").

**ACK (0x1):** 设置时，位0指示了这个帧用于确认对端的SETTINGS帧的接收和应用。当设置了这个位时，SETTINGS帧的载荷必须是空的。接收到一个设置了ACK标记的SETTINGS帧，而长度字段的值不是0，这 **必须(MUST)** 被当做一个类型为[FRAME_SIZE_ERROR](https://http2.github.io/http2-spec/#FRAME_SIZE_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。要获得更多信息，请参考[Section 6.5.3](https://http2.github.io/http2-spec/#SettingsSync) ("[Settings Synchronization](https://http2.github.io/http2-spec/#SettingsSync)")。

> SETTINGS frames always apply to a connection, never a single stream. The stream identifier for a SETTINGS frame MUST be zero (0x0). If an endpoint receives a SETTINGS frame whose stream identifier field is anything other than 0x0, the endpoint MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

SETTINGS帧总是应用于一个连接，而不是一个单独的流。SETTINGS帧的流标识符 **必须(MUST)** 是零(0x0)。如果一个端点收到了一个流标识符字段不是0的SETTINGS帧，则 **必须(MUST)** 以一个类型为 [PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR) 的连接错误 ([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler)) 来响应。

> The SETTINGS frame affects connection state. A badly formed or incomplete SETTINGS frame MUST be treated as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

SETTINGS帧影响连接的状态。一个格式错误或不完整的SETTINGS帧 **必须(MUST)** 被当做一个类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> A SETTINGS frame with a length other than a multiple of 6 octets MUST be treated as a connection error (Section 5.4.1) of type FRAME_SIZE_ERROR.

SETTINGS帧的长度如果不是6字节的整数倍的话，必须被作为一个类型是FRAME_SIZE_ERROR](https://http2.github.io/http2-spec/#FRAME_SIZE_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

6.5.1 SETTINGS Format

### 6.5.1 SETTINGS格式

> The payload of a SETTINGS frame consists of zero or more parameters, each consisting of an unsigned 16-bit setting identifier and an unsigned 32-bit value.

SETTINGS帧的载荷由零个或多个参数组成，每个参数由一个无符号16位的设置标识符和一个无符号的32位值组成。

```
 +-------------------------------+
 |       Identifier (16)         |
 +-------------------------------+-------------------------------+
 |                        Value (32)                             |
 +---------------------------------------------------------------+
```
> Figure 10: Setting Format

图 10: 设置项的格式

6.5.2 Defined SETTINGS Parameters

### 6.5.2 已定义的SETTINGS参数

> The following parameters are defined:

已定义了如下的参数：

> SETTINGS_HEADER_TABLE_SIZE (0x1):
> Allows the sender to inform the remote endpoint of the maximum size of the header compression table used to decode header blocks, in octets. The encoder can select any size equal to or less than this value by using signaling specific to the header compression format inside a header block (see [COMPRESSION]). The initial value is 4,096 octets.

**SETTINGS_HEADER_TABLE_SIZE (0x1):**
允许发送者通知远端，用于解码首部块的首部压缩表的最大大小，以字节位单位。编码器可以可以选择任何等于或小于这个值的大小，通过使用首部块内信号特有的首部压缩格式(参考[[COMPRESSION]
](https://http2.github.io/http2-spec/#COMPRESSION))。初始值是4,096字节。

> SETTINGS_ENABLE_PUSH (0x2):
> This setting can be used to disable server push (Section 8.2). An endpoint MUST NOT send a PUSH_PROMISE frame if it receives this parameter set to a value of 0. An endpoint that has both set this parameter to 0 and had it acknowledged MUST treat the receipt of a PUSH_PROMISE frame as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

**SETTINGS_ENABLE_PUSH (0x2):**
这个设置项可被用于禁用服务端推送([Section 8.2](https://http2.github.io/http2-spec/#PushResources))。一个终端如果收到了这个设置项，且值为0，则它 **一定不能(MUST NOT)** 发送[PUSH_PROMISE](https://http2.github.io/http2-spec/#PUSH_PROMISE)帧。一个已经将这个参数设置为了0，且已经收到了对这个设置项的确认的终端，则在收到一个[PUSH_PROMISE](https://http2.github.io/http2-spec/#PUSH_PROMISE)帧时， **必须(MUST)** 将其作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> The initial value is 1, which indicates that server push is permitted. Any value other than 0 or 1 MUST be treated as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

初始值是1，这表示服务端推送是允许的。任何0或1之外的值 **必须(MUST)** 被作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> SETTINGS_MAX_CONCURRENT_STREAMS (0x3):
> Indicates the maximum number of concurrent streams that the sender will allow. This limit is directional: it applies to the number of streams that the sender permits the receiver to create. Initially, there is no limit to this value. It is recommended that this value be no smaller than 100, so as to not unnecessarily limit parallelism.

**SETTINGS_MAX_CONCURRENT_STREAMS (0x3):**
指明了发送者允许的最大的并发流个数。这个限制是有方向的：它应用于发送者允许接收者创建的流的个数。初始时，这个值没有限制。建议这个值不要小于100，以便于不要不必要地限制了并发性。

> A value of 0 for SETTINGS_MAX_CONCURRENT_STREAMS SHOULD NOT be treated as special by endpoints. A zero value does prevent the creation of new streams; however, this can also happen for any limit that is exhausted with active streams. Servers SHOULD only set a zero value for short durations; if a server does not wish to accept requests, closing the connection is more appropriate.

SETTINGS_MAX_CONCURRENT_STREAMS被设置为0 **不应该(SHULD NOT)** 被终端特殊对待。值为0确实阻止创建新的流；然而，这也可能发生在活跃的流超出了限制的时候。服务器 **应该(SHOULD)** 只短暂地将这个值设为0；如果一个服务器不希望接受请求，关闭连接更合适。

> SETTINGS_INITIAL_WINDOW_SIZE (0x4):
> Indicates the sender's initial window size (in octets) for stream-level flow control. The initial value is 216-1 (65,535) octets.

**SETTINGS_INITIAL_WINDOW_SIZE (0x4):**
指明了发送者stream-level flow control的初始窗口大小(以字节为单位)。初始值为2^16 - 1 (65,535)字节。

> This setting affects the window size of all streams (see Section 6.9.2).

这个设置项影响所有流的窗口大小(参见[Section 6.9.2](https://http2.github.io/http2-spec/#InitialWindowSize))。

> Values above the maximum flow-control window size of 231-1 MUST be treated as a connection error (Section 5.4.1) of type FLOW_CONTROL_ERROR.

值大于2^31-1的最大flow-control窗口大小 **必须(MUST)** 被作为一个类型是[FLOW_CONTROL_ERROR](https://http2.github.io/http2-spec/#FLOW_CONTROL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> SETTINGS_MAX_FRAME_SIZE (0x5):
> Indicates the size of the largest frame payload that the sender is willing to receive, in octets.

**SETTINGS_MAX_FRAME_SIZE (0x5):**
指明了发送者期望接收的最大的帧载荷大小，以字节为单位。

> The initial value is 214 (16,384) octets. The value advertised by an endpoint MUST be between this initial value and the maximum allowed frame size (224-1 or 16,777,215 octets), inclusive. Values outside this range MUST be treated as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

初始值是2^14 (16,384)。终端广告的值 **必须(MUST)** 介于初始值和允许的最大帧大小(2^24-1 or 16,777,215 字节)之间，包含。这个范围之外的值 **必须(MUST)** 被作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> SETTINGS_MAX_HEADER_LIST_SIZE (0x6):
> This advisory setting informs a peer of the maximum size of header list that the sender is prepared to accept, in octets. The value is based on the uncompressed size of header fields, including the length of the name and value in octets plus an overhead of 32 octets for each header field.

**SETTINGS_MAX_HEADER_LIST_SIZE (0x6):**
这个建议性的设置通知对端发送者准备接受的首部列表的最大大小，以字节为单位。这个值是基于首部字段未压缩的大小来计算的，包括名字和值以字节为单位的长度，再为每个首部字段加上32字节。

> For any given request, a lower limit than what is advertised MAY be enforced. The initial value of this setting is unlimited.

对于任何给定的请求，**可以(MAY)** 实施小于广告的限制的值。这个设置项的初始值没有限制。

> An endpoint that receives a SETTINGS frame with any unknown or unsupported identifier MUST ignore that setting.

一个终端接收了一个SETTINGS帧，其中未知的或不支持的标识符的设置项 **必须(MUST)** 被忽略。

6.5.3 Settings Synchronization

### 6.5.3 设置同步

> Most values in SETTINGS benefit from or require an understanding of when the peer has received and applied the changed parameter values. In order to provide such synchronization timepoints, the recipient of a SETTINGS frame in which the ACK flag is not set MUST apply the updated parameters as soon as possible upon receipt.

SETTINGS中的大多数值受益于或需要知道对端在何时接收并应用改变的参数值。为了提供这种同步时间点，一个ACK标记没有设置的SETTINGS帧的接收者 **必须(MUST)** 一收到帧就尽可能快地应用更新后的参数。

> The values in the SETTINGS frame MUST be processed in the order they appear, with no other frame processing between values. Unsupported parameters MUST be ignored. Once all values have been processed, the recipient MUST immediately emit a SETTINGS frame with the ACK flag set. Upon receiving a SETTINGS frame with the ACK flag set, the sender of the altered parameters can rely on the setting having been applied.

SETTINGS帧中的参数 **必须(MUST)** 以它们出现的顺序处理，值之间没有对其它帧的处理。不支持的参数 **必须(MUST)** 被忽略。一旦处理了所有值，则接收者 **必须(MUST)** 立即发送一个设置了ACK标记的SETTINGS帧。一旦接收到设置了ACK标记的SETTINGS帧，改变了参数的发送者可以依赖于已经应用的设置了。

> If the sender of a SETTINGS frame does not receive an acknowledgement within a reasonable amount of time, it MAY issue a connection error (Section 5.4.1) of type SETTINGS_TIMEOUT.

如果SETTINGS帧的发送者没有在合理的时间内收到确认，它 **可以(MAY)** 产生一个类型为[SETTINGS_TIMEOUT](https://http2.github.io/http2-spec/#SETTINGS_TIMEOUT)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

## 6.6 PUSH_PROMISE

> The PUSH_PROMISE frame (type=0x5) is used to notify the peer endpoint in advance of streams the sender intends to initiate. The PUSH_PROMISE frame includes the unsigned 31-bit identifier of the stream the endpoint plans to create along with a set of headers that provide additional context for the stream. Section 8.2 contains a thorough description of the use of PUSH_PROMISE frames.

PUSH_PROMISE帧 (type=0x5)用于通知对端发送方想要启动一个流。PUSH_PROMISE帧包含终端计划创建的流的无符号31位标识符，及为流提供额外上下为的一系列的首部。[Section 8.2](https://http2.github.io/http2-spec/#PushResources)包含对于PUSH_PROMISE帧的使用的一个详尽的描述。

```
 +---------------+
 |Pad Length? (8)|
 +-+-------------+-----------------------------------------------+
 |R|                  Promised Stream ID (31)                    |
 +-+-----------------------------+-------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```
> Figure 11: PUSH_PROMISE Payload Format

图 11: PUSH_PROMISE载荷格式

> The PUSH_PROMISE frame payload has the following fields:

PUSH_PROMISE帧载荷具有如下的字段：

> Pad Length:
> An 8-bit field containing the length of the frame padding in units of octets. This field is only present if the PADDED flag is set.

**填充长度：**
一个8位的字段，包含了以字节为单位的帧填充的长度。只有在PADDED标记设置时这个字段才会出现。

> R:
> A single reserved bit.

**R:**
一个保留位。

> Promised Stream ID:
> An unsigned 31-bit integer that identifies the stream that is reserved by the PUSH_PROMISE. The promised stream identifier MUST be a valid choice for the next stream sent by the sender (see "new stream identifier" in Section 5.1.1).

**约定流ID:**
一个无符号31位整数，标识了PUSH_PROMISE保留的流。约定流标识符 **必须(MUST)** 是发送者后续要发送的流的一个有效的流标识符（参见 [Section 5.1.1](https://http2.github.io/http2-spec/#StreamIdentifiers)中的"新的流标识符"）。

> Header Block Fragment:
> A header block fragment (Section 4.3) containing request header fields.

**首部块片段：**
一个首部块片段([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))包含了请求的首部字段。

> Padding:
> Padding octets.

**填充：** 填充字节。

> The PUSH_PROMISE frame defines the following flags:

PUSH_PROMISE帧定义了如下的标记：

> END_HEADERS (0x4):
> When set, bit 2 indicates that this frame contains an entire header block (Section 4.3) and is not followed by any CONTINUATION frames.

**END_HEADERS (0x4):**
设置时，位2表示这个帧包含一个完整的首部块([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))，而且后面没有[CONTINUATION](https://http2.github.io/http2-spec/#CONTINUATION)帧。

> A PUSH_PROMISE frame without the END_HEADERS flag set MUST be followed by a CONTINUATION frame for the same stream. A receiver MUST treat the receipt of any other type of frame or a frame on a different stream as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

PUSH_PROMISE帧的END_HEADERS标记没有设置的话，它的后面 **必须(MUST)** 要有相同流的CONTINUATION帧。接收者 **必须(MUST)** 将收到其它类型的帧，或在一个不同的流上收到帧，作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> PADDED (0x8):
> hen set, bit 3 indicates that the Pad Length field and any padding that it describes are present.

**PADDED (0x8):**
设置时，位2表示 **填充长度** 字段和它所对应的填充将会出现。

> PUSH_PROMISE frames MUST only be sent on a peer-initiated stream that is in either the "open" or "half-closed (remote)" state. The stream identifier of a PUSH_PROMISE frame indicates the stream it is associated with. If the stream identifier field specifies the value 0x0, a recipient MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

PUSH_PROMISE帧 **必须(MUST)** 只在对端初始化的处于"open"或 "half-closed (remote)"状态的流上发送。PUSH_PROMISE帧的流标识符指明了它关联的流。如果流标识符字段是0x0，则接收者 **必须(MUST)** 响应一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> Promised streams are not required to be used in the order they are promised. The PUSH_PROMISE only reserves stream identifiers for later use.

约定的流不需要以它们约定的顺序使用。PUSH_PROMISE只为后面的使用保留了流标识符。

> PUSH_PROMISE MUST NOT be sent if the SETTINGS_ENABLE_PUSH setting of the peer endpoint is set to 0. An endpoint that has set this setting and has received acknowledgement MUST treat the receipt of a PUSH_PROMISE frame as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

如果对端将[SETTINGS_ENABLE_PUSH](https://http2.github.io/http2-spec/#SETTINGS_ENABLE_PUSH)设置为了0，则 **一定不能(MUST NOT)** 发送PUSH_PROMISE。已经设置了这个设定项且已经收到了确认的终端，收到了一个PUSH_PROMISE帧，则它 **必须(MUST)** 将这作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> Recipients of PUSH_PROMISE frames can choose to reject promised streams by returning a RST_STREAM referencing the promised stream identifier back to the sender of the PUSH_PROMISE.

PUSH_PROMISE的接收者可以通过返回一个引用了约定的流标识符的[RST_STREAM](https://http2.github.io/http2-spec/#RST_STREAM)给PUSH_PROMISE的发送者来拒绝约定的流。

> A PUSH_PROMISE frame modifies the connection state in two ways. First, the inclusion of a header block (Section 4.3) potentially modifies the state maintained for header compression. Second, PUSH_PROMISE also reserves a stream for later use, causing the promised stream to enter the "reserved" state. A sender MUST NOT send a PUSH_PROMISE on a stream unless that stream is either "open" or "half-closed (remote)"; the sender MUST ensure that the promised stream is a valid choice for a new stream identifier (Section 5.1.1) (that is, the promised stream MUST be in the "idle" state).

PUSH_PROMISE帧以两种方式改变连接状态。首先，它包含的首部块([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))潜在地改变为首部压缩维护的状态。其次，PUSH_PROMISE也为后续的使用保留一个流，这使得约定的流进入"reserved"状态。除非流处于"open"或"half-closed (remote)"状态，否则发送者 **一定不能(MUST NOT)** 在那个流上发送PUSH_PROMISE；发送者 **必须(MUST)** 确保约定的流是一个新的流标识符([Section 5.1.1](https://http2.github.io/http2-spec/#StreamIdentifiers))的有效的选择(即，约定的流 **必须(MUST)** 处于"idle"状态)。

> Since PUSH_PROMISE reserves a stream, ignoring a PUSH_PROMISE frame causes the stream state to become indeterminate. A receiver MUST treat the receipt of a PUSH_PROMISE on a stream that is neither "open" nor "half-closed (local)" as a connection error (Section 5.4.1) of type PROTOCOL_ERROR. However, an endpoint that has sent RST_STREAM on the associated stream MUST handle PUSH_PROMISE frames that might have been created before the RST_STREAM frame is received and processed.

由于PUSH_PROMISE保留一个流，则忽略一个PUSH_PROMISE帧导致流的状态变为不确定的。接收者必须将不处于"open"或"half-closed (local)"状态的流上接收到的PUSH_PROMISE作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。然而，已经在关联的流上发送了[RST_STREAM](https://http2.github.io/http2-spec/#RST_STREAM)的终端， **必须(MUST)** 处理可能在RST_STREAM](https://http2.github.io/http2-spec/#RST_STREAM)帧接收和处理之前创建的PUSH_PROMISE帧。

> A receiver MUST treat the receipt of a PUSH_PROMISE that promises an illegal stream identifier (Section 5.1.1) as a connection error (Section 5.4.1) of type PROTOCOL_ERROR. Note that an illegal stream identifier is an identifier for a stream that is not currently in the "idle" state.

接收者接收了一个PUSH_PROMISE，但其约定了一个非法的流标识符([Section 5.1.1](https://http2.github.io/http2-spec/#StreamIdentifiers))，则接收者必须将这作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。注意，非法的流标识符是当前不处于"idle"状态的流的标识符。

> The PUSH_PROMISE frame can include padding. Padding fields and flags are identical to those defined for DATA frames (Section 6.1).

PUSH_PROMISE帧可能包含填充。填充字段和标记与DATA帧一节([Section 6.1](https://http2.github.io/http2-spec/#DATA))中定义的一致。

## 6.7 PING

> The PING frame (type=0x6) is a mechanism for measuring a minimal round-trip time from the sender, as well as determining whether an idle connection is still functional. PING frames can be sent from any endpoint.

PING帧 (type=0x6) 是一种测量自发送者开始的最小往返时间的机制，也用于测定一个idle连接是否仍然有效。PING帧可以自连接的任何一端发送。

```
 +---------------------------------------------------------------+
 |                                                               |
 |                      Opaque Data (64)                         |
 |                                                               |
 +---------------------------------------------------------------+
```
> Figure 12: PING Payload Format

图 12: PING 载荷格式

> In addition to the frame header, PING frames MUST contain 8 octets of opaque data in the payload. A sender can include any value it chooses and use those octets in any fashion.

除了帧首部，PING帧 **必须(MUST)** 在载荷中包含8字节的不透明数据。发送者可以包含它选择的任何值，并且可以以任何方式使用它们。

> Receivers of a PING frame that does not include an ACK flag MUST send a PING frame with the ACK flag set in response, with an identical payload. PING responses SHOULD be given higher priority than any other frame.

没有包含ACK标记的PING帧的接收者 **必须(MUST)** 必须在响应中发送一个设置了ACK标记的PING帧，且载荷要一致。**应该(SHOULD)** 给PING响应一个相对于其它帧更高的优先级

> The PING frame defines the following flags:

PING帧定义了如下的标记：

> ACK (0x1):
> When set, bit 0 indicates that this PING frame is a PING response. An endpoint MUST set this flag in PING responses. An endpoint MUST NOT respond to PING frames containing this flag.

**ACK (0x1):**
当设置时，位0指明了这个PING帧是一个PING响应。终端在PING响应中 **必须(MUST)** 设置这个标记。终端 **一定不能(MUST NOT)** 响应包含了这个标记的PING帧。

> PING frames are not associated with any individual stream. If a PING frame is received with a stream identifier field value other than 0x0, the recipient MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

PING帧不与任何独立的流关联。如果接收到一个流标识符字段不是0的PING帧，接收者必须以一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))来响应。

> Receipt of a PING frame with a length field value other than 8 MUST be treated as a connection error (Section 5.4.1) of type FRAME_SIZE_ERROR.

接收到一个PING帧，其长度字段的值不是8，则 **必须(MUST)** 被作为一个类型是[FRAME_SIZE_ERROR](https://http2.github.io/http2-spec/#FRAME_SIZE_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

## 6.8 GOAWAY

> The GOAWAY frame (type=0x7) is used to initiate shutdown of a connection or to signal serious error conditions. GOAWAY allows an endpoint to gracefully stop accepting new streams while still finishing processing of previously established streams. This enables administrative actions, like server maintenance.

GOAWAY帧(type=0x7)用于初始化一个连接的关闭，或通知严重的错误条件。GOAWAY允许一个终端在处理之前建立的流的同时优雅地停止接收新的流。这使管理操作称为可能，如服务器维护。

> There is an inherent race condition between an endpoint starting new streams and the remote sending a GOAWAY frame. To deal with this case, the GOAWAY contains the stream identifier of the last peer-initiated stream that was or might be processed on the sending endpoint in this connection. For instance, if the server sends a GOAWAY frame, the identified stream is the highest-numbered stream initiated by the client.

在一个终端启动新流和远端发送一个GOAWAY帧之间有内在的竞态条件。要处理这种情况，GOAWAY包含这个连接中，发送端处理或可能会处理的由对端初始化的最后的流的流标识符。比如，如果服务器发送了一个GOAWAY帧，则标识的流是客户端初始化的流标识符最大的流。

> Once sent, the sender will ignore frames sent on streams initiated by the receiver if the stream has an identifier higher than the included last stream identifier. Receivers of a GOAWAY frame MUST NOT open additional streams on the connection, although a new connection can be established for new streams.

GOAWAY帧一旦发送，则发送者将忽略由接收者初始化的，流标识符大于帧中包含的流标识符的流上发送的帧。GOAWAY的接收者 **一定不能(MUST NOT)** 在连接上打开额外的流，尽管可以为新的流创建一个新的连接。

> If the receiver of the GOAWAY has sent data on streams with a higher stream identifier than what is indicated in the GOAWAY frame, those streams are not or will not be processed. The receiver of the GOAWAY frame can treat the streams as though they had never been created at all, thereby allowing those streams to be retried later on a new connection.

如果GOAWAY的接收者已经在流标识符比GOAWAY帧中指明的更大的流上发送了数据，那些流将不会被处理。GOAWAY帧的接收者可以像它们从来没有创建一样处理，因此允许那些流稍后在一个新的连接上重试。

> Endpoints SHOULD always send a GOAWAY frame before closing a connection so that the remote peer can know whether a stream has been partially processed or not. For example, if an HTTP client sends a POST at the same time that a server closes a connection, the client cannot know if the server started to process that POST request if the server does not send a GOAWAY frame to indicate what streams it might have acted on.

终端 **应该(SHOULD)** 总是在关闭连接之前发送一个GOAWAY帧，以使远处的对端可以知道一个流是否被部分处理了，还是么有。比如，如果一个HTTP客户端在服务器关闭一个连接的同时发送了一个POST，如果服务器不发送GOAWAY帧指明它已经处理的流的话，客户端无法知道服务器是否启动了对那个POST请求的处理。

> An endpoint might choose to close a connection without sending a GOAWAY for misbehaving peers.

一个终端可能选择在不发送GOAWAY的情况下关闭连接来使对端行为不当。

> A GOAWAY frame might not immediately precede closing of the connection; a receiver of a GOAWAY that has no more use for the connection SHOULD still send a GOAWAY frame before terminating the connection.

一个GOAWAY帧可以不紧贴着连接的关闭；不再使用连接的GOAWAY的接收者依然 **应该(SHOULD)** 在终止连接之前发送GOAWAY帧。

```
 +-+-------------------------------------------------------------+
 |R|                  Last-Stream-ID (31)                        |
 +-+-------------------------------------------------------------+
 |                      Error Code (32)                          |
 +---------------------------------------------------------------+
 |                  Additional Debug Data (*)                    |
 +---------------------------------------------------------------+
```
> Figure 13: GOAWAY Payload Format

图 13: GOAWAY载荷格式

> The GOAWAY frame does not define any flags.

GOAWAY帧不定义任何标记。

> The GOAWAY frame applies to the connection, not a specific stream. An endpoint MUST treat a GOAWAY frame with a stream identifier other than 0x0 as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

GOAWAY帧应用于连接，而不是特定的流。一个终端 **必须(MUST)** 将一个流标识符不是0x0的GOAWAY帧作为一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> The last stream identifier in the GOAWAY frame contains the highest-numbered stream identifier for which the sender of the GOAWAY frame might have taken some action on or might yet take action on. All streams up to and including the identified stream might have been processed in some way. The last stream identifier can be set to 0 if no streams were processed.

GOAWAY帧中的最后的流标识符包含了GOAWAY帧的发送者可能已经处理或可能会处理的最大的流标识符。所有流标识符小于等于标识的流的流可能已经以某种方式被处理了。如果没有流被处理的话，最后的流标识符可以被设置为0.

> Note: In this context, "processed" means that some data from the stream was passed to some higher layer of software that might have taken some action as a result.

**注意:** 在这个上下文中，"处理"意味着来自于流的一些数据被传给了一些更高层级的软件，那些软件可能已经做了一些处理。

> If a connection terminates without a GOAWAY frame, the last stream identifier is effectively the highest possible stream identifier.

如果流终止了，而没有GOAWAY帧，则最后的流标识符实际上是最大的可能流标识符。

> On streams with lower- or equal-numbered identifiers that were not closed completely prior to the connection being closed, reattempting requests, transactions, or any protocol activity is not possible, with the exception of idempotent actions like HTTP GET, PUT, or DELETE. Any protocol activity that uses higher-numbered streams can be safely retried using a new connection.

具有比标识的流的标识符更低或等于，在连接关闭前没有完全关掉的流，重试请求，事务，或任何的协议活动都是不合理的，除了幂等的行为，比如HTTP GET，PUT，或DELETE。任何使用了更大的流标识符的流的协议活动可以安全地使用一个新的连接来重试。

> Activity on streams numbered lower or equal to the last stream identifier might still complete successfully. The sender of a GOAWAY frame might gracefully shut down a connection by sending a GOAWAY frame, maintaining the connection in an "open" state until all in-progress streams complete.

流标识符小于等于最后的流标识符的流上的活动可能依然可以成功完成。GOAWAY帧的发送者可以通过发送一个GOAWAY帧优雅地关闭一个连接，维护处于"open"状态地连接，直到所有进行中的流完成。

> An endpoint MAY send multiple GOAWAY frames if circumstances change. For instance, an endpoint that sends GOAWAY with NO_ERROR during graceful shutdown could subsequently encounter a condition that requires immediate termination of the connection. The last stream identifier from the last GOAWAY frame received indicates which streams could have been acted upon. Endpoints MUST NOT increase the value they send in the last stream identifier, since the peers might already have retried unprocessed requests on another connection.

如果条件变了的话，一个终端 **可以(MAY)** 发送多个GOAWAY帧。比如，一个终端在优雅的关闭期间发送了一个带有[NO_ERROR](https://http2.github.io/http2-spec/#NO_ERROR)错误码的GOAWAY，后面遇到了一个需要立即关闭连接的条件。收到的最后的GOAWAY帧中的最后流标识符字段指明了可能已经处理的流。终端 **一定不能(MUST NOT)** 增加它们发送的最后流标识符字段的值，因为对端可能已经在另一个连接上重试了未处理的请求。

> A client that is unable to retry requests loses all requests that are in flight when the server closes the connection. This is especially true for intermediaries that might not be serving clients using HTTP/2. A server that is attempting to gracefully shut down a connection SHOULD send an initial GOAWAY frame with the last stream identifier set to 231-1 and a NO_ERROR code. This signals to the client that a shutdown is imminent and that initiating further requests is prohibited. After allowing time for any in-flight stream creation (at least one round-trip time), the server can send another GOAWAY frame with an updated last stream identifier. This ensures that a connection can be cleanly shut down without losing requests.

一个不能重试请求的客户端丢失所有在服务器关闭连接时处于飞行中状态的请求。特别是对于不为使用HTTP/2的客户端服务的intermediaries。一个试图优雅地关闭一个连接的服务器 **应该(SHOULD)** 发送一个初始的GOAWAY帧，其中的最后流标识符被设为2^31-1，并包含[NO_ERROR](https://http2.github.io/http2-spec/#NO_ERROR)错误码。这通知客户端连接即将关闭，禁止初始化更多的请求。在一段为飞行中的流创建准备的时间之后(至少一个来回的时间)，服务器可以发送另一个GOAWAY帧，其中携带了一个更新过的最后流标识符。这确保连接可以被干净地关闭而不会丢失氢气。

> After sending a GOAWAY frame, the sender can discard frames for streams initiated by the receiver with identifiers higher than the identified last stream. However, any frames that alter connection state cannot be completely ignored. For instance, HEADERS, PUSH_PROMISE, and CONTINUATION frames MUST be minimally processed to ensure the state maintained for header compression is consistent (see Section 4.3); similarly, DATA frames MUST be counted toward the connection flow-control window. Failure to process these frames can cause flow control or header compression state to become unsynchronized.

发送一个GOAWAY帧之后，发送者可以丢弃由接收者初始化的，流标识符大于最后的流标识的流。然而，任何改变连接状态的帧不能被完全忽略。比如[HEADERS](https://http2.github.io/http2-spec/#HEADERS)，[PUSH_PROMISE](https://http2.github.io/http2-spec/#PUSH_PROMISE)，和[CONTINUATION](https://http2.github.io/http2-spec/#CONTINUATION)帧必须被最低限度地处理掉以确保为首部压缩维护的状态是一致的(参见[Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))；类似地，DATA帧 **必须(MUST)** 被计入连接的flow-control窗口。处理这些帧失败的话可能导致flow control或首部压缩状态变得不同步。

> The GOAWAY frame also contains a 32-bit error code (Section 7) that contains the reason for closing the connection.

GOAWAY帧也包含了一个32位的错误码([Section 7](https://http2.github.io/http2-spec/#ErrorCodes))，其中包含了关闭连接的原因。

> Endpoints MAY append opaque data to the payload of any GOAWAY frame. Additional debug data is intended for diagnostic purposes only and carries no semantic value. Debug information could contain security- or privacy-sensitive data. Logged or otherwise persistently stored debug data MUST have adequate safeguards to prevent unauthorized access.

终端 **可以(MAY)** 给GOAWAY帧的载荷附加不透明的数据。附加的调试数据只用于诊断目的，而不携带语义值。调试信息可能包含安全或隐私数据。记录或持续存储调试信息 **必须(MUST)** 具有充足的保护措施来防止未授权的访问。

## 6.9 WINDOW_UPDATE

> The WINDOW_UPDATE frame (type=0x8) is used to implement flow control; see Section 5.2 for an overview.

WINDOW_UPDATE帧(type=0x8)用于实现flow control；参考[Section 5.2](https://http2.github.io/http2-spec/#FlowControl)来做整体的了解。

> Flow control operates at two levels: on each individual stream and on the entire connection.

Flow control操作于两个层次：在每个单独的流上，和整个连接上。

> Both types of flow control are hop by hop, that is, only between the two endpoints. Intermediaries do not forward WINDOW_UPDATE frames between dependent connections. However, throttling of data transfer by any receiver can indirectly cause the propagation of flow-control information toward the original sender.

flow control的两种类型都是逐段的，即，只在两个端点之间。Intermediaries不在依赖的连接之间转发WINDOW_UPDATE帧。然而，任何接收者数据传输的限制可以间接地使得flow-control信息被传播到最初的发送者。

> Flow control only applies to frames that are identified as being subject to flow control. Of the frame types defined in this document, this includes only DATA frames. Frames that are exempt from flow control MUST be accepted and processed, unless the receiver is unable to assign resources to handling the frame. A receiver MAY respond with a stream error (Section 5.4.2) or connection error (Section 5.4.1) of type FLOW_CONTROL_ERROR if it is unable to accept a frame.

Flow control只应用于被认为是受控于flow control的帧。就这份文档中定义的帧类型而言，这只包括[DATA](https://http2.github.io/http2-spec/#DATA)帧。那些免除flow control的帧 **必须(MUST)** 被接受并处理，除非接收者不能分配资源来处理帧。如果接收者不能接受一个帧，则它 **可以(MAY)** 响应一个类型是[FLOW_CONTROL_ERROR](https://http2.github.io/http2-spec/#FLOW_CONTROL_ERROR)的流错误([Section 5.4.2](https://http2.github.io/http2-spec/#StreamErrorHandler))或连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

```
 +-+-------------------------------------------------------------+
 |R|              Window Size Increment (31)                     |
 +-+-------------------------------------------------------------+
```
> Figure 14: WINDOW_UPDATE Payload Format

图 14: WINDOW_UPDATE载荷格式

> The payload of a WINDOW_UPDATE frame is one reserved bit plus an unsigned 31-bit integer indicating the number of octets that the sender can transmit in addition to the existing flow-control window. The legal range for the increment to the flow-control window is 1 to 231-1 (2,147,483,647) octets.

WINDOW_UPDATE帧的载荷由一个保留位外加一个无符号31位的整数组成，其中后者指明了发送者在已有的flow-control窗口之外可以传输的字节数。flow-control窗口合法的增量范围是1到2^31-1 (2,147,483,647)字节。

> The WINDOW_UPDATE frame does not define any flags.

WINDOW_UPDATE帧不定义任何标记。

> The WINDOW_UPDATE frame can be specific to a stream or to the entire connection. In the former case, the frame's stream identifier indicates the affected stream; in the latter, the value "0" indicates that the entire connection is the subject of the frame.

WINDOW_UPDATE帧可以应用于特定的流，也可以是整个连接。对于前者，帧的流标识符指明了受影响的流；而后者，"0"值指明了整个连接受控于该帧。

> A receiver MUST treat the receipt of a WINDOW_UPDATE frame with an flow-control window increment of 0 as a stream error (Section 5.4.2) of type PROTOCOL_ERROR; errors on the connection flow-control window MUST be treated as a connection error (Section 5.4.1).

接收者 **必须(MUST)** 将收到flow-control窗口增量为0的WINDOW_UPDATE帧 作为一个类型为[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的流错误([Section 5.4.2](https://http2.github.io/http2-spec/#StreamErrorHandler))；连接的flow-control窗口的错误flow-control **必须(MUST)** 被作为一个连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> WINDOW_UPDATE can be sent by a peer that has sent a frame bearing the END_STREAM flag. This means that a receiver could receive a WINDOW_UPDATE frame on a "half-closed (remote)" or "closed" stream. A receiver MUST NOT treat this as an error (see Section 5.1).

WINDOW_UPDATE可以由已经发送了一个携带END_STREAM标记的帧的对端发送。这意味着接收者可能在一个"half-closed (remote)"或"closed"流上接收到WINDOW_UPDATE帧。接收者 **一定不能(MUST NOT)** 将这当做一个错误(参见 [Section 5.1](https://http2.github.io/http2-spec/#StreamStates)).。

> A receiver that receives a flow-controlled frame MUST always account for its contribution against the connection flow-control window, unless the receiver treats this as a connection error (Section 5.4.1). This is necessary even if the frame is in error. The sender counts the frame toward the flow-control window, but if the receiver does not, the flow-control window at the sender and receiver can become different.

接收到一个flow-controlled帧的接收者 **必须(MUST)** 总是考虑到它对连接flow-control窗口的影响，除非接收者把这当做一个连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。即使帧是错误的这也是必须的。发送者将帧计入flow-control，但如果接收者没有这样做的话，发送者和接收者的flow-control窗口可能变得不同。

> A WINDOW_UPDATE frame with a length other than 4 octets MUST be treated as a connection error (Section 5.4.1) of type FRAME_SIZE_ERROR.

长度不是4字节的WINDOW_UPDATE帧 **必须(MUST)** 被作为一个类型是[FRAME_SIZE_ERROR](https://http2.github.io/http2-spec/#FRAME_SIZE_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

6.9.1 The Flow-Control Window

### 6.9.1 Flow-Control窗口

> Flow control in HTTP/2 is implemented using a window kept by each sender on every stream. The flow-control window is a simple integer value that indicates how many octets of data the sender is permitted to transmit; as such, its size is a measure of the buffering capacity of the receiver.

HTTP/2中的Flow control使用一个由每个流上的每个发送者持有的窗口实现。flow-control窗口是一个简单的整型值，它指明了发送者允许传输的字节数；同样地，它的大小衡量了接收者的缓存能力。

> Two flow-control windows are applicable: the stream flow-control window and the connection flow-control window. The sender MUST NOT send a flow-controlled frame with a length that exceeds the space available in either of the flow-control windows advertised by the receiver. Frames with zero length with the END_STREAM flag set (that is, an empty DATA frame) MAY be sent if there is no available space in either flow-control window.

两种flow-control窗口是适用的：流flow-control窗口和连接flow-control窗口。发送者 **一定不能(MUST NOT)** 发送一个长度超出了由接收者广告的任一种flow-control窗口的可用空间的flow-controlled帧。两种flow-control窗口中没有可用空间时，**可以(MAY)** 发送长度为0且END_STREAM标记被设置的帧(即，一个空的[DATA](https://http2.github.io/http2-spec/#DATA)帧)。

> For flow-control calculations, the 9-octet frame header is not counted.

对于flow-control的计算，9字节的帧首部不计入内。

> After sending a flow-controlled frame, the sender reduces the space available in both windows by the length of the transmitted frame.

发送一个flow-controlled帧之后，发送者给两个窗口中的可用空间都减小发送的帧的大小。

> The receiver of a frame sends a WINDOW_UPDATE frame as it consumes data and frees up space in flow-control windows. Separate WINDOW_UPDATE frames are sent for the stream- and connection-level flow-control windows.

帧的接收者由于它消耗数据并释放flow-control窗口的空间而发送一个WINDOW_UPDATE帧。独立的WINDOW_UPDATE帧为流级及连接级的flow-control窗口而发送。

> A sender that receives a WINDOW_UPDATE frame updates the corresponding window by the amount specified in the frame.

接收到一个WINDOW_UPDATE帧的发送者给对应窗口更新帧中描述的数量。

> A sender MUST NOT allow a flow-control window to exceed 231-1 octets. If a sender receives a WINDOW_UPDATE that causes a flow-control window to exceed this maximum, it MUST terminate either the stream or the connection, as appropriate. For streams, the sender sends a RST_STREAM with an error code of FLOW_CONTROL_ERROR; for the connection, a GOAWAY frame with an error code of FLOW_CONTROL_ERROR is sent.

发送者 **一定不能(MUST NOT)** 允许一个flow-control窗口超出2^31-1字节。如果发送者接收了一个WINDOW_UPDATE，它导致flow-control窗口超出了最大值，则它 **必须(MUST)** 酌情终止流或连接。对于流，发送者发送一个[RST_STREAM](https://http2.github.io/http2-spec/#RST_STREAM)，其中由一个错误码[FLOW_CONTROL_ERROR](https://http2.github.io/http2-spec/#FLOW_CONTROL_ERROR)；对于连接，发送一个[GOAWAY](https://http2.github.io/http2-spec/#GOAWAY)帧，其中包含错误码[FLOW_CONTROL_ERROR](https://http2.github.io/http2-spec/#FLOW_CONTROL_ERROR)。

> Flow-controlled frames from the sender and WINDOW_UPDATE frames from the receiver are completely asynchronous with respect to each other. This property allows a receiver to aggressively update the window size kept by the sender to prevent streams from stalling.

来自于发送者的Flow-controlled帧和来自于接收者的WINDOW_UPDATE帧相互之间是完全异步的。这个属性允许接收者侵略地更新由发送者保存的窗口大小来防止流的失速。

6.9.2 Initial Flow-Control Window Size

### 6.9.2 初始的Flow-Control窗口大小

> When an HTTP/2 connection is first established, new streams are created with an initial flow-control window size of 65,535 octets. The connection flow-control window is also 65,535 octets. Both endpoints can adjust the initial window size for new streams by including a value for SETTINGS_INITIAL_WINDOW_SIZE in the SETTINGS frame that forms part of the connection preface. The connection flow-control window can only be changed using WINDOW_UPDATE frames.

当HTTP/2连接首次被建立时，新建立的流的初始flow-control窗口大小为65,535字节。连接的flow-control窗口也是65,535字节。两端可以通过在构成连接preface一部分的[SETTINGS](https://http2.github.io/http2-spec/#SETTINGS)帧中包含一个[SETTINGS_INITIAL_WINDOW_SIZE](https://http2.github.io/http2-spec/#SETTINGS_INITIAL_WINDOW_SIZE)值来为新流调整初始窗口大小。连接的flow-control窗口只能使用WINDOW_UPDATE帧来改变。

> Prior to receiving a SETTINGS frame that sets a value for SETTINGS_INITIAL_WINDOW_SIZE, an endpoint can only use the default initial window size when sending flow-controlled frames. Similarly, the connection flow-control window is set to the default initial window size until a WINDOW_UPDATE frame is received.

在接收到设置了[SETTINGS_INITIAL_WINDOW_SIZE](https://http2.github.io/http2-spec/#SETTINGS_INITIAL_WINDOW_SIZE)值的[SETTINGS](https://http2.github.io/http2-spec/#SETTINGS)值之前，一个端点在发送flow-controlled帧时只能使用默认的初始窗口大小。类似地，连接的flow-control窗口在收到WINDOW_UPDATE帧之前，被设置为默认的初始窗口大小。

> In addition to changing the flow-control window for streams that are not yet active, a SETTINGS frame can alter the initial flow-control window size for streams with active flow-control windows (that is, streams in the "open" or "half-closed (remote)" state). When the value of SETTINGS_INITIAL_WINDOW_SIZE changes, a receiver MUST adjust the size of all stream flow-control windows that it maintains by the difference between the new value and the old value.

除了给还不处于活跃状态的流修改flow-control，[SETTINGS](https://http2.github.io/http2-spec/#SETTINGS)帧可以改变具有活跃的flow-control窗口的流(即，处于"open"或"half-closed (remote)"状态的流)的初始flow-control窗口大小。当[SETTINGS_INITIAL_WINDOW_SIZE](https://http2.github.io/http2-spec/#SETTINGS_INITIAL_WINDOW_SIZE)的值改变时，接收者 **必须(MUST)** 根据新值和老值之间的差异调整它维护的所有的flow-control窗口的大小

> A change to SETTINGS_INITIAL_WINDOW_SIZE can cause the available space in a flow-control window to become negative. A sender MUST track the negative flow-control window and MUST NOT send new flow-controlled frames until it receives WINDOW_UPDATE frames that cause the flow-control window to become positive.

对于[SETTINGS_INITIAL_WINDOW_SIZE](https://http2.github.io/http2-spec/#SETTINGS_INITIAL_WINDOW_SIZE)的改变，可能导致一个flow-control窗口中的可用空间变为负值。一个发送者 **必须(MUST)** 追踪负的flow-control窗口，且 **一定不能(MUST NOT)** 发送新的flow-controlled帧，直到它收到了使flow-control窗口变为正值的WINDOW_UPDATE帧。

> For example, if the client sends 60 KB immediately on connection establishment and the server sets the initial window size to be 16 KB, the client will recalculate the available flow-control window to be -44 KB on receipt of the SETTINGS frame. The client retains a negative flow-control window until WINDOW_UPDATE frames restore the window to being positive, after which the client can resume sending.

比如，如果连接一建立客户端就立即发送了60 KB，而服务器将初始的窗口大小设置为16 KB，则客户端在收到[SETTINGS](https://http2.github.io/http2-spec/#SETTINGS)帧时将重新计算可用的flow-control窗口，其值为-44 KB。客户端保持负的flow-control窗口直到WINDOW_UPDATE帧将窗口恢复为正值，然后客户端可以恢复发送。

> A SETTINGS frame cannot alter the connection flow-control window.

一个[SETTINGS](https://http2.github.io/http2-spec/#SETTINGS)帧不能改变连接的flow-control窗口。

> An endpoint MUST treat a change to SETTINGS_INITIAL_WINDOW_SIZE that causes any flow-control window to exceed the maximum size as a connection error (Section 5.4.1) of type FLOW_CONTROL_ERROR.

终端必须将导致任何flow-control窗口超出最大大小的[SETTINGS_INITIAL_WINDOW_SIZE](https://http2.github.io/http2-spec/#SETTINGS_INITIAL_WINDOW_SIZE)的修改作为一个类型[FLOW_CONTROL_ERROR](https://http2.github.io/http2-spec/#FLOW_CONTROL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

6.9.3 Reducing the Stream Window Size

### 6.9.3 减小流窗口大小

> A receiver that wishes to use a smaller flow-control window than the current size can send a new SETTINGS frame. However, the receiver MUST be prepared to receive data that exceeds this window size, since the sender might send data that exceeds the lower limit prior to processing the SETTINGS frame.

想要使用一个比当前大小更小的flow-control窗口的接收者可以发送一个新的[SETTINGS](https://http2.github.io/http2-spec/#SETTINGS)帧。然而，接收者 **必须(MUST)** 准备接收超出窗口大小的数据，因为发送者可能在处理[SETTINGS](https://http2.github.io/http2-spec/#SETTINGS)帧之前发送数据而超出了更低的限制。

> After sending a SETTINGS frame that reduces the initial flow-control window size, a receiver MAY continue to process streams that exceed flow-control limits. Allowing streams to continue does not allow the receiver to immediately reduce the space it reserves for flow-control windows. Progress on these streams can also stall, since WINDOW_UPDATE frames are needed to allow the sender to resume sending. The receiver MAY instead send a RST_STREAM with an error code of FLOW_CONTROL_ERROR for the affected streams.

发送了一个减小初始flow-control窗口大小的SETTINGS帧之后，接收者 **可以(MAY)** 继续处理超出flow-control限制的流。允许流继续不允许接收者立即减小它为flow-control窗口保留的空间。这些流中的进程可能失速，由于需要[WINDOW_UPDATE](https://http2.github.io/http2-spec/#WINDOW_UPDATE)帧来允许发送者恢复发送。接收者 **可以(MAY)** 为受到影响的流发送错误码为[FLOW_CONTROL_ERROR](https://http2.github.io/http2-spec/#FLOW_CONTROL_ERROR)的[RST_STREAM](https://http2.github.io/http2-spec/#RST_STREAM)。

## 6.10 CONTINUATION

> The CONTINUATION frame (type=0x9) is used to continue a sequence of header block fragments (Section 4.3). Any number of CONTINUATION frames can be sent, as long as the preceding frame is on the same stream and is a HEADERS, PUSH_PROMISE, or CONTINUATION frame without the END_HEADERS flag set.

CONTINUATION帧 (type=0x9) 被用于继续发送首部块片段([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))的序列。只要相同流上的前导帧是没有设置END_HEADERS标记的[HEADERS](https://http2.github.io/http2-spec/#HEADERS)，[PUSH_PROMISE](https://http2.github.io/http2-spec/#PUSH_PROMISE)，或CONTINUATION帧，就可以发送任意数量的CONTINUATION帧。

```
 +---------------------------------------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
```
> Figure 15: CONTINUATION Frame Payload

图 15: CONTINUATION帧载荷

> The CONTINUATION frame payload contains a header block fragment (Section 4.3).

CONTINUATION帧载荷包含一个首部块片段([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))。

> The CONTINUATION frame defines the following flag:

CONTINUATION帧定义了如下的标记：

> END_HEADERS (0x4):
> When set, bit 2 indicates that this frame ends a header block (Section 4.3).

**END_HEADERS (0x4):**

当设置时，位2指明这个帧结束了一个首部块([Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock))。

> If the END_HEADERS bit is not set, this frame MUST be followed by another CONTINUATION frame. A receiver MUST treat the receipt of any other type of frame or a frame on a different stream as a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

如果END_HEADERS位没有设置，则这个帧后面 **必须(MUST)** 跟着另一个CONTINUATION帧。如果一个接收者接收到了其它类型的帧，或在一个不同的流上接收到了帧，则必须将这作为类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。

> The CONTINUATION frame changes the connection state as defined in Section 4.3.

CONTINUATION帧如[Section 4.3](https://http2.github.io/http2-spec/#HeaderBlock)中定义的那样改变连接状态。

> CONTINUATION frames MUST be associated with a stream. If a CONTINUATION frame is received whose stream identifier field is 0x0, the recipient MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

CONTINUATION帧 **必须(MUST)** 与一个流关联。如果接收到了一个流标识符字段为0x0的CONTINUATION帧，则接收者必须以一个类型为PROTOCOL_ERROR的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))来响应。

> A CONTINUATION frame MUST be preceded by a HEADERS, PUSH_PROMISE or CONTINUATION frame without the END_HEADERS flag set. A recipient that observes violation of this rule MUST respond with a connection error (Section 5.4.1) of type PROTOCOL_ERROR.

CONTINUATION帧的前面必须是END_HEADERS标记没有设置的[HEADERS](https://http2.github.io/http2-spec/#HEADERS)，[PUSH_PROMISE](https://http2.github.io/http2-spec/#PUSH_PROMISE)或CONTINUATION帧。接收者如果发现违背了这个规则， **必须(MUST)** 响应一个类型是[PROTOCOL_ERROR](https://http2.github.io/http2-spec/#PROTOCOL_ERROR)的连接错误([Section 5.4.1](https://http2.github.io/http2-spec/#ConnectionErrorHandler))。
