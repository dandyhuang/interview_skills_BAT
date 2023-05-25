#### 套接字编程中epoll如何处理？

1. 对于监听(listend)的 sockfd，最好使用水平触发模式，边缘触发模式会导致高并发情况下，有的客户端会连接不上。如果非要使用边缘触发，可以用 while 来循环 accept()。
2. 对于读写的 connfd，水平触发模式下，阻塞和非阻塞效果都一样，因为在阻塞模式下，如果数据读取不完全则返回继续触发，反之读取完则返回继续等待。全建议设置非阻塞。
3. 对于读写的 connfd，边缘触发模式下，必须使用非阻塞 IO，并要求一次性地完整读写全部数据。如果没有全部读完数据，会有问题。因为非阻塞 IO 如果没有数据可读时，会立即返回，并设置 errno。这里我们根据 EAGAIN 和 EWOULDBLOCK 来判断数据是否全部读取完毕了，如果读取完毕，就会正常退出循环了。



#### Socket/Epoll对socket错误码正确处理

1. (send/sendto)和(recv/recvfrom)不要把错误(EINTR/EAGAIN/EWOULDBLOCK)当成Fatal.
2. connect不要把错误(EINTR/EINPROGRESS/EAGAIN)当成Fatal.
3. accept不要把错误(EINTR/ECONNABORTED/EPROTO)当成Fatal.
4. epoll_wait对EPOLLERR/EPOLLHUP事件和socket error处理.
5. recv/recvfrom接收空数据需要区分.

#### 字节序大小端问题？



#### epoll是否使用了mmap



[epoll早期使用了mmap， 后面没有用了?](https://segmentfault.com/q/1010000022582226)

[字节序问题之大小端模式讲解](https://www.cnblogs.com/renhui/p/13600572.html)