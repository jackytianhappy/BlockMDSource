---
title: 直播日记-4 RTMP原理篇章
date: 2017-03-24 11:13:11
tags: 直播
---

## RTMP
RTMP协议是Real Time Message Protocol(实时信息传输协议)的缩写，它是由Adobe公司提出的一种应用层的协议，用来解决多媒体数据传输流的多路复用（Multiplexing）和分包（packetizing）的问题。

### 1.总体介绍
RTMP协议是应用层协议，是要靠底层可靠的传输层协议（通常是TCP）来保证信息传输的可靠性的。在基于传输层协议的链接建立完成后，RTMP协议也要客户端和服务器通过“握手”来建立基于传输层链接之上的RTMP Connection链接，在Connection链接上会传输一些控制信息，如SetChunkSize,SetACKWindowSize。<br>
其中CreateStream命令会创建一个Stream链接，用于传输具体的音视频数据和控制这些信息传输的命令信息。RTMP协议传输时会对数据做自己的格式化，这种格式的消息我们称之为RTMP Message，而实际传输的时候为了更好地实现多路复用、分包和信息的公平性，发送端会把Message划分为带有Message ID的Chunk，每个Chunk可能是一个单独的Message，也可能是Message的一部分，在接受端会根据chunk中包含的data的长度，message id和message的长度把chunk还原成完整的Message，从而实现信息的收发。

### 2.握手
要建立一个有效的RTMP Connection链接，首先要“握手”:客户端要向服务器发送C0,C1,C2（按序）三个chunk，服务器向客户端发送S0,S1,S2（按序）三个chunk，然后才能进行有效的信息传输。RTMP协议本身并没有规定这6个Message的具体传输顺序，但RTMP协议的实现者需要保证这几点：<br>
-  客户端要等收到S1之后才能发送C2<br>
-  客户端要等收到S2之后才能发送其他信息（控制信息和真实音视频等数据)<br>
-  服务端要等到收到C0之后发送S1<br>
-  服务端必须等到收到C1之后才能发送S2<br>
-  服务端必须等到收到C2之后才能发送其他信息（控制信息和真实音视频等数据）<br>
服务端收发的顺序如下：<br>
![Alt text](https://github.com/jackytianhappy/ImgSource/blob/master/RTMP-HandShake.png?raw=true)<br>
理论上来讲只要满足以上条件，如何安排6个Message的顺序都是可以的，但实际实现中为了在保证握手的身份验证功能的基础上尽量减少通信的次数，一般的发送顺序是这样的，这一点可以通过wireshark抓ffmpeg推流包进行验证：<br>
｜client｜Server ｜<br>
｜－－－C0+C1—->|<br>
｜<－－S0+S1+S2– |<br>
｜－－－C2-－－－> ｜<br>

### 3. RTMP Chunk Stream
Chunk Stream是对传输RTMP Chunk的流的逻辑上的抽象，客户端和服务器之间有关RTMP的信息都在这个流上通信。这个流上的操作也是我们关注RTMP协议的重点。

#### 3.1 Message(消息)
这里的Message是指满足该协议格式的、可以切分成Chunk发送的消息，消息包含的字段如下：<br>
-  Timestamp（时间戳）：消息的时间戳，4个字节
-  Length(长度)：是指Message Payload（消息负载）即音视频等信息的数据的长度，3个字节
-  TypeId(类型Id)：消息的类型Id，1个字节
-  Message Stream ID（消息的流ID）：每个消息的唯一标识，划分成Chunk和还原Chunk为Message的时候都是根据这个ID来辨识是否是同一个消息的Chunk的，4个字节，并且以小端格式存储

#### 3.2 Chunking(Message分块)
RTMP在收发数据的时候并不是以Message为单位的，而是把Message拆分成Chunk发送，而且必须在一个Chunk发送完成之后才能开始发送下一个Chunk。每个Chunk中带有MessageID代表属于哪个Message，接受端也会按照这个id来将chunk组装成Message。<br>
















