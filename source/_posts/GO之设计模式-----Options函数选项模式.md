---
title: GO之设计模式-----Options函数选项模式
date: 2021-09-06 09:23:49
tags: 
    - golang   
category:
    - 后端
thumbnail: /thumbnails/lv2.png
---
> 场景一：因为Go语言没有构造函数、方法重载之说,声明一个结构体后
- 如何优雅地初始化内部的值(可选|默认)呢？
- 结构体更新了添加了很多新字段，我们还需要更改之前的代码？

>
#### 1、之前的初始化方式

```golang
type User struct {
    ID int64
    Name string
    Address string
    gender  int8
}

func NewUser(id int64,name string) *User{
    return &User{
        ID:id,
        Name: name,
        ... //这些可选参数 并且不方便扩展
    }
}
```

#### 2、选项Options模式
 <!-- more -->
```golang
// 定义一个OptionsFunc 返回setValue的方法
type OptionsFunc func(*User)

func NewUserOfOptions(user *User,options ...OptionsFunc) *User {
    for _, option := range options {
        option(user)
    }
    return user
}

// 可以设置每个字段的OptionsFunc。WithSomething 为命名规范
func WithAddress(address string) OptionsFunc {
    return func(user *User) {
        user.Address = address
    }
}
func WithGender(gender int8) OptionsFunc {
    return func(user *User) {
        user.Gender = gender
    }
}

type User struct {
    ID int64
    Name string
    Address string
    Gender  int8
}

func NewUser(id int64,name string) *User{
    return &User{
        ID:id,
        Name: name,
    }
}
func main() {
    user := NewUser(22,"enzo")
    fmt.Printf("User:%+v",user)
    //拥有默认值 并可选set某个值
    user = NewUserOfOptions(user,WithAddress("天津"),WithGender(1))
    fmt.Printf("User:%+v",user)
}

```

> 场景二：GO的函数无法设置默认值，所有参数全是必填。当一个函数的参数很多时，维护起来就不方便了。
>


#### 3、此种设计方式可以看出会增加许多代码，实现成本会变得比较高。所以在开发过程中应该去权衡利弊是否真的适合使用。
gRPC框架和[go-micro](https://github.com/asim/go-micro/blob/master/options.go)中就很多options模式。


#### 4、References

- https://studygolang.com/articles/12329
- https://segmentfault.com/a/1190000013653494