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

# 变量

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

**默认值**：在 dart 一切变量在未被赋值的时候都是 null，即使是 int 型的初始值也是 null，因为 dart 一切皆对象。

## final 和 const

两者都表示常量，但是const 是一个编译期常量，final 在他第一次被调用的时候初始化。

const 要是定义在类的内部，则需要加上 static 修饰符。

可以使用下面的方式定义一个变量:

```dart
  var foo = const [1,2,3,4,5,6,7,8];
  //  foo[1] = 10;   foo 不能被修改
  // 即使用 cosnt 初始化了一个变量，它一样可以被重新赋值
  foo = [1,2,3,4];
  foo = [4,5,6,7,5];
  foo[1] = 10; // 被重新赋值以后 就可以被修改了
```


# 基本类型

- numbers
- strings
- booleans
- lists
- maps
- runes
- symbols


## numbers

numbers 有两种类型

- int

  表达不大于64位的整数，-2^63 ~ 2^63 -1

- double
  
  表示 64 位浮点数

两种类型都可以使用 num 来定义，因为他们都是 num 的子类型。

```dart
num age = 0;
```

dart 还支持使用科学计数法来表示:

```dart
// 表示 2.14的10次方
var nums = 3.14e10;
```

dart 也给提供了一些转换的方法：

```dart
var one = int.parse("10");
// var one = int.parse("10.1"); 这样会出错的
var oneString = 1.toString();
// 保留两位小数，并转换成字符串  3.14
var pi = 3.1415926.toStringAsFixed(2);
...
```

## String

Dart 的 string 是 utf-16 字符集的。使用起来和 java 以及 kotlin 类似，特别是 kotlin。

**==** 在 dart 中比较的是两个字符串的内容是否相同。

```dart
// 这是不是和 kotlin 一模一样，支持使用 $ 来拼接字段，或者表达式
var testStr = "this is $testString ${testString.toUpperCase()}test";


// dart 可以支持换行时不写 +
var testString = "this"
                "is"
                "test string";
// 支持使用 三个单引号或者双引号 中间的字符串会原封不动的被打印出来，（包括换行和空格）
// 包括字符串中的 单 双引号等都不需要转义了。但是 \ 还是需要的
var str = """this is String \\ "" ''
            this is String""";

// 支持使用一个 r 表示真正的字符串 里面的任何字符都不需要转义，但是这个时候 单引号 会出问题？
var rawStr = r'"test /\n ${/''}"';

```

## boolean

和 java 以及 kotlin 并无区别

## Lists

和 kotlin 并无太大区别，下标也是从 0 开始。

dart 能够自动推断 list 的类型：

```dart
var intList = [1,2,3,4]; // 这种情况下自动推断城 list<int> 型

var testList = [1,2,3,"string"]; // 这种情况下 自动推断成 List 型 可以add 任
```

List 给了一系列的操作方法[文档](https://api.dartlang.org/stable/2.1.0/dart-core/List-class.html)