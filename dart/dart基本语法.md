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

## maps

map 的使用和语法与 kotlin 并无太大区别。

```dart
  // 一样支持自动推断类型
  // 包括赋值的方法，比 kotlin 还要简单一些
  var map = {1:"test","aa":String};
  map["booleanKey"] = true;
  map.forEach((key,value){
    print("key::$key   value:$value");
  });
```

## Runes

UTF-32 字符串，因为 dart 的字符串是 UTF-16 型的，所以要表示 Unicode 字符串就需要使用 runes。

```dart
var clapping = '\u{1f44f}';
  print(clapping); //👏
  print(clapping.codeUnits);  //[55357, 56399]
  print(clapping.runes.toList()); //[128079]

  Runes input = new Runes(
      '\u2665  \u{1f605}  \u{1f60e}  \u{1f47b}  \u{1f596}  \u{1f44d}');
  print(new String.fromCharCodes(input));//♥  😅  😎  👻  🖖  👍
```

# 函数

在 dart 中函数也是一个对象，可以把函数作为一个参数传递给其他方法。也可以把 dart 类的实例当做一个函数来调用。

```dart
bool isString(Object obj){
  return obj is String;
}

isString2(Object obj){
  return obj is String;
}

isString3(Object obj) => obj is String;
```

- 上面三种写法效果一样
- 可以指定具体类型的返回，也可以不指定，dart 会自动推断
- 如果函数体内部只有一个表达式，则可以使用 => 来替代函数体，而且 => 后面只能有一个表达式

## 指定参数名

```dart
/**
 * 指定参数名，这样在调用的时候不用关注顺序，但是必须指定参数名。
  enableFlags(hidden:false,bold:true);
 * 
 * 使用指定参数名，可以在复杂函数下使代码更易读
 * 
 */
enableFlags({bool bold, bool hidden}){
  print("bold::$bold  hidden::$hidden");
}
```

## 可选参数以及默认值

```dart

/**
 * 在指定参数名时可以使用使用 required 来标识当前参数是可选的
 * 
 * @required 包含在 meta 库中，在 dart 需要引入`package:meta/meta.dart`
 * 在 fluter 中需要引入 `package:flutter/material.dart`
 * 
 * 同时也可以指定默认值，当指定默认值的时候，这个参数一样可以不传  testRequired();
 * 当参数名被指定的时候，任何参数都能指定默认值
 * 
 */
testRequired({String name = "laowang", @required int age}){
  print("name::$name   age::$age");
}

/**
 * 不使用指定参数名时，
 * 也可以使用 `[]` 来标记一个参数是可选的，但是这个时候就不能放到 `{}` 中了
 * 
 * 如果不指定参数名的时候，只有在 `[]` 即可选参数的时候给指定默认值，比如下面的 interests 参数
 */
testRequired2(String name, [int age, String interests = "book"]){
  print("name::$name   age::$age");
}


doStuff({
  List<int> list = const [1,2,3]
}){
  list[1] = 10;
  print("list:: $list");
}
```

## 匿名函数

匿名函数的作用和 java 一样，用在接口或者一些把函数作为参数的地方。比如下面的 List.forEach

```dart
  var testList = [1,2,3,4];

  // 匿名函数   参数放在括号里 {} 中放入函数体
  testList.forEach((item){
      //do Something
  });
```

也可以用在这样

```dart
 var madd = makeAdder(1);
 var madd2 = makeAdder(10);

  print(madd(1) == 2);
  print(madd2(8) == 18);


/**
 * 返回一个 函数 (num i) => addBy + i
 */
Function makeAdder(num addBy) {
  // return (num i) => addBy + i;
  return (num i){
      return addBy + i;
  };
}
```

## 闭包

上面的那一段代码也就体现了闭包。

```dart
  //  因为是闭包，以及完全的面向对象特性，所以有这样的神奇写法
  var callbacks = [];
  for (var i = 0; i < 2; i++) {
    callbacks.add(() => print(i));
  }
  callbacks.forEach((c) => c());
```

## 返回值

所有的函数都有默认值，即使下面的函数。未指定返回值的函数会自动返回 null。

```dart
funTest(){}
```

# 导入包

如果是本地的就直接 import 就可以了。

`import './dartTest.dart';`

导入网上的库：

在 `pubspec.yaml` 中添加依赖，然后导入即可。

如果没有 `pubspec.yaml` 则新建一个：

```yaml
# name 是必须的
name: dartTest

# 添加依赖
dependencies:
  meta: ^1.1.6
```

然后引入就可以了

`import 'package:meta/meta.dart';`

如果还没有安装上，就执行一次 `pub get` （flutter 中执行`flutter packages get`）