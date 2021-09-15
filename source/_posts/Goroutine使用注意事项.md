---
title: 请勿滥用Goroutine
date: 2021-09-09 19:09:27
tags:
    - golang   
category:
    - 后端
thumbnail: /thumbnails/jscw1.png
---

### Goroutine泄漏|滥用Goroutine|Goroutine 弊端|Goroutine面试

> Go中 goroutine创建成本实在过于低，随便开启goroutine 和 channel 到底会有什么问题？

#### 泄漏原因

1. goroutine中的读写操作 一直被阻塞
2. goroutine中的业务逻辑死循环，资源无法释放
3. 锁机制使用不当


#### 具体原因：
 <!-- more -->
1. Channel
   1. 发送不及时
   2. 接收不发送
   3. nil channel

2. 第三方包没有超时设置等原因 导致 goroutine 一直等待，一直阻塞 （例如 http.Client ）

```go
//建议添加上超时时间
httpClient := http.Client{
       Timeout: time.Second * 15,
}

//限流

//熔断
```

3. 互斥锁Mutex解锁不及时 
4. 同步锁 wg.Add与wg.Done数量不一致

```go
//以下为建议写法
 var wg sync.WaitGroup
    for i := 0; i < v; i++ {
        wg.Add(1)
        defer wg.Done()
        fmt.Println("do something")
    }
    wg.Wait()
```



#### 控制Goroutine的方法

1. Context包

>  主要作用就是在不同的 `goroutine` 之间同步请求特定的数据、取消信号以及处理请求的截止日期。
>
> 更多地使用在`超时控制`和`取消`。

2. Channel

- 使用select的时候 要带上default，避免出现nil channel问题一直阻塞；
- 发送者调用Close(ch),而不是让接收者去调用

3. errgroup包 —— 多用于并发的任务分发

errgroup的原理其实就是Context、Waitgroup、Sync.once



### 建议

- 任务分发类的并发操作交给Channel去处理(要注意 buffer channel 和 unbuffer channel使用场景)
- 更改值的并发操作 建议交给**同步锁，sync和automic包**，防止 `Happens before`,`data race`，因为此类并发操作相对任务分发就比较轻量了，一般任务分发中会涉及请求或者查询，但是更新值这类更多的是准确同步.

> 使用Goroutine的时候应该问自己两个问题，才不易发生内存泄漏
>
> 1. 它什么时候执行？
> 2. 我能使用什么方式将其停止？



#### References

- https://eddycjy.com/posts/go/goroutine-leak/
- https://blog.csdn.net/qq_39397165/article/details/119881256

