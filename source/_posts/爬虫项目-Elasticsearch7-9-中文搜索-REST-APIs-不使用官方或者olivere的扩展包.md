---
title: '[爬虫项目] Elasticsearch7.9 中文搜索 REST APIs (不使用官方或者olivere的扩展包)'
date: 2020-09-08 16:08:33
tags:
    - Golang
    - elasticsearch
    - 爬虫
category:
    - 后台
---
### 前言
> 学习golang也一个月时间了，你说巧不巧，领导正好找我谈话，说：文博啊，这边有个新项目要你做一下，很简单的，就是之前给xxx局做的一个爬虫项目，太老了也太单一了，不能重用，希望你能够在今年底完成一版本可复用的爬虫系统。以后可以做成SASS什么的。 主要就是一些文件的索引和拉取。。。balabala .PHP实现这些也很容易的。

> 我说：哈哈好的，既然时间上不是很紧张，那我这次打算使用golang来写这个系统，最近也在学习分布式并发爬虫项目。大文件内容的索引记录，肯定不能使用关系型数据库了。咱就用es吧~

> 领导：那肯定，就用es就行了。

> > 这个 **[爬虫项目]** 系列文章会记录这个项目所涉及的一些技术重点和心得等。
#### 目的
> 项目还没有正式启动，因为还要总结一些需求，所以我先接触一下es，完成一些基本的语法操作。(说句可能丢人的话，3年php的经验并没有接触过es，所以这篇是入门篇)
***
#####     可能你们的项目使用的是官方的es扩展包或者是star数最多的olivere的扩展包。这两个扩展包我都看了，olivere我也测试使用了，感觉还不是很顺手，并且我觉得不适合我这个golang新手(轮子是给熟练工使用的而不是实习生)，所以我就直接采用官方的REST Api的方式，来构建我们的es检索存储系统，其实以后我也打算这种方式因为他真的太方便了，你可以随便改变的输入参数，熟悉es的操作等等。
***
## 废话不多说 上干货
1.  docker 部署es + 添加中文分词（不使用中文分词的话 你在搜索中文的时候它的分词方式是 每个字都算一个keyword；添加以后它就niubi了，但是你得会配置 字段映射等）
- 方式一 ：手动安装 
```shell
docker pull elasticsearch:7.9.1
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.1/elasticsearch-analysis-ik-7.9.1.zip 
#下载速度慢的话就去地址下载好然后 拷贝到容器内 进行离线安装 不赘述了
docker run -p 9200:9200 -p 9300:9300 -v E:/gocode/es/data/nodes:/usr/share/elasticsearch/data/nodes -v  E:/gocode/es/logs:/usr/share/elasticsearch/logs  --name es  -e "discovery.type=single-node" elasticsearch:7.9.1
### 先在本地建立目录哦 
```
[参考这里](https://www.cnblogs.com/szwdun/p/10664348.html "参考这里")
- 方式二 ：拉取docker images 不只有中文分词的 还有带拼音的(pinyin的版本上网自行搜索 因为我这个爬虫系统未使用到拼音搜索方式 所以就先不花费时间搞了)

```shell
docker pull ar414/elasticsearch-7.9-ik-plugin
docker run -p 9200:9200 -p 9300:9300 -v \ E:/gocode/ikes/data:/var/lib/elasticsearch -v\ E:/gocode/ikes/log:/var/log/elasticsearch --name es_ik \  -e "discovery.type=single-node" ar414/elasticsearch-7.9-ik-plugin
```

> 两种方式各有好处 前者可以让你熟悉一些es的文件目录和配置，以及插件的安装方式；后者就是方便快捷 直接部署

``` shell
访问 localhost 9200端口 不报错 你就成功安装了
```
2. 直接上代码 根据官方文档以及插件文档编写的

```go
package main
import (
	"fmt"
	"io/ioutil"
	"net/http"
	url2 "net/url"
	"strings"
)

//创建 index curl -XPUT http://localhost:9200/index
const url = "http://localhost:9200/"
func createIndex(indexName string) {
	client := http.Client{}
	req, _ := http.NewRequest(http.MethodPut, url+indexName, nil)
	resp, err := client.Do(req)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	//不要头部的信息 只需要判code信息即可
	if resp.StatusCode != http.StatusOK {
		fmt.Errorf("error:status code %d", resp.StatusCode) //fmt.Errorf()
	}
	all, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}
	fmt.Print(string(all)) //{"acknowledged":true,"shards_acknowledged":true,"index":"ikes2"}
}

/* 创建映射  配置字段映射
curl -XPOST http://localhost:9200/index/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "content": { 你的字段名称 
                "type": "text", text date 等等
                "analyzer": "ik_max_word",最细度分词
                "search_analyzer": "ik_smart" 粗度分词
            }
        }
}
*/
func mapping(indexName string) {
	// json
	contentType := "application/json"
	data := `{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            },
			"ep": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            }
        }
	}`
	resp, err := http.Post(url+indexName+"/_mapping", contentType, strings.NewReader(data))
	if err != nil {
		fmt.Println("post failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("get resp failed,err:%v\n", err)
		return
	}
	fmt.Println(string(b)) //{"acknowledged":true}
}

/* 录入数据
curl -XPOST http://localhost:9200/index/_create/1 -H 'Content-Type:application/json' -d'
{"content":"美国留给伊拉克的是个烂摊子吗"}
*/
func inputData(indexName string, data string, id string) {
	// json
	contentType := "application/json"
	var inputUrl string
	if id != "0" {
		inputUrl = url + indexName + "/_create/" + id
	} else {
		inputUrl = url + indexName + "/_doc" //这里就是根据es官方文档编写 找半天不用手写id的添加方式。。要是直接post /index 他就会报错 应该就是版本更新的问题
	}
	resp, err := http.Post(inputUrl, contentType, strings.NewReader(data))
	if err != nil {
		fmt.Println("post failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("get resp failed,err:%v\n", err)
		return
	}
	fmt.Println(string(b)) //{"acknowledged":true}
	//{"_index":"ikes2","_type":"_doc","_id":"JKZWbHQB8wLP38GV5ldD","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":2,"_primary_term":1}
}

//测试分词效果
//curl -X POST "http://localhost:9200/your_index/_analyze?pretty" -H 'Content-Type: application/json' -d'
func analyze(indexName string) {
	// json
	contentType := "application/json"
	data := `{
		  "analyzer": "ik_max_word",
		  "text":     "laravel天下无敌"
		}`
	resp, err := http.Post(url+indexName+"/_analyze", contentType, strings.NewReader(data))
	if err != nil {
		fmt.Println("post failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("get resp failed,err:%v\n", err)
		return
	}
	fmt.Println(string(b)) //{"tokens":[{"token":"laravel","start_offset":0,"end_offset":7,"type":"ENGLISH","position":0},{"token":"天下无敌","start_offset":7,"end_offset":11,"type":"CN_WORD","position":1},{"token":"天下","start_offset":7,"end_offset":9,"type":"CN_WORD","position":2},{"token":"无敌","start_offset":9,"end_offset":11,"type":"CN_WORD","position":3}]}
}

/* 搜索
curl -XPOST http://localhost:9200/index/_search  -H 'Content-Type:application/json' -d'
{
    "query" : { "match" : { "content" : "中国" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}*/
func highLightSearch(indexName string, search string) {
	// json
	contentType := "application/json"
	data := `{
		"query" : { "match" : { "content" : "` + search + `" }},
		"highlight" : {
			"pre_tags" : ["<tag1>", "<tag2>"],
			"post_tags" : ["</tag1>", "</tag2>"],
			"fields" : {
				"content" : {}
			}
		}
	}`
	resp, err := http.Post(url+indexName+"/_search", contentType, strings.NewReader(data))
	if err != nil {
		fmt.Println("post failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("get resp failed,err:%v\n", err)
		return
	}
	fmt.Println(string(b))
}

//http://localhost:9200/ikes2/_search?q=context:中国
func search(indexName string, attr string, search string) {
	var attrSearchUrl string
	if attr != "" {
		attrSearchUrl = attr + ":" + search
	} else {
		attrSearchUrl = search
	}
	resp, err := http.Get(url + indexName + "/_search?q=" + url2.QueryEscape(attrSearchUrl)) //需要url转义 不然中文搜索不到哦
	if err != nil {
		fmt.Println("post failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("get resp failed,err:%v\n", err)
		return
	}
	fmt.Println(string(b))
}
func main() {
	createIndex("ikes2")
	mapping("ikes2")
	analyze("ikes2")
	inputData("ikes2", `{"content":"明天星期几","ep":"今天是星期三"}`, "0")
	highLightSearch("ikes2", "烂渔船")
	//search("ikes2", "ep", "今天")
}
```
3. 项目中的使用 等我下一篇后续文章。。。（还没开启项目呢）

