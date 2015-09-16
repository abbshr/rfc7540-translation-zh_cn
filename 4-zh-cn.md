# 4. HTTP Frames

## 4.1 Frame Format

所有的frame都以9字节的header开始, 后跟变长的payload(有效载荷):

```
+-----------------------------------------------+
|                 Length (24)                   |
+---------------+---------------+---------------+
|   Type (8)    |   Flags (8)   |
+-+-------------+---------------+-------------------------------+
|R|                 Stream Identifier (31)                      |
+=+=============================================================+
|                   Frame Payload (0...)                      ...
+---------------------------------------------------------------+
```

`Length`: payload长度, 无符号24bit整型. **大于2^14(16384 byte)的payload只有在接收方设置了更大的`SETTINGS_MAX_FRAME_SIZE`才可以发送**

`Type`: 帧类型, 决定帧的格式和语义

`Flags`: 为Type保留的bool标识, 针对确定的帧类型赋予特定的语义, 否则发送时必须忽略(设置为0x0).

`R`: 1bit的保留字段, 尚未定义语义, 发送和接收必须忽略(0x0).

`Stream Identifier`: 31字节的流标示符. 0x0作为保留值, 表示与连接相关的frames作为一个整体而不是一个单独的流.

## 4.2 Frame Size

payload大小被接收方设置的`SETTINGS_MAX_FRAME_SIZE`所限制, 而这个值的取值区间是[2^14 (16384), 2^24-1(16777215)] byte.

**The size of the frame header is not when describing frame sizes.**

## 4.3 Header压缩和解压

HTTP/2与HTTP/1的Header结构一样, 一键多值. 除了用于HTTP请求和响应之外还用于server push操作.

### 压缩&分片
Header list包含0个或多个Header字段. 传输过程是: 先用压缩把Header list序列化成一个Header block. 然后将block分割成一个或多个字节序列, 称之为header block分片. 分片放在HEADERS Frame, PUSH_PROMISE FRAME以及CONTINUATION Frame的payload里传输.

其中Cookie字段由HTTP mapping特殊处理.

### 解压&重组
报文接收端将分片拼接起来以重组header block, 然后解压block得到原始的header list.

一个完整地header block可以通过下面任意一种结构组成:

+ 一个设置了`END_HEADERS`标记的`HEADERS`或`PUSH_PROMISE` frame.
+ 一个`END_HEADERS`标记置空得`HEADERS`或`PUSH_PROMISE` frame, 后接一个或多个`CONTINUATION`frame, 并且最后一个`CONTINUATION`frame设置了`END_HEADERS`标记.

Header压缩是**有状态**的. 整个连接复用同一个压缩和解压上下文.
Header block中发生的解码错误必须当成连接错误, 类型为`COMPRESSION_ERROR`.

每个Header block当做离散的单元处理. 必须确保Header block分片连续传输, 期间不能交叉其他类型的或来自任何其他流的frame. `HEADERS`与`PUSH_PROMISE`的 `CONTINUATION`传输序列中最后一个分片一定设置了`END_HEADERS`标记. 这保证了header block在逻辑上等价于单个frame.

Header block分片只能作为`HEADERS`, `PUSH_PROMISE`以及`CONTINUATION`frame的payload. 因为这些frame携带的数据可能修改接收方维护的压缩上下文: 接收方拿到这三种类型之一的frame之后需要重组header block并解压, 即便这些frame将被丢弃. 如果无法解压header block, 接收方必须以一个类型为`COMPRESSION_ERROR`的错误断开连接.
