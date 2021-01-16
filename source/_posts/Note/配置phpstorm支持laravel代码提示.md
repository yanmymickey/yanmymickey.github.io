---
title: 配置phpstorm支持laravel代码提示
date: 2020-01-12
tags: [Note,php,laravel]
categories:
- [Note,PHP]
comments: false
---

# 配置phpstorm支持laravel代码提示

## 前言

安装和使用laravel-ide-helper和phpstorm中的laravel插件
<!-- more -->
## laravel插件

1. 在file->settings->plugins中搜索laravel并安装
2. 在file->settings->languages & Frameworks->php->laravel中启用,勾选enabled选择框
3. 两种情况配置views等提示
   - 若laravel为phpstorm打开的项目目录,则默认配置即可用
   - 若laravel为phpstorm打开的项目目录的子目录,则需要配置
     - file->settings->languages & Frameworks->php->laravel->views/templates
     - 仿照默认的配置,从根目录开始书写路径
       - 例如D:\phpstudy_pro\WWW\blog\app\views

## laravel-ide-helper

需要借助composer

### composer 安装 laravel-ide-helper

- `composer require --dev barryvdh/laravel-ide-helper`

### 配置

在 `「config/app.php」`的 `「providers」`数组中加入

```
Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider::class
```

> Laravel 版本小于 5.5 的话，需要注册提供者，5.5之后的版本laravel加入了自动注册

**注:**如果只在开发环境中安装「larave-ide-helper」，可以在`「app/Providers/AppServiceProvider.php」`的`「register」`方法中写入下面代码：

```
public function register()
{
    if ($this->app->environment() !== 'production') {
        $this->app->register(\Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider::class);
    }
    // ...
}
```

### 导出配置文件

（如果默认配置就满足需求了，也可以忽略这一步）

```
php artisan vendor:publish --provider="Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider" --tag=config
```

### 使用

```
php artisan ide-helper:generate - 为 Facades 生成注释
php artisan ide-helper:models - 为数据模型生成注释
php artisan ide-helper:meta - 生成 PhpStorm Meta file
```

#### 自动为Facades 生成注释

终端输入以下命令

```
php artisan ide-helper:generate
```

**注:** 如果存在文件 「bootstrap/compiled.php」 需要先删除， 可以在生成文当前运行 php artisan clear-compiled。

#### 自动为模型生成注释

为所有模型生成注释

```
php artisan ide-helper:models
#这时会出现询问：
Do you want to overwrite the existing model files? Choose no to write to _ide_helper_models.php instead? (Yes/No):  (yes/no) [no]:
```

**注:**输入 yes 则会直接在模型文件中写入注释，否则会生成「_ide_helper_models.php」文件。建议选择 yes，这样在跟踪文件的时候不会跳转到「_ide_helper_models.php」文件，不过这么做最好对模型文件做个备份，至少在生成注释之前用 git 控制一下版本，以防万一。

> **提示：** 为模型生成字段信息必须在数据库中存在相应的数据表，不要生成 migration 还没运行 migrate 的时候就生成注释，这样是得不到字段信息的。

#### 自动为链式操作注释

这是什么意思呢？举个例子，在 migration 文件中经常可以看见这样的代码：

```
$table->string('email')->unique();
```

这时就算调用过了 `php artisan ide-helper:generate`，在调用像 `->unique()` 这样的链式操作的时候也无法实现代码提示.

需要将配置文件(`ide-help.php`,可以使用phpstorm的Find->Find in path 查找 include_fluent)`'include_fluent' => false` 修改为 `'include_fluent' => true`，

重新运行 `php artisan ide-helper:generate`。

#### 生成 .phpStorm.meta.php

可以生成一个`.phpStorm.meta.php`文件去支持工厂模式.

对于 Laravel, 这意味着我们可以让 PhpStorm 理解我们从 IoC 容器中解决了什么类型的对象。例如：事件将返回一个`「Illuminate\Events\Dispatcher」`对象，利用 meta 文件您可以调用 `app('events')` 并且它将自动完成 Dispatcher 的方法。

```
app('events')->fire();
\App::make('events')->fire();

/** @var \Illuminate\Foundation\Application $app */
$app->make('events')->fire();

// When the key is not found, it uses the argument as class name
app('App\SomeClass');
```

> **提示:**可能需要重启 Phpstorm 使 .phpStorm.meta.php 文件生效。

#### 自动运行 generate

想在依赖包更新是自动更新注释，可以在 composer.json 文件中做如下配置：

```
"scripts":{
    "post-update-cmd": [
        "Illuminate\\Foundation\\ComposerScripts::postUpdate",
        "php artisan ide-helper:generate",
        "php artisan ide-helper:meta"
    ]
}
```

#### 代码提示

当想提示views文件,models文件时可以使用默认快捷键`ctrl+space`

**注:**但可能会被切换输入法等占用,所以可以更改输入法快捷键或者phpstorm中的快捷键

##### 修改phpstorm中的快捷键方法

在file->settings->keymap中搜索completion,修改冲突快捷键