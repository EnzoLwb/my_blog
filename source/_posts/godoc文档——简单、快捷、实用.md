---
title: go doc 初次见面
date: 2021-09-09 09:09:27
tags:
    - golang   
category:
    - 后端
thumbnail: /thumbnails/lv4.png
---

1. go get golang.org/x/tools/cmd/godoc

> demo
 <!-- more -->
```go
package queue

type Queue []int

//尾部插入
func (q *Queue) Push(val int)  {
	*q = append(*q,val)
}

//弹出首个元素
func (q *Queue) Pop() int {
	head := (*q)[0]
	*q = (*q)[1:]
	return head
}

//队列是否为空
func (q *Queue) IsEmpty() bool  {
	return len(*q) == 0
}

```



2. 选择生成文档的包 *go doc* queue ——这里我们选择上述的queue 自创建的包

3. 查看文档 **godoc** -http :6060 

![image-20210913151428590](/thumbnails/godoc.png)