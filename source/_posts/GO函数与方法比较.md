---
title: GO函数与方法比较
date: 2021-09-06 09:09:27
tags:
    - golang   
category:
    - 后端
thumbnail: /thumbnails/lv3.png
---
>很多初学者在学习`方法`的时候都有这样的一个疑问,这写成函数不也可以吗？
>我们先看个例子

```golang
type Vertex2 struct {
    X, Y float64
}

func (v Vertex2) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func AbsFunc(v Vertex2) float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    v := Vertex2{3, 4}
    fmt.Println(v.Abs())
    fmt.Println(AbsFunc(v))

    p := &Vertex2{4, 3}
    fmt.Println(p.Abs())
    fmt.Println(AbsFunc(*p)) //AbsFunc(p) 就不行
}
```

- 通过以上样例不难发现：若接收一个值 ，函数的话 就必须是值类型 指针类型不可以；
但若是方法就可以接收指针，也可以是值类型，因为方法会默认将以上的`p.Abs()` 变为 `(*p).Abs()`

- 除此之外，指针接收者也有好处：直接修改接收者的值；可以避免每次调用方法时复制该值，在大结构体时显得更高效