1、双方的通信：简单的对称加密方式通信。
2、https：解决对称加密的密钥怎么安全传给对方，用：非对称加密方式（公钥加密私钥解密）
3、非对称加密的公钥传递如何防篡改：就是 CA 机构的非对称加密（私钥加密公钥解密）


A向CA机构要求加密A的公钥
CA机构用自己的私钥加密了A的公钥给A
A把CA发给他的信息发给B
B用CA机构的公钥解开信息得到A的公钥
B用A的公钥加密对称钥给A
A用私钥解开信息得到B提供的对称钥
然后A与B就可以快乐的用对称钥进行信息通信了

说明和理解：在所有的传输中，有2次非对称加密，分别是：A和B向CA要的公钥。解锁得到A的公钥。1次对称加密，就是传输内容的对称加密。

我们先来回忆一下HTTPS的通信流程，HTTPS协议 = HTTP协议 + SSL/TLS协议，摘取一下网上一些八股文的回答(以RSA密钥交换的为例)！

(1)客户端生成一个随机数client_random,TLS版本号，发送到服务端
(2)服务端发送自己的随机数server_random，服务器使用的证书，发送到客户端
(3)客户端利用CA公钥对证书进行验证，取出服务器公钥
(4)客户端生成随机数pre_master_secret,利用服务器公钥进行加密，传送到服务端
(5)服务端利用服务器私钥进行解密取出pre_master_secret
(6)服务端和客户端此时利用随机数client_random,server_random,
pre_master_secret算出对称密钥(master_secret)，利用对称密钥进行对称加密通信
画外音:是不是贼熟悉，有背过网络八股文的，一看就懂！

关键问题就在步骤(6)，怎么进行加密的？很多文章都没有说明，甚至有的人以为，拿client_random+server_random+pre_master_secret直接拼成一个字符串，然后就是对称加密密钥，客户端和服务端拿这个密钥对数据进行加密通信！


在流加密模式下，MAC验证码公式为(摘自rfc2246,section6.2.3.1)

HMAC_hash(MAC_write_secret, 
seq_num + TLSCompressed.type +
TLSCompressed.version + TLSCompressed.length +
TLSCompressed.fragment))
看到入参中的seq_num了么？这就是数据的序列号，这个序号就是用来防止重放攻击的

假设，客户端和服务端相互通信了4次，client_send = server_recv=3(从0开始，所以是3)，服务端检验完第4次消息后，server_recv加1，此时server_recv=4。攻击者如果想重放第4次消息，第4次消息中的client_send值是3，就会出现校验失败的情况！从而能够抵挡住重放攻击！