# interface

`Go`语言中，interface可以分为两类

- 使用 [`runtime.iface`](https://draveness.me/golang/tree/runtime.iface) 结构体表示包含方法的接口
- 使用 [`runtime.eface`](https://draveness.me/golang/tree/runtime.eface) 结构体表示不包含任何方法的 `interface{}` 类型；

### eface

- 空接口的主要目的有 2 个，一是实现"泛型"，二是使用反射。

- interface还可以作为一中通用的类型，其他类型变量可以给interface声明的变量赋值。
- interface 可以作为一种数据类型，实现了该接口的任何对象都可以给对应的接口类型变量赋值

```go
type eface struct { // 16 字节
    _type *_type
    data  unsafe.Pointer
}
```

### iface

- `Go`中不需要显式的声明实现了某个接口，只要实现了其中的所有方法，就默认为实现了该接口

- interface 是方法或行为声明的集合
- interface接口方式实现比较隐性，任何类型的对象实现interface所包含的全部方法，则表明该类型实现了该接口。

**如果实现了接收者是值类型的方法，会隐含地也实现了接收者是指针类型的方法**，因为有了指针总是能得到指针指向的值是什么



### 课外知识补充

Plan9 汇编常见寄存器含义：
BP: 栈基，栈帧（函数的栈叫栈帧）的开始位置。
SP: 栈顶，栈帧的结束位置。
PC: 就是IP寄存器，存放CPU下一个执行指令的位置地址。
TLS: 虚拟寄存器。表示的是 thread-local storage，Golang 中存放了当前正在执行的g的结构体。