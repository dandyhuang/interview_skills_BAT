
### 交替打印
```
func main() {
    // ch用来同步两个协程交替执行
    ch := make(chan int)
    // ch_end用来阻塞主程序，让两个协程可以执行完
    ch_end := make(chan int)
    go func() {
        for i := 1; i <= 100; i++ {
            ch <- 1
            if i % 2 == 0 {
                fmt.Println(i)
            }
        }
        ch_end <- 1
    }()
    go func() {
        for i := 1; i <= 100; i++ {
            <-ch
            if i % 2 != 0 {
                fmt.Println(i)
            }
        }
    }()
    <-ch_end
}
```

### 实现超时机制
```
func TimeOut(timeout time.Duration) {
    ch_to := make(chan bool, 1)
    go func() {
        time.Sleep(timeout)
        ch_to <- true
    }()

    ch_do := make(chan int, 1)
    go func() {
        time.Sleep(3e9)
        ch_do <- 1
    }()

    select {
    case i := <- ch_do:
        fmt.Println("do something, id:", i)
    case <-ch_to:
        fmt.Println("timeout")
        break
    }
}
```



### 迭代器实现
```
// 迭代器
func iterator(iterable []interface{}) chan interface{}{
    yield := make(chan interface{})
    go func() {
        for i := 0; i < len(iterable); i++ {
            yield <- iterable[i]
        }
        close(yield)
    }()
    return yield
}

// 获取下一个元素
func next(iter chan interface{}) interface{} {
    for v := range iter {
        return v
    }
    return nil
}

func main() {
    nums := []interface{}{1, 2, 3, 4, 5}
    iter := iterator(nums)
    for v := next(iter); v != nil; v = next(iter) {
        fmt.Println(v)
    }
}
```

###  golang互斥锁的两种实现
- 用Mutex实现
```
package main
import (
    "fmt"
    "sync"
)
var num int
var mtx sync.Mutex
var wg sync.WaitGroup
func add() {
    mtx.Lock()
    defer mtx.Unlock()
    defer wg.Done()
    num += 1
}
func main() {
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go add()
    }
    wg.Wait()
    fmt.Println("num:", num)
}
```
- 使用chan实现

```
package main
import (
    "fmt"
    "sync"
)
var num int
func add(h chan int, wg *sync.WaitGroup) {
    defer wg.Done()
    h <- 1
    num += 1
    <-h
}
func main() {
    ch := make(chan int, 1)
    wg := &sync.WaitGroup{}
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go add(ch, wg)
    }
    wg.Wait()
    fmt.Println("num:", num)
}
```

