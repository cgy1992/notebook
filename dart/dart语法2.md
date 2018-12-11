# 操作符

一般操作符和 java 以及 kotlin 没有区别，这里关注几个不一样的。

## ... 级联操作符

```dart
  // .. 是级联操作符，有点像 react 中的级联操作符
  // 它和 java 的链式编程是有区别的，它对当前这个对象起作用。比如下面这样
  var test = Test() // 初始化一个对象 test
              ..name = "test" // 设置name 为 test
              ..age = 10  //设置 age 为 10
              ..hello() // 调用了一下 hello 函数
              ..getName() // 调用了一下 getName 函数 
              ..name = "maintel"; // 设置了 name 为 maintel


class Test{
  var name;
  var age = 0;

  void hello(){
    print("name::$name  age::$age");
  }

  String getName(){
    return name;
  }
}
```