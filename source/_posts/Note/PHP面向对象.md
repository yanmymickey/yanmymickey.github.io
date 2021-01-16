---
title: PHP面向对象
date: 2019-05-20
tags: [Note,php]
categories:
- [Note,PHP]
comments: false
---
# PHP面向对象

## 为什么要面向对象

面向对象的好处远远不只是便于维护代码，他所体现的面向对象的思想将一个体系分作很多的个体，这样来，别人只需要知道怎么调用你所写的类或对象。
<!-- more -->

## 类和对象：

**类**： 定义了一件事物的抽象特点。类的定义包含了数据的形式以及对数据的操作。
**对象**：是类的实例。
具体一点，有一个类是人类，这个人类就是抽象的，而张三这个人就是实例化的对象。
类是对象的**模板**，对象是类的**实例**
**成员变量** − 定义在类内部的变量。该变量的值对外是不可见的，但是可以通过成员函数访问，在类被实例化为对象后，该变量即可称为对象的**属性**。
类结构

```
class classname
{

}
```

**类的基本定义**：

```
class classname
{
    成员属性//前面有访问控制符 public protected private
    成员方法
}
```

**创建对象**

```
$对象名 = new 类名（）；
```

类名不区分大小写，采用驼峰式

**访问对象的属性**

```
$对象名->$属性名
```

对象传递机制说明
对象传递的不是数据本身，而是**对象标识符**
这里有两个有趣的例子

```
class cat
{
    public $name;
    public $age;
    public $color;
}

$a = new cat();
$a->name = 'Tom';
$a->age = '2';
$a->color = 'red';
$b = $a;
echo $b->color;
$b->name = "Sam";
echo $a->name;
```

在这里，你会发现输出的a的name居然随着b的修改而变化了
这是应为，这种传值是传对象标识符，相当于在创建`$a`的时候就创建了一个#1的对象标识符，对`$b`来说他只是复制了一份和`$a`一模一样的对象标识符指向数据存储的位置，所以会得到那种结果啦

下面这个有点不同，仔细分析一下结果吧

```
class cat
{
    public $name;
    public $age;
    public $color;
}

$a = new cat();
$a->name = 'Tom';
$a->age = '2';
$a->color = 'red';
$b = &$a;//传址
$b = 'blue';
echo $a->color;//会报错哦
echo $a;
```

**成员函数**
基本样式

```
class classname
{
    访问修饰符 function 函数名(参数){
        alalalal;
    }
}
```

**构造函数**
在class里面的构造函数
构造函数嘛，用于创建对象时给属性赋初始值和调用成员函数
构造函数的访问修饰符默认为public
没有返回值
一个类中只能有一个构造函数
样式：

```
class classname
{
    public $color;
    public $age;
    访问修饰符 function __construct($param_color,$param_age){
        $this->color = $param_color;
        $this->age = $param_age;//$this代表调用这个构造函数的对象
    }
}
$coco = new classname('red','12');//在创建这个新的对象Coco时系统自动完成了对象的属性定义
```

**析构函数**
析构函数(destructor) 与构造函数相反，当对象结束其生命周期时（例如对象所在的函数已调用完毕），系统自动执行析构函数
释放资源，关闭文件
默认情况下，先创建的对象后销毁（栈区的分配）

```
class classname
{
    function __destruct(){//No param

    }
}
```

在程序结束前也会销毁对象
当没有变量指向对象，这个对象就会被销毁
在销毁时就会调用析构函数
析构函数并不是销毁对象本身，而是销毁对象创造的相关资源，如数据库连接
PHP的内存回收机制

```
<?php
class test{
    function __destruct(){
        echo "当对象销毁时会调用！！！";
    }

}
$a = $b = $c = new test();

$a = null;
unset($b);

echo "<hr />";

?>
```

此例子，如上图，有三个变量引用`$a,$b,$c`指向test对象，test对象就有3个引用计数，当`$a` = null时，`$a`对test对象的引用丢失，计数-1，变为2，当`$b`被unset()时，`$b`对test对象的引用也丢失了，计数再-1，变为1，最后页面加载完毕，`$c`指向test对象的引用自动被释放，此时计数再-1，变为0，test对象已没有变量引用，就会被销毁，此时就会调用析构函数。
**访问修饰符初讲**
类中的方法可以被定义为公有（public），私有(private)或受保护(protected)。如果没有设置这些关键字，则该方法默认为公有。
其中，public定义的方法或属性在类的内部和外部都可以调用或读取
private和protected只能在类的内部调用或读取，他们的不同后面讲解。

**魔术方法**
__set()
**调用条件**：在类的**外部**对受保护的或者私有的属性赋值
可以对程序有更好的控制

```
class Dog
{
    public $name;
    public $age;
    public $sex;
    public function __construct($name,$sex){//这里没有问题，$sex只是个参数哦，什么名字都行
        $this->name = $name;//但是为了方便阅读，还是尽量规范参数的命名
        $this->age = $sex;
    }
    public function __set($name,$value){
        $this->$name = $value;
    }
}
//在类的外部
$coco = new Dog("titi",1);
echo $coco->name;
echo $coco->age;
$coco->age = 2;
$coco->name="coco";
echo $coco->name;
echo $coco->age;
```

不光可以对其赋值，还可以完成对值的一些操作
比如判断要修改的值是否符合修改要求等等，下面的几个魔术方法也一样
__get
**调用条件**：在类的**外部**得到受保护的或私有的属性的值

```
class Dog

{
    public $name;
    private $age;
    public function __construct($value1,$value2){
        $this->nmae = $value1;
        $this->age= $value2;
    }
    public function __get($name){
        return $this->$name;
    }
}
//在类的外部
$coco = new Dog("titi",1);
echo $coco->name;
echo $coco->age;
```

__isset()
**调用条件**：在**外部** isset()受保护的或者私密的属性

```
class Dog
{
    public $name;
    private $age;
    public function __construct($name,$age){
        $this->nmae = $name;
        $this->age = $age;
    }
    public function __isset($name){
        isset($this->$name);
    }
}
//在类的外部
$coco = new Dog("titi",1);
var_dump(isset($coco->age));//这个返回true 或者faulse
```

__unset
**调用条件**：在外部释放受保护的或者私密的属性

```
class Dog
{
    public $name;
    private $age;
    public function __construct($name,$sex){
        $this->nmae = $name;
        $this->sex = $sex;
    }
    public function __unset($name){
        unset($this->$name);
    }
}
//在类的外部
$coco = new Dog("titi",1);
unset($coco->$age);
var_dump(($coco);
```

## **继承**

**父类**：一个类被其他类继承，可将该类称为父类，或基类，或超类。
**子类**：一个类继承其他类称为子类，也可称为派生类。
**访问修饰符补充**：
public,protected修饰的方法和属性可被继承，而private不可
**值得注意的是，PHP中，如果在子类中创建了构造函数，则子类不会在构造函数中隐式调用父类中的构造函数，要执行父类的构造函数，需要在子类的构造函数中调用parent::__construct()。如果子类中没有定义构造函数则会如同普通类一样从父类继承（非private）**

```
class A//父类
{
    public $name;
    private $age;
    protected $job;
    function __construct($name,$age,$job)
    {
        $this->name = $name;
        $this->age = $age;
        $this->job = $job;
    }

    public function speak(){
        echo "hello".$this->name;
    }
}
class B extends A//子类
{
    public function eat(){
        echo "<br />eat".$this->job;//如果是age会如何呢？
    }
}

$a = new B("Tom",18,"student");
$a->speak();
$a->eat();
```

**方法重写**
如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
**抽象方法和抽象类**
含有抽象方法的一定是抽象类
抽象类不一定含有抽象方法
抽象类中可以有一般的函数
抽象类无法实例化，只能由一个继承他的子类实例化，继承他的子类将实现所有的抽象方法（重写）
抽象体

```
abstract class Father{
    public abstract function run();//抽象方法是没有方法体的
    public function eat(){//这个就是个普通的成员方法（函数）
        echo "eat";
    }
}
class son extends Father{
    public function run(){
        //父债子偿
    }

} 
```

抽象类其实就是介于普通类和接口之间的一个类，普通类需要实现所有方法，接口所有方法都不需要实现，而抽象类可以根据自己的需要去选择实现部分方法；但是一旦类里面有抽象方法，这个类就必须是抽象类，另外注意，抽象类跟接口一样，不能直接实例化为对象，只能被普通类继承，，其实抽象类同样体现了面向对象的多态现象
注意：
有什么可以实现类似多重继承的东西呢？接下来介绍接口
**接口：interface**
接口是通过 interface 关键字来定义的，就像定义一个标准的类一样，但其中定义所有的方法都是空的。

> 接口是一种对象用于与外界进行交互行为的约束，对于实现接口的类，要求一定要满足接口所指定的方法，也可以成为满足接口的“特征”。
> OOP中类有一种性质叫“封装”，就是要求类内部与外界环境要进行隔离，而类又需要实现一定的业务功能，因此就需要暴露出能够对外界产生影响的部分，这部分就被称之为接口。 ——–许致中大佬

**我的理解**：相当于一个菜单，列出了里面的功能，功能怎么实现外界不用管，只要用就可以

接口中定义的所有方法都必须是**公有**，这是接口的特性。
要实现一个接口，使用 implements 操作符。类中必须实现接口中定义的所有方法，否则会报一个致命错误。类可以实现多个接口，用逗号来分隔多个接口的名称

```
interface C
{
    function run();//因为公有是默认值，so略
}
interface E
{
    function walk();
}
class D implements C,E
{
    function  run(){
        echo "<br/>run";
    }
    function  walk(){
        echo "<br/>walk";
    }
}
```

有没有一种方法是既能实现类似多继承又能不带来麻烦呢？
**Trait**
很像接口，但是接口中的方法都是不会写的，只是放了一个定义，但是Trait就会在定义这个方法的时候写好具体的操作，例子如下

```
trait
trait能在定义一个标准类的时候定义它的方法
trait SayWorld {
    public function sayHello() {
        echo 'Hello World!';//trait中的
    }
}
class MyHelloWorld extends Base {
    use SayWorld;//
}
```

**优先级**
当前类中的方法或属性大于trait中的大于父类中的

```
<?php
class Base {
    public function sayHello() {
        echo 'Hello ';//父类中的
    }
}

trait SayWorld {
    public function sayHello() {
        echo 'World';//trait中的
    }
}

class MyHelloWorld extends Base {
    use SayWorld;
    public function sayHello(){//当前类的
        echo '!';
    }
}

$o = new MyHelloWorld();
$o->sayHello();
//输出结果是 !
?>
```

引用多个trait

```
<?php
trait Hello {
    public function sayHello() {
        echo 'Hello ';
    }
}

trait World {
    public function sayWorld() {
        echo 'World';
    }
}

class MyHelloWorld {
    use Hello, World;
    public function sayExclamationMark() {
        echo '!';
    }
}
```