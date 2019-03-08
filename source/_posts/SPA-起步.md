---
title: Vue 单页项目 笔记
date: 2019-02-27 13:58:51
tags: #可以有多个标签
    - Vue   
    - Laravel
category:
    - 前端
thumbnail: https://cn.vuejs.org/images/logo.png
---

####  填充组件 <router-view> </router-view>

```js
//app.js中
require('./bootstrap');

window.Vue = require('vue');

import VueRouter from 'vue-router'
import router from './routes'

Vue.use(VueRouter)

new Vue({
    el: '#app',
    router:router
});

```
<!-- more -->
```js 
//新建路由文件 routes.js
//引入 模块
import VueRouter from 'vue-router'

//定义路由
let routes = [
    {
        path:'/',
        component: require('./components/Home')
    },
    {
        path:'/about',
        component: require('./components/About')
    }
    ....
    .....
    ......
]

//导出
export default new VueRouter({
    routes
})

```

> 安装模块： cnpm install [modulename] --save

> npm run watch 

> php artisan serve

#### 数据填充
``` shell
$ php artisan make:model Post -m
$ php artisan make:factory PostFactory --model=Post

//在tinker 中
$ factory(App\Post::class)->make();//测试
$ factory(App\Post::class,50)->create();//生成50条数据
```
```php
//factory 填充实例 （直接看$faker 类 即可）
        'title' => $faker->sentence,
        'body' => $faker->paragraph,
        'user_id' => factory(\App\User::class)->create()->id,
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'remember_token' => str_random(10),
//

```
#### Vue 路由
```js
let routes = [
    {
        path:'/about',
        component: require('./components/About')
    },
    {
        path:'/posts/:id',
        name: 'post',
        component: require('./components/Post')
    }
]

```

#### axios 方法调用例子
```js
axios.get('/api/posts/'+this.$route.params.id).then(response => {
                    this.post =  response.data;//对象格式时
                    this.post =  response.data.data;//数组格式时
            });
            
axios.post('/api/register',formData).then(response =>{
                    //跳转到confirm页面
                    this.$router.push({name:'confirm'})
              })            

```

#### 注意事项
- VUE文件命名规则 大写无符号如：RegisterForm
- 组件引用时 应该有- 例如：register-form
- @keydown="form.errors.clear($event.target.name)"，这个用于输入时清空input框地下的错误提示
- 


