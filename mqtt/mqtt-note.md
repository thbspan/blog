## 概述
    MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是IBM开发的一个即时通讯协议，有可能成为物联网的重要组成部分。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和制动器（比如通过Twitter让房屋联网）的通信协议
## 特点
使用发布/订阅消息模式，提供一对多的消息发布，解除应用程序耦合

对负载内容屏蔽的消息传输

有三种消息发布服务质量：
* “至多一次”，消息发布完全依赖底层 TCP/IP 网络。会发生消息丢失或重复
* “至少一次”，确保消息到达，但消息重复可能会发生
* “只有一次”，确保消息到达一次。这一级别可用于如下情况，在计费系统中，消息重复或丢失会导致不正确的结果。

## 3.1.1版本文档地址
[英文原文](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Introduction)

[中文](https://mcxiaoke.gitbooks.io/mqtt-cn/content/mqtt/01-Introduction.html?q=)

