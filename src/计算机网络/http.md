HTTP/2 （超文本传输协议第2版，最初命名为 HTTP 2.0 ），简称为 h2 （基于TLS/1.2或以上版本的加密连接）或 h2c （非加密连接)，是 HTTP 协议的的第二个主要版本。 管线化请求 Pipelining of requests. 对数据传输采用多路复用，让多个请求合并在同一 TCP 连接内 Multiplexing multiple requests over a single TCP connection， 因为每一个tcp 连接在创建的时候都需要耗费资源，而且在创建初期，传输也是比较慢的。 采用了二进制而非明文来打包、传输 客户端<——>服务器 间的数据。 HTTP/2 的设计本身允许非加密的 HTTP 协议，也允许使用 TLS 1.2 或更新版本协议进行加密。

[SPDY介绍](https://www.cnblogs.com/keva/p/spdy-protocol.html)


Linux操作系统实现了计算机网络的各种协议，包括 http tcp udp dns arp 等协议，协议本身是理论，《计算机网络》这本书学习的是理论，实际落地和实现，需要依赖 os ，比如 Windows 和 Linux、macOS 等等