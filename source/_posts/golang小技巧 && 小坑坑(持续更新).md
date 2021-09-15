---
title: golang 小技巧 && 小坑坑 （持续更新ing）
date: 2021-09-03 11:10:21
tags: 
    - golang   
category:
    - 后端
thumbnail: /thumbnails/rencheheyi.png
---
### Issue 1
> 计算用户名长度
### 要点 ：
- len计算字节长度，中文=3个字节 emoji还有可能4个字节，
所以保险起见计算字符串长度时应该使用 `RuneCountInString`
```golang
func main()  {
    fmt.Printf("我是中国人的字符长度:%d \n",utf8.RuneCountInString("我是中国人"))
    fmt.Printf("我是中国人的字符长度:%d \n",len("我是中国人"))
}
```
### Issue 2
> For 和 Clourse ：for循环或者遍历执行完了闭包函数才开始执行，导致i的值为遍历中最后一个元素
>
> for 循环下 注意 函数 参数的作用域
 <!-- more -->
### 解决方案 ：
- 將i作为一个参数传给gouroutine，这样每个i都会独立计算并保存到goroutine的栈中
```golang
    //错误
    for i := 0; i < 5; i++ {
        go func() {
            fmt.Printf("i=%d \n",i)
            /*
            此时编辑器也会给出提示：
            Loop variables captured by 'func' literals in 'go' statements might have unexpected values
            */
        }()
    }
    //正确
    for i := 0; i < 5; i++ {
        go func(i int) {
            fmt.Printf("clourse---i=%d \n",i)
        }(i)
    }
    //或者不用闭包
    for i := 0; i < 5; i++ {
        go delay(i)
    }
    func delay(i int)  {
        fmt.Printf("delay()---i=%d \n",i)
    }
```
```golang
package main

import "fmt"

func createPrintFunc(v int) func() {
	return func() {
		fmt.Println("fmt.Println in function with v copied as arg", v)
	}
}

func main() {
	a := []int{1, 2, 3, 4, 5}
	var funcArr []func()
	for _, v := range a {
		fmt.Println("fmt.Println", v) // 这里立刻打印出v，是对的

		funcArr = append(funcArr, func() {
			fmt.Println("fmt.Println in function", v) // 这里做了一个函数，在循环结束之后再执行，所以打印出来的都是5
		})

		vv := v // 这里复制了一份到vv，vv的作用域是{}内，因此有5份拷贝，是对的。
		funcArr = append(funcArr, func() {
			fmt.Println("fmt.Println in function with copy of v", vv)
		})

		funcArr = append(funcArr, createPrintFunc(v)) // 这里通过传参的方式，虽然参数在createPrintFunc里面也叫v，但是其实是v的一份拷贝。
	}
	for _, f := range funcArr {
		f()
	}
}
```
### Issue 3
> 使用&取地址时，默认会对该类型实例化一次，&实例化等同于new关键字返回指针类型
```golang
    //一般使用以下方式初始化一个结构体
    func newStaticNode(path string) *node {
    	return &node{
    		children: make([]*node, 0, 2),
    		matchFunc: func(p string, c *Context) bool {
    			return path == p && p != "*"
    		},
    	}
    }
```
### Issue 4
> Context 借用邓明老师的理解：爷爷告诉爸爸 30分钟后吃饭 不来抽你；爸爸告诉我 30分钟后吃饭 不来抽你；我告诉儿子 30分钟吃饭 不来抽你；
>爷爷一看到30分钟了，没人来 都挨抽了。
>
>长请求都应该有超时限制，需要在这个过程传递这个超时。好比一个请求调用四个RPC，当RPC2出问题时就通知RPC3,4 停止运行，不要浪费时间和资源
### 解决方案 ：
- 用信号的方式来通知请求
- 用 channel 来通知请求
- 超时
### 注意： 
需要注意```WithValue```的坑 （类似继承 父类无法访问子类的方法）

```golang
func WithValue() {
    parentKey := "parent"
    parent := context.WithValue(context.Background(), parentKey, "this is parent")
    
    sonKey := "son"
    son := context.WithValue(parent, sonKey, "this is son")
    
    // 尝试从 parent 里面拿出来 key = son的，会拿不到
    if parent.Value(parentKey) == nil {
        fmt.Printf("parent can not get son's key-value pair")
    }
    
    if val := son.Value(parentKey); val != nil {
        fmt.Printf("parent can not get son's key-value pair")
    }
    
}

```

- [ ] Data race
- [ ] sync pool 复用Context 第四节课ppt
- [ ] filter 和 factory
- [x] 勾选

> 不建议在生产环境调用Log.Fatal函数，因为它内部会执行os.Exit（1）无条件地终止了程序，defer也不会被调用。除了 main函数和init

> 写这个channel的人才能去调用panic。不然就会出现读的时候 panic