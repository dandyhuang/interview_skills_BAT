# 开放性问题

### redis的setnx底层怎么实现的

### 判读40亿数字中是否有某个数字

- 首先40\*(10^8)\*4/1024/1024/1024)G=14.901G占用这么大内存。整数最大范围为2^32是42亿多。如果用位图那么40亿的数字，一个字节可以表示8个bit，需要占用的内存为40\*(10^8)/1024/1024/1024/8=0.46G

- 1个int占4字节即4*8=32位，只需要申请一个int数组长度为 `int tmp[(N/32)+1]`即可存储完这些数据，其中N代表要进行查找的总数,+1是因为如果N等于32，那么就没有申请数组大小了。
- 以34数字为例，那么34/32=1，这时候34就是在`tmp[1]`上。之后再对34%32=2，即在tmp[1]的第2位。所以tmp[1]第一位至于2可以表示为tmp[1]|=1<<2位。

- 如果让你写代码
