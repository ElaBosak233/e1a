---
title: PHP 魔术方法笔记
date: 2023-11-15
categories:
  - CTF
  - Web
---

## `__construct()`

这是一个构造方法，在创建一个对象时自动调用，它可以用来初始化对象的属性和执行其他必要的操作

```php
class MyClass {
    public function __construct() {
        echo "对象已创建！";
    }
}

$obj = new MyClass(); // 输出：对象已创建！
```

## `__destruct()`

这是一个析构方法，在对象被销毁时自动调用，它可以用来进行清理操作，例如关闭数据库连接或释放资源

```php
class MyClass {
    public function __destruct() {
        echo "对象已销毁！";
    }
}

$obj = new MyClass();
unset($obj); // 输出：对象已销毁！
```

## `__get($property)`

当访问一个类的私有属性时自动调用，它接收要访问的属性名作为参数，并返回属性的值

```php
class MyClass {
    private $name = "Ella";

    public function __get($property) {
        if (property_exists($this, $property)) {
            return $this->$property;
        } else {
            return "属性不存在！";
        }
    }
}

$obj = new MyClass();
echo $obj->name; // 输出：Ella
echo $obj->age; // 输出：属性不存在！
```

## `__set($property, $value)`

当给一个类的私有属性赋值时自动调用，它接收要设置的属性名和属性值作为参数，并将属性的值设置为指定的值

```php
class MyClass {
    private $name;

    public function __set($property, $value) {
        if ($property === "name") {
            $this->name = $value;
        } else {
            echo "属性不存在！";
        }
    }
}

$obj = new MyClass();
$obj->name = "Ella";
echo $obj->name; // 输出：Ella
$obj->age = 25; // 输出：属性不存在！
```

## `__isset($property)`

当使用 `isset()` 函数或 `empty()` 函数检查一个类的私有属性是否存在或为空时自动调用，它接收要检查的属性名作为参数，并返回一个布尔值

```php
class MyClass {
    private $name = "Ella";

    public function __isset($property) {
        return property_exists($this, $property);
    }
}

$obj = new MyClass();
var_dump(isset($obj->name)); // 输出：bool(true)
var_dump(isset($obj->age)); // 输出：bool(false)
```

## `__unset($property)`

当使用 `unset()` 函数删除一个类的私有属性时自动调用，它接收要删除的属性名作为参数

```php
class MyClass {
    private $name = "Ella";

    public function __unset($property) {
        echo "属性被删除！";
    }
}

$obj = new MyClass();
unset($obj->name); // 输出：属性被删除！
```

## `__call($method, $args)`

当调用一个类中不存在或不可访问的方法时自动调用，它接收要调用的方法名和传递给方法的参数数组作为参数

```php
class MyClass {
    public function __call($method, $args) {
        echo "调用的方法：$method，参数：" . implode(", ", $args);
    }
}

$obj = new MyClass();
$obj->doSomething("参数1", "参数2"); // 输出：调用的方法：doSomething，参数：参数1, 参数2
```

## `__callStatic($method, $args)`

当调用一个不存在或不可访问的静态方法时自动调用，它接收要调用的方法名和传递给方法的参数数组作为参数

```php
class MyClass {
    public static function __callStatic($method, $args) {
        echo "调用的静态方法：$method，参数：" . implode(", ", $args);
    }
}

MyClass::doSomething("参数1", "参数2"); // 输出：调用的静态方法：doSomething，参数：参数1, 参数2
```

## `__toString()`

当将一个对象转换为字符串时自动调用，它没有任何参数，并返回一个表示对象的字符串

```php
class MyClass {
    public function __toString() {
        return "这是一个对象的字符串表示！";
    }
}

$obj = new MyClass();
echo $obj; // 输出：这是一个对象的字符串表示！
```

## `__invoke($args)`

当尝试将一个对象作为函数调用时自动调用，它接收传递给函数的参数数组，并返回一个结果

```php
class MyClass {
    public function __invoke($args) {
        echo "调用了对象作为函数，参数：" . implode(", ", $args);
    }
}

$obj = new MyClass();
$obj(["参数1", "参数2"]); // 输出：调用了对象作为函数，参数：参数1, 参数2
```

## `__clone()`

当使用 `clone` 关键字复制一个对象时自动调用，它可以用来执行一些特殊的克隆操作

```php
class MyClass {
    public function __clone() {
        echo "对象已被克隆！";
    }
}

$obj1 = new MyClass();
$obj2 = clone $obj1; // 输出：对象已被克隆！
```