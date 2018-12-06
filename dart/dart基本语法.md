感觉和 java kotlin 有点类似，思想更像是 kotlin 一样，真真正正的一切皆对象，一切可以赋值的都是对象。

- dart 一切皆对象，所有对象都继承自 object
- dart 是强类型
- dart 能推断类型
- 支持通用类型
- 支持在函数内定义函数
- dart 没有 public protected private 关键字

写一个 dart 文件比如 dartTest.dart

定义一个 main 函数：

```dart
main() {
  for (var i = 0; i < 5; i++) {
    printNumber(i);
  }
}

printNumber(num aNumber){
  // 字符串的拼接有点像 kotlin
  print("this num is $aNumber");
}
```

然后直接执行或者命令行 `dart dartTest.dart` 就能打印下面：

    this num is 0
    this num is 1
    this num is 2
    this num is 3
    this num is 4    

## 变量

变量在声明的时候可以指定类型也可以不指定类型，比如上面的 for 循环也可以把 var 改成 int 或 num。

```dart
// 自动推断类型
var name = 'Bob';
// 不推断类型
dynamic name = 'Bob';
Object name = "bob";
// 指定了具体类型
String name = 'Bob';
```

- var 自动推断类型，在给它赋值的时候就已经明确了是什么类型
- String 指定类型，和 java 一样
- Object 表示任何类型，原因是 dart 中 object 也是最高基类
- dynamic 也是表示任何类型，但是和 object 不一样的是，它是在运行时才确定什么类型的，所以如下面的代码：

```dart
  dynamic dynamics = "test";
  dynamics++;
```

在编译时不会出错，但是运行时就出错了。

或者说 dynamic 是没有类型的， object 实际是有类型的它就是  object 类型，单 dynamic 是没有任何类型的或者说 dart 中没有任何对象能满足你想要的？慎用！
