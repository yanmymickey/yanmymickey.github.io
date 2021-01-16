---
title: PHP反序列化
layout: categories
date: 2020-04-14
tags: [CTF, PHP]
categories:
- [CTF, PHP]
comments: false
---
# PHP反序列化

## 魔术方法

[PHP魔术方法](https://www.php.net/manual/zh/language.oop5.magic.php)

<!-- more -->

1、`__sleep()`

- `serialize()`函数会检查类中是否存在一个魔术方法 `__sleep()`。
- 如果存在，该方法会先被调用，然后才执行序列化操作。此功能可以用于清理对象，并返回一个包含对象中所有应被序列化的变量名称的数组。
- 如果该方法未返回任何内容，则 **NULL** 被序列化，并产生一个 **E_NOTICE** 级别的错误。

2、`__wake()`

- `unserialize()`会检查是否存在一个 `__wakeup()`方法。如果存在，则会先调用 *__wakeup* 方法，预先准备对象需要的资源。
- `__wakeup()`经常用在反序列化操作中，例如重新建立数据库连接，或执行其它初始化操作。

3、`__construct()`

- PHP 5 允行开发者在一个类中定义一个方法作为构造函数。具有构造函数的类会在每次创建新对象时先调用此方法，所以非常适合在使用对象之前做一些初始化工作。
- 如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用 **parent::__construct()**。如果子类没有定义构造函数则会如同一个普通的类方法一样从父类继承（假如没有被定义为 private 的话）。

4、`__destruct ()`

- PHP 5 引入了析构函数的概念，这类似于其它面向对象的语言，如 C++。析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。

5、`__toString()`

- [__toString()](https://www.php.net/manual/zh/language.oop5.magic.php#object.tostring) 方法用于一个类被当成字符串时应怎样回应。例如 *echo $obj;* 应该显示些什么。此方法必须返回一个字符串，否则将发出一条 **E_RECOVERABLE_ERROR** 级别的致命错误。
- 需要指出的是在 PHP 5.2.0 之前，`__toString()`方法只有在直接使用于 `echo` 或 `print` 时才能生效。
- PHP 5.2.0 之后，则可以在任何字符串环境生效（例如通过 `printf()`，使用 *%s* 修饰符），但不能用于非字符串环境（如使用 *%d* 修饰符）。
- 自 PHP 5.2.0 起，如果将一个未定义 `__toString()` 方法的对象转换为字符串，会产生 **E_RECOVERABLE_ERROR** 级别的错误。

6、`__invoke()`

- 当尝试以调用函数的方式调用一个对象时，`__invoke()`方法会被自动调用。

- 示例

  ```
  <?php
  class CallableClass 
  {
      function __invoke($x) {
          var_dump($x);
      }
  }
  $obj = new CallableClass;
  $obj(5);
  var_dump(is_callable($obj));
  ?>
  //样例输出    
  int(5)
  bool(true)
  ```

7、`__call()`和`__callStatic()`

- 方法重载
- 在对象中调用一个不可访问方法时，`__call()` 会被调用
- 在静态上下文中调用一个不可访问方法时，`__callStatic()` 会被`调用`
- `$name`参数是要调用的方法名称。`$arguments`参数是一个枚举数组，包含着要传递给方法 `$name` 的参数。

8、`__set()` `__get()` `__isset()` `__unset`

- 在给不可访问属性赋值时，`__set()` 会被调用。
- 读取不可访问属性的值时，`__get()` 会被调用。
- 当对不可访问属性调用 `isset()` 或 `empty()` 时，`__isset()` 会被调用。
- 当对不可访问属性调用 `unset()` 时，`__unset()` 会被调用。
- 参数 `$name` 是指要操作的变量名称。`__set()` 方法的 `$value` 参数指定了 `$name`变量的值。
- 属性重载只能在对象中进行。在静态方法中，这些魔术方法将不会被调用。所以这些方法都不能被声明为 `static`。从 PHP 5.3.0 起, 将这些魔术方法定义为 *static* 会产生一个**警告**。

9、`__set_state()`

- 自 PHP 5.1.0 起当调用 `var_export()`导出类时，此**静态**方法会被调用。

- 本方法的唯一参数是一个数组，其中包含按 *array(‘property’ => value, …)* 格式排列的类属性。

- 示例

  ```
  <?php
  class A{
      public $var1;
      public $var2;
      public static function __set_state($an_array){ // As of PHP 5.1.0
          $obj = new A;
          $obj->var1 = $an_array['var1'];
          $obj->var2 = $an_array['var2'];
          return $obj;
      }
  }
  
  $a = new A;
  $a->var1 = 5;
  $a->var2 = 'foo';
  
  eval('$b = ' . var_export($a, true) . ';'); 
  // $b = A::__set_state(array(
  //    'var1' => 5,
  //    'var2' => 'foo',
  // ));
  var_dump($b);
  
  //以上样例会输出：
  object(A)#2 (2) {
    ["var1"]=>
    int(5)
    ["var2"]=>
    string(3) "foo"
  }
  ```

10、`__clone()`

- 当对象被复制后，PHP 5 会对对象的所有属性执行一个浅复制（shallow copy）。所有的引用属性 仍然会是一个指向原来的变量的引用。

- 当复制完成时，如果定义了 `__clone()` 方法，则新创建的对象（复制生成的对象）中的`__clone()`方法会被调用，可用于修改属性的值（如果有必要的话）

- 示例

  ```
  <?php
  class SubObject{
      static $instances = 0;
      public $instance;
  
      public function __construct() {
          $this->instance = ++self::$instances;
      }
  
      public function __clone() {
          $this->instance = ++self::$instances;
      }
  }
  
  class MyCloneable{
      public $object1;
      public $object2;
  
      function __clone(){      
          // 强制复制一份this->object， 否则仍然指向同一个对象
          $this->object1 = clone $this->object1;
      }
  }
  
  $obj = new MyCloneable();
  
  $obj->object1 = new SubObject();
  $obj->object2 = new SubObject();
  
  $obj2 = clone $obj;
  
  print("Original Object:\n");
  print_r($obj);
  
  print("Cloned Object:\n");
  print_r($obj2);
  
  ?>
  ```

  ```
  //以上样例会输出
  Original Object:
  MyCloneable Object
  (
      [object1] => SubObject Object
          (
              [instance] => 1
          )
  
      [object2] => SubObject Object
          (
              [instance] => 2
          )
  
  )
  Cloned Object:
  MyCloneable Object
  (
      [object1] => SubObject Object
          (
              [instance] => 3
          )
  
      [object2] => SubObject Object
          (
              [instance] => 2
          )
  
  )
  //原始类经过克隆后
  //object1对象instance属性的值自增了1次,触发了__clone()方法,达到了修改属性值的目的
  //object2对象instance属性的值只是简单复制.仍然指向原来对象
  ```

## 小结

- 构造函数 `__construct` 对象被创建的时候调用
- 析构函数 `__destruct` 对象被销毁的时候调用
- 方法重载 `__call` 在对象中调用一个不可访问方法时调用
- 方法重载 `__callStatic` 在静态上下文中调用一个不可访问方法时调用
- 在给不可访问属性赋值时，`__set()` 会被调用。
- 读取不可访问属性的值时，`__get()` 会被调用。
- 当对不可访问属性调用 `isset()` 或 `empty()` 时，`__isset()` 会被调用
- 当对不可访问属性调用 `unset()` 时，`__unset()` 会被调用
- `__sleep()` 在`serialize()` 函数执行之前调用
- `__wakeup()` 在`unserialize()` 函数执行之前调用
- `__toString` 在一个类被当成字符串时被调用（不仅仅是echo的时候,比如file_exists()判断也会触发

## 例题

### 源码

```
//index.php
<?php 
    require_once('shield.php');
    $x = new Shield();
    isset($_GET['class']) && $g = $_GET['class'];
    if (!empty($g)) {
        $x = unserialize($g);
    }
    echo $x->readfile();
?>
//shield.php
<?php
    //flag is in flag.php
    class Shield {
        public $file;
        function __construct($filename = '') {
            $this -> file = $filename;
        }
        function readfile() {
            if (!empty($this->file) && stripos($this->file,'..')===FALSE  
            && stripos($this->file,'/')===FALSE && stripos($this->file,'\\')==FALSE) {
                return @file_get_contents($this->file);
            }
        }
    }
?>
```

### 分析

传入一个参数`class`,class赋值给$g,然后对g进行反序列化

利用的对象是Shield,两个方法,`__construct`在对象被创建时调用,这里并没有创建对象,这里不需要创建对象,所以反序列化不会触发该方法

```
class Shield {
        public $file="flag.php";
}
$a=new Shield();
echo serialize($a);
//将输出值传过去即可读取flag文件得到flag
```