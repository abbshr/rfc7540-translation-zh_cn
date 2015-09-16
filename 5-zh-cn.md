# 5. 流和多路复用

在一个HTTP/2的连接中, 流是用于客户端与服务器交换frame的独立双向序列. 流有几个重要的特点:

+ 一个HTTP/2连接可以包含多个并发的流, 各个端点从多个流中交换frame
+ 流可以被客户端或服务器单方面建立, 使用或共享
+ 流也可以被任意一方关闭
+ frames在一个流上的发送顺序很重要. 接收方将按照他们的接收顺序处理这些frame. 特别是HEADERS和DATA frame的顺序, 在协议的语义上显得尤为重要.
+ 流用一个整数(流标识符)标记. 端点初始化流的时候就为其分配了标识符.

## 5.1 流状态

下图展示了流的生存周期:

```
                         +--------+
                 send PP |        | recv PP
                ,--------|  idle  |--------.
               /         |        |         \
              v          +--------+          v
       +----------+          |           +----------+
       |          |          | send H /  |          |
,------| reserved |          | recv H    | reserved |------.
|      | (local)  |          |           | (remote) |      |
|      +----------+          v           +----------+      |
|          |             +--------+             |          |
|          |     recv ES |        | send ES     |          |
|   send H |     ,-------|  open  |-------.     | recv H   |
|          |    /        |        |        \    |          |
|          v   v         +--------+         v   v          |
|      +----------+          |           +----------+      |
|      |   half   |          |           |   half   |      |
|      |  closed  |          | send R /  |  closed  |      |
|      | (remote) |          | recv R    | (local)  |      |
|      +----------+          |           +----------+      |
|           |                |                 |           |
|           | send ES /      |       recv ES / |           |
|           | send R /       v        send R / |           |
|           | recv R     +--------+   recv R   |           |
| send R /  `----------->|        |<-----------'  send R / |
| recv R                 | closed |               recv R   |
`----------------------->|        |<----------------------'
                         +--------+

   send:   发送这个frame的终端
   recv:   接受这个frame的终端

   H:  HEADERS frame (隐含CONTINUATION frame)
   PP: PUSH_PROMISE frame (隐含CONTINUATION frame)
   ES: END_STREAM标记
   R:  RST_STREAM frame
```

该图只展示了流的状态转换以及frame和标记如何对转换产生影响. 这方面, `CONTINUATION`frames不会导致状态的转换, 他们只是跟在HEADERS或PUSH_PROMISE frame后面的有效组成部分.

状态转换的用途, 对于设置了`END_STREAM`标记的frame来说, `END_STREAM`被当做一个分开的事件处理. 设置了`END_STREAM`标记的HEADERS frame会导致两次状态转换.

在传输过程中, 每个端点对流状态的主观认识可能不同. 这些终端不会协商流的创建, 都是由终端独立创建的. 端点的流状态不同会带来负面影响: 在发送了`RST_STREAM`之后流处于关闭状态, 而frame可能在流关闭之后才到达.

流有如下状态:

+ idle
+ reserved(local)
+ reserved(remote)
+ open
+ half-closed(local)
+ half-closed(remote)
+ closed
