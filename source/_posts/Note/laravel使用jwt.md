---
title: Laravel使用JWT
date: 2020-04-02
tags: [Note,php,laravel,jwt]
categories:
- [Note,PHP]
comments: false
---
# Laravel使用JWT

## 前言

laravel使用JWT有两种方法,一中是使用内置的Auth结合JWT定义的中间件进行认证,这种方法laravel社区有很多教程,这里不再细说，谈谈如何使用自定义JWT来完成用户认证
<!-- more -->
## JWT-oath

### 安装

```
#使用composer安装jwt-oath扩展
#当然也可以写进composer.json文件中
composer require tymon/jwt-auth
```

### 生成加密密匙

```
# 这条命令会在 .env 文件下生成一个加密密钥，如：JWT_SECRET=foobar
#注意.env文件默认是不会上传git的,为了你的同伴也可以正常使用认证,需要在.env.example中手动添加相同的字段
php artisan jwt:secret
```

#### 注册两个 Facade

这两个 Facade 并不是必须的，但是使用它们会给你的代码编写带来一点便利。

**config/app.php**

```
'aliases' => [
        ...
        // 添加以下两行
        'JWTAuth' => 'Tymon\JWTAuth\Facades\JWTAuth',
        'JWTFactory' => 'Tymon\JWTAuth\Facades\JWTFactory',
],
```

#### 修改 auth.php

**config/auth.php**

```
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'jwt',      // 原来是 token 改成jwt
        'provider' => 'users',
    ],
],
```

### 处理流程

[![mark](https://blogjpg.yanmy.top/blog/20200402/xwyk8GUazxXF.png?imageslim)](https://blogjpg.yanmy.top/blog/20200402/xwyk8GUazxXF.png?imageslim)

### 使用方法

#### 生成

```
<?php
//自定义的载荷填充
$customClaims = [
            'iss' => "http://market.sky31.com",
            'share' => md5($stu_id),
        ];
//利用JWT工厂类生成根据自定义的载荷生成payload
$payload = \JWTFactory::customClaims($customClaims)->make();
//调用Auth类的encode方法就可以生成token
$token = \JWTAuth::encode($payload);
//注意,此处的token是一个类,如何直接添加进response()->json()中将会报错,所以强制转换为String便可以当成字符床正常使用了
return (string)$token;
```

#### 验证

```
<?php
//认证可以写在中间件中
//首部use添加JWT的所有异常类
use Tymon\JWTAuth\Exceptions\JWTException;
use Tymon\JWTAuth\Exceptions\TokenExpiredException;
use Tymon\JWTAuth\Exceptions\TokenInvalidException;

//一定要使用try catch版快来验证token,token的验证失败,过期等等异常都是throw一个异常,不捕获异常程序直接死掉了,且错误信息不可以定义,就会导致代码不严谨
try {
 	//定义一个Auth类来验证token,首先将传过来的token绑定在类中
    $token = \JWTAuth::setToken($data['token']);
    //调用以下方法可以获取token中载荷的数据来使用
    $user_id=$token->payload()->get()['user_id'];
    //验证token,正确就会取出其中的载荷返回一个只含有载荷的对象,
    //失败就会抛出异常,失败的原因有许多,过期,数据格式不对,整个JWT过期
    $token->checkOrFail();
    //还有一个$token()->check方法可以验证token,返回值为bool,验证失败不会抛出异常而是返回false
    //捕获过期异常然后去刷新过期时间
} catch (TokenExpiredException $e) {
    try {
        //刷新token,返回值是更新过期时间的新token
        $new_token=$token->refresh();
        //更新数据库中的token
        $user= User::query()->where('user_id',$user_id)->first();
        $user->token=$new_token;
        $user->save();
        //将新token设置在request的参数中传给下一个路由
        $request->request->set("token",$new_token);
        //如果刷新不了,也过期了就返回错误码让用户重新登录
    }catch (TokenExpiredException $e){
        //msg是我自定义的用来返回json的,不用管
        return msg(3,__LINE__);
    }
    //token格式错误
} catch (TokenInvalidException $e) {
    return msg(403,$e->getMessage());
}catch (JWTException $e) {
    return msg(403,$e->getMessage());
}
//将request传给下下一个路由,到这里认证便成功了
return $next($request);
```

### 说明

JWT-oath扩展会在config目录下生成一个jwt.php,里面定义了JWT的相关配置

**注意:**这个扩展有两种过期方式,一种的token有效期,时间短,过期了需要刷新,一种是刷新有效期,时间长,只要在有效期内拿不在黑名单的旧token来刷新就可以,时间一长一短可以一定程度上防止token盗用

常用配置

```
//jwt有效期,默认是60分钟,单位是分钟
JWT_TTL 
//jwt的刷新有效期,单位是分钟,默认是20160(14天)
JWT_REFRESH_TTL
//注意以上两个时间不可以用60*24这种格式,需要设置为120这样的整数
//必须填充的载荷,在你自己设定载荷的时候必须全部手动填充,不然会报错,无法生成,不想自定义的可以注释掉
    'required_claims' => [
        'iss',
//        'iat',
//        'exp',
//        'nbf',
//        'sub',
//        'jti',
    ], 
//iss默认是使用用于请求的api路径
//设置黑名单,当为false时,刷新后生成新token,旧token仍然可以用
//true验证时就会抛出黑名单异常信息
blacklist_enabled
```

建议是修改.env中的环境变量,不要修改jwt.php文件,因为jwt.php文件中的配置项大都是从.env中获取的,没有获取到会用默认值