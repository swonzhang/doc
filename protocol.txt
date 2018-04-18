

参考资料： https://www.cnblogs.com/syfwhu/p/5219814.html#

tcp正常通讯

11:51:41.720356 IP 192.168.2.63.60298 > 192.168.2.21.9501: Flags [S], seq 542865493, win 29200, options [mss 1460,sackOK,TS val 72696915 ecr 0,nop,wscale 7], length 0
11:51:41.720421 IP 192.168.2.21.9501 > 192.168.2.63.60298: Flags [S.], seq 862544496, ack 542865494, win 28960, options [mss 1460,sackOK,TS val 210719011 ecr 72696915,nop,wscale 7], length 0
11:51:41.720949 IP 192.168.2.63.60298 > 192.168.2.21.9501: Flags [.], ack 1, win 229, options [nop,nop,TS val 72696916 ecr 210719011], length 0
11:51:42.722935 IP 192.168.2.63.60298 > 192.168.2.21.9501: Flags [P.], seq 1:36, ack 1, win 229, options [nop,nop,TS val 72697918 ecr 210719011], length 35
11:51:42.722978 IP 192.168.2.21.9501 > 192.168.2.63.60298: Flags [.], ack 36, win 227, options [nop,nop,TS val 210720014 ecr 72697918], length 0
11:51:42.735189 IP 192.168.2.21.9501 > 192.168.2.63.60298: Flags [P.], seq 1:45, ack 36, win 227, options [nop,nop,TS val 210720026 ecr 72697918], length 44
11:51:42.735783 IP 192.168.2.63.60298 > 192.168.2.21.9501: Flags [.], ack 45, win 229, options [nop,nop,TS val 72697931 ecr 210720026], length 0
11:51:43.736866 IP 192.168.2.63.60298 > 192.168.2.21.9501: Flags [F.], seq 36, ack 45, win 229, options [nop,nop,TS val 72698932 ecr 210720026], length 0
11:51:43.750807 IP 192.168.2.21.9501 > 192.168.2.63.60298: Flags [F.], seq 45, ack 37, win 227, options [nop,nop,TS val 210721042 ecr 72698932], length 0
11:51:43.751694 IP 192.168.2.63.60298 > 192.168.2.21.9501: Flags [.], ack 46, win 229, options [nop,nop,TS val 72698946 ecr 210721042], length 0


tcp + tls 通讯



三次握手
12:53:54.583145 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [S], seq 974194940, win 29200, options [mss 1460,sackOK,TS val 76429777 ecr 0,nop,wscale 7], length 0
12:53:54.583183 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [S.], seq 440742550, ack 974194941, win 28960, options [mss 1460,sackOK,TS val 214451874 ecr 76429777,nop,wscale 7], length 0
12:53:54.583530 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [.], ack 1, win 229, options [nop,nop,TS val 76429778 ecr 214451874], length 0

TCP连接建立后，客户端发送一些协商信息，如TLS协议版本，支持的密码套件的列表，和其他TLS选项。
12:53:54.584496 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [P.], seq 1:290, ack 1, win 229, options [nop,nop,TS val 76429779 ecr 214451874], length 289
12:53:54.584545 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [.], ack 290, win 235, options [nop,nop,TS val 214451875 ecr 76429779], length 0

服务器挑选TLS协议版本，在加密套件列表中挑选一个密码套件，附带自己的证书，并将响应返回给客户端。可选的，服务器也可以发送对客户端的证书认证请求和其他TLS扩展参数。
12:53:54.630552 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [P.], seq 1:2124, ack 290, win 235, options [nop,nop,TS val 214451921 ecr 76429779], length 2123
12:53:54.630958 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [.], ack 2124, win 262, options [nop,nop,TS val 76429825 ecr 214451921], length 0

假设双方协商好一个共同的TLS版本和加密算法，客户端使用服务器提供的证书，生成新的对称密钥，并用服务器的公钥进行加密，并告诉服务器切换到加密通信流程。到现在为止，所有被交换的数据都是以明文方式传输，除了对称密钥外，它采用的是服务器端的公钥加密。
12:53:54.635587 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [P.], seq 290:460, ack 2124, win 262, options [nop,nop,TS val 76429830 ecr 214451921], length 170
12:53:54.635606 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [.], ack 460, win 243, options [nop,nop,TS val 214451926 ecr 76429830], length 0

服务器用自己的私钥解密客户端发过来的对称密钥，并通过验证MAC检查消息的完整性，并返回给客户端一个加密的“Finished”的消息。
12:53:54.637021 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [P.], seq 2124:2350, ack 460, win 243, options [nop,nop,TS val 214451928 ecr 76429830], length 226
客户端采用对称密钥解密消息，并验证MAC，如果一切OK，加密隧道就建立好了。应用程序数据就可以发送了。
12:53:54.677694 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [.], ack 2350, win 284, options [nop,nop,TS val 76429872 ecr 214451928], length 0

开始发送数据
12:53:55.638443 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [P.], seq 460:524, ack 2350, win 284, options [nop,nop,TS val 76430832 ecr 214451928], length 64
12:53:55.677616 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [.], ack 524, win 243, options [nop,nop,TS val 214452969 ecr 76430832], length 0
12:53:55.678509 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [P.], seq 2350:2415, ack 524, win 243, options [nop,nop,TS val 214452969 ecr 76430832], length 65
12:53:55.682320 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [.], ack 2415, win 284, options [nop,nop,TS val 76430876 ecr 214452969], length 0

断开连接
12:53:56.683691 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [F.], seq 524, ack 2415, win 284, options [nop,nop,TS val 76431878 ecr 214452969], length 0
12:53:56.687232 IP 192.168.2.21.9501 > 192.168.2.63.60310: Flags [F.], seq 2415, ack 525, win 243, options [nop,nop,TS val 214453978 ecr 76431878], length 0
12:53:56.688191 IP 192.168.2.63.60310 > 192.168.2.21.9501: Flags [.], ack 2416, win 284, options [nop,nop,TS val 76431882 ecr 214453978], length 0


断开连接这里，服务端没有 ack客户端的fin

服务器返回的FIN+ACK就是四次挥手的第二次和第三次挥手啊，只是两次挥手的报文合成一个报文了而已。这种情况下没必要分成两个报文来回复，除非出现io或sleep等延时操作，才会分开发送