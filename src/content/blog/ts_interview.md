---
author: Kiera
pubDatetime: 2024-03-28T13:05:56.066Z
modDatetime: 2024-03-28
title: TypeScript面试题
slug: typeScript
featured: true
draft: false
tags:
  - typescript
description: TypeScript相关面试题整理
---

## 1. TypeScript 是什么？它与 JavaScript 有什么区别？

typescript是js的超集，提供了更严格的类型检查，更强大的面向对象编程能力支持和更好的工具支持，从而提高了js开发的安全性/可维护性以及开发效率

区别：

a. 静态类型检查

b. 类型推断和注解

c. 对象和接口

## 2. TypeScript 中的 any 类型和 unknown 类型有何异同？

any和unknown在ts中都是表示可接收任何类型的意思。

不同在于：

**any类型**：在编译时不接受任何检查，相当于直接使用js代码。

**unknown类型**：编译器会在没有使用类型断言（as）或类型宽化（if typeof value === string）的情况下，阻止进行任何操作。

## 3. 如何理解 TypeScript 中的泛型？举例说明其用法？

ts中的泛型是一种创建可重用组件的方式，同时还能保证组件类型的安全性.泛型提供一种可泛化函数/接口或类的方法，使得你可以在定义类型时不需要指定特定的类型，在使用或实例话这些泛化的对象时才提供具体的类型参数.

用法：

a. 数组：

```js
let arr = Array < number > [1, 2, 3];
```

b. 函数：

```js
function helloWorld<T> (args: T) {
  return args
}
let output = helloWorld<string>("myString");
```

c. 接口：

```js
interface introduce<T> {
  (arg: T): T;
}
function helloWorld<T> (args: T) {
  return args
}
let info:introduce<string> = helloWorld;
```

d. 类：

```js
class hello<T> {
  args: T;
  add: (x: T, y: T) => T;
}
let helloNumber = new hello<number>();
```

e. 泛型约束:

```js
interface introduce {
  name: string;
}
function introduceInfo<T extends introduce>(arg: T): T {
  return arg;
}
introduceInfo({ name: 'lmy' });
```

## 4. 何为装饰器？在 TypeScript 中如何使用？

装饰器是一种类型声明的方法，可附加在类/方法/属性或参数上。用@符号进行标注，其后是一个表达式。

在执行该表达式后，会返回一个函数，系统将在运行时调用这个函数，并将装饰的声明信息传入作为参数。

## 5. 什么是类型断言？如何在 TypeScript 中使用类型断言？

类型断言：可以将一个共用类型的变量指定为一个更具体的类型，让开发者告诉类型系统：“我知道自己在干什么”。

类型断言有两种语法，即“尖括号语法”和“as语法”。（在使用 JSX 时，只有 as 语法断言是被允许的）

```js
  let someValue: any = "this is a string";
  let strLength: number = (someValue as string).length;
```

## 6. TypeScript 中的接口是什么？

用于对类进行抽象，能把对象的结构和类型约束起来，并且可以描述一个对象应该具有哪些方法和属性.(interface)

## 7. TypeScript 中的命名空间和模块有何不同？

- **命名空间**: 是用来组织全局范围的代码，防止全局污染，使用关键词 namespace 定义。

- **模块**: 是 ES6 引入的用来组织和重用模块化代码的工具，使用 export 和 import 进行模块的导出和导入，每个文件在默认情况下就是一个模块。

## 8. TypeScript 支持哪些访问修饰符？

- **public**: 公有的，这是默认的访问修饰符，可以在任何地方访问此属性或方法。

- **private**: 私有的，这意味着属性或方法只能在其所属的类内部访问

- **protected**: 保护的，与 private 类似，但它也允许在派生类中访问该属性或方法。(访问范围：类内部 + 派生类内部)

- **readonly**: 只读的，只有在声明或在构造函数内初始化该属性

- **static**: 静态的，意味着这个属性或方法属于类本身，而不是类的实例。虽然这不是一个访问修饰符，但它修改了访问方式.

  (衍生出另一道面试题：静态方法和实例方法的区别---静态方法是直接定义在类（或构造函数）上的方法，只能通过类来调用，而不能在类的实例上调用；实例方法需要在类的实例上被调用，不能直接在类上调用。实例方法可以访问并修改实例的属性。)

## 9. 在 TypeScript 中如何实现类和接口的继承？extend

- **类的继承**：我们使用 extends 关键字来实现类的继承.

- **接口的继承**：我们同样使用 extends 关键字来实现接口的继承。一个接口可以继承一个或多个接口。

  可继承多个接口，如：interface A extends B, C {}

  接口继承多个接口时，如何处理同名成员？

  每个接口定义的同名成员必须具有相同的类型。如果类型不同，就会在编译时报错。

总结：所以在使用接口继承时，需要注意不同接口中同名成员的类型必须一致。

## 10. TypeScript 的基本语法有哪些？

- 变量声明: let name:string = 'aaa';
- 数组定义： let arr: number[] = [1, 2, 3];
- 元组类型: 元组类型允许表示一个已知元素数量和类型的数组;如：let x: [string, number];
- 枚举类型：enum Color {Red, Green, Blue};let c: Color = Color.Green;
- 任意值： any;
- 函数声明: function add(x: number, y: number): number {
  return x + y;}

## 11. 在 TypeScript 中函数如何定义？

- 函数声明:

```js
function greet(name: string): string {
  return "Hello, " + name;
}
```

- 函数表达式: 函数也可以赋值给一个变量

```js
let greet = function(name: string): string {
  return "Hello, " + name;
}
```

- 箭头函数:

```js
let greet = (name: string): string =>  {
  return "Hello, " + name;
}
```
