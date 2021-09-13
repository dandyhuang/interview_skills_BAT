grpc.Invoke()最终调用的是invoke()方法，invoke()表示一次请求过程，包括：

新建clientStream对象，标记调用“/proto.Hello/SayHello” Method，监听用户是否关闭此连接
发送请求SendMsg()，底层调用http2client.Write方法发送req。底层调用函数依次为clientStream.SendMsg()->csAttempt.sendMsg()->http2Client.Write()
接收响应数据RecvMsg()，底层调用函数关系依次为RecvMsg()->recvMsg()->recv()->Stream.Read()->io.ReadFull()->io.ReadAtLeast() ->transportReader.Read()->recvBufferReader.Read()->recvBufferReader.readClient()。readClient从接收数据channel里读取数据。接收数据channel数据依赖http2协议写入。