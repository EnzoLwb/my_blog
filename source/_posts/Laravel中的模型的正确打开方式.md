---
title: Laravel中模型的正确打开方式
date: 2019-02-27 17:29:52
tags:   
    - Laravel
category:
    - 后台
thumbnail: /thumbnails/hyld2.png
---
> $casts 属性应是一个数组，且数组的键是那些需要被转换的字段名，值则是你希望转换的数据类型。支持转换的数据类型有： integer，json，float，double，string，boolean，object，array，collection，date，datetime 和 timestamp。

```php
protected $casts = [
    'closed'    => 'boolean',
    'reviewed'  => 'boolean',
    'address'   => 'json',
    'ship_data' => 'json',
    'extra'     => 'json',
];
```
> 模型的监听事件 有creating,created,updating,updated,saving,saved,deleting,delete

```php
protected static function boot()
{
    parent::boot();
    // 监听模型创建事件，在写入数据库之前触发
    static::creating(function ($model) {
        // 如果模型的 no 字段为空
        if (!$model->no) {
            // 调用 findAvailableNo 生成订单流水号
            $model->no = static::findAvailableNo();
            // 如果生成失败，则终止创建订单
            if (!$model->no) {
                return false;
            }
        }
    });
}
```
<!-- more -->
```php

// 指明这两个字段是日期类型
protected $dates = ['not_before', 'not_after'];

public function index(Request $request)
{
    return view('user_addresses.index', [
        'addresses' => $request->user()->addresses,
    ]);
}

//路由传入的是id     
public function store(UserAddressRequest $request)
{   
//验证已经写在request中，使用address模型直接创建
    $request->user()->addresses()->create($request->only([
        'province',
        'city',
        'district',
        'address',
        'zip',
        'contact_name',
        'contact_phone',
    ]));

    return redirect()->route('user_addresses.index');
}
   
public function destroy(UserAddress $user_address)
{
    $user_address->delete();

    return redirect()->route('user_addresses.index');
}  

```
```php
$builder = Products::where('on_sale',1)......;
// 模糊搜索商品标题、商品详情、SKU 标题、SKU描述
    $builder->where(function ($query) use ($like) {
        $query->where('title', 'like', $like)
            ->orWhere('description', 'like', $like)
            ->orWhereHas('skus', function ($query) use ($like) {
                $query->where('title', 'like', $like)
                    ->orWhere('description', 'like', $like);
            });
    });
    //先用 $builder->where() 传入一个匿名函数，然后才在这个匿名函数里面再去添加 like 搜索，这样做目的是在查询条件的两边加上 ()，也就是说最终执行的 SQL 语句类似 select * from products where on_sale = 1 and ( title like xxx or description like xxx )
    //skus 为模型中的关联ORM
```

```php
//创建属性 用于视图显示等 返回img正确的url格式
public function getImageUrlAttribute(){
    return config('filesystems.disks.admin.url').$this->attributes['image'];
}
```
```html
//image_url字段本不存在 
<div class="img"><img src="{{$product->image_url}}" alt=""></div>
```
#### 模型关联关系举例
- 用户和收货地址 或者 文章和评论的关系 就是 一对多关系。使用belongsTo 和 hasMany
- 用户和收藏商品 之间的关系就是多对多关系。使用belongsToMany(商品，‘中间关系表（收藏表）’)
- 当更新关联时，可以使用 associate 方法。此方法会在子模型中设置外键：

```php
// 创建一个新的购物车记录
$item = new CartItem(['amount' => $amount]);
$item->user()->associate($user);
$item->productSku()->associate($skuId);
$item->save();
```