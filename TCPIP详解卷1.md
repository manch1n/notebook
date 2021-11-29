1. ISN(Initial Sequence Number)初始序号的生成规律：1+2\*seconds\*64000 p235
2. MSS的默认大小：536字节，双方指定了最小的MSS并不意味着在传输过程中不会分包，如果某一段链路的MSS小于双方协商的数值仍然会分包，MSS的协商发生在第一和第二次握手之间。
3. TCP常见的选项就是MSS还有window scale，window scale是window的偏移量，能够让发送方表示更大的接收缓冲。
4. TCP TIME_WAIT的大小2MSL，其主要作用有两个
   1. 让主动结束方的最后一个ack确保让对方收到。
   2. 让一些之前慢到达的报文在链路中消逝或被丢弃，避免被认为是新的连接。 p243
5. SO_REUSEADDR的作用，使得处在TIME_WAIT状态的端口重新绑定，但这并不意味着之前的socket pair连接会成功，如果socket pair完全一样并且新连接的初始序号与之前最后的序号一样那么仍然会失败。当初始序号发生变化后且上一次是服务端（监听端）主动关闭才会成功。 p245
6. TCP在收到一个序号错误的报文段会怎么样，立刻终止这个连接并发送RST报文 ref：A packet with RST flag set aborts (resets) the connection. A SYN to a non-listening port will be ack'ed, but the ACK packet will have the RST flag set. The initiating party will immediately abort. A packet with RST set can be sent during a communication, for example, if an invalid sequence number is received. The connection is immediately aborted by both parties. A RST is not ACK'ed.
   https://www.cs.miami.edu/home/burt/learning/Csc524.032/notes/tcp_nutshell.html
7. RST报文发送的场景：
   1. 客户端尝试连接一个服务器并不存在监听的端口。服务器会返回RST报文。
   2. 利用SO_LINGER选项来异常终止连接，而不是四次挥手，客户端调用close后直接发送RST报文，服务器收到后并不回复。
   3. 半打开的场景，发生在某一端突然崩溃，但另一端却仍认为连接在继续，等到重启后，另一端传输数据会接收RST报文。 p249 注：与半关闭的区别。
8. 同时打开（发送syn）的场景：两个终端同时连接同一个端口号，这会导致四次握手，而不是三次。比较少见。 p250
9. 同时关闭（发送fin）的场景：两个终端同时发送fin报文，经过四次握手后也同时进入TIME_WAIT状态。也比较少见。 p251
10. accept函数backlog的值是指服务器所能接收的最大未accept的连接数。当服务器的最大连接数满了之后，仅仅丢弃该报文，并且期待客户端再次发送连接报文。  p259
11. 接收端的延迟确认机制：接收数据后并不立即ack，而是等待一段时间（200ms的TCP计时器），如果有数据或者计时器超时才会ACK。p267
12. 发送端的抑制小包发送的Nagle算法：当前者并未被ack时，延迟发送；当前未发送数据达到MSS，立即发送。
13. Nagle与延迟确认的相互作用：延迟确认会导致没有数据发送时，等到计时器触发才发送确认；而nagle也并不一定要积累完MSS大小的数据包才发送，等待对方的延迟ack到来也会发送。
14. TCP的PUSH标志：当发送缓冲区耗尽时会对接收方发送，现在的PUSH标志完全不需要用户控制；当窗口大小变化即使没有数据要发送也会给对方会送一个ACK以通知窗口变化来告知可以再次发送数据。
15. TCP的URGENT标志，几乎没用，紧急指针标记urgent数据的最后一个字节，而不是第一个字节。
16. 注意：send返回并不意味着已发送到对方，而是已成功拷贝入内核缓冲区中，不管是阻塞还是非阻塞。
17. TCP的7种定时器初探：
    1.  SYN定时器：建立连接时若syn或synack没有应答则需要重传。
    2.  保活计时器：若双方长时间没有数据的交互则会发送一个嗅探报文观测对方是否还存在。
    3.  FIN_WAIT2计时器，在对方ack己方的FIN之后若长时间没有FIN则直接中止这个连接。
    4.  重传定时器：发送的报文段没有被ack，则重传。其实以上三种属于特殊的重传定时器。
    5.  延迟应答计时器，这是打开了nagle算法的情况下才会启用。
    6.  坚持定时器：当看到接收窗口是0时提醒对方要更新窗口的大小
    7.  TIME_WAIT计时器：故名思意。