## go常用工具

### GO runtime查看

- GODEBUG=gctrace=1

- GODEBUG=scheddetail=1,schedtrace=1000

 - [go tool trace trace.out](https://segmentfault.com/a/1190000019736288)

## go的profile工具？

- CPU 分析
- 内存分析
- 阻塞分析
- 锁竞争分析

### go tool vet使用

- 检测如mutex copy的问题

## Reference

[实战Go内存泄露](https://segmentfault.com/a/1190000019222661)

[golang pprof使用](https://lrita.github.io/2017/05/26/golang-memory-pprof/)

[使用 pprof 和 Flame-Graph 调试 Golang 应用](https://www.cnblogs.com/upyun/p/8526925.html)

