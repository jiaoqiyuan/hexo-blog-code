---
title: 为什么面向对象编程糟透了
date: 2019-05-19 11:11:23
tags:
categories: 翻译
photos: https://github.com/jiaoqiyuan/pics/raw/master/blog/why-oop-sucks/why-oo-sucks.png
---

信息来源：[阮一峰每周分享](http://www.ruanyifeng.com/blog/2019/05/weekly-issue-56.html)

文章来源：[Why OO Sucks](http://www.cs.otago.ac.nz/staffpriv/ok/Joe-Hates-OO.htm)

-----

When I was first introduced to the idea of OOP I was sceptical but didn't know why—it just felt “wrong”. After its introduction OOP became very popular (I will explain why later) and criticising OOP was rather like “swearing in church”. OOness became something that every respectable language just had to have.

我第一次接触面向对象编程时不知道为什么是持怀疑态度的，我就是感觉“不对劲”。在面向对象编程变得越来越流行（我稍后会介绍为什么）后，批评面向对象就好像“在教堂诅咒别人”。面向对象变成每种可敬的编程语言所必须要拥有的东西。

<!--more-->

As Erlang became popular we were often asked “Is Erlang OO”—well, of course the true answer was “No of course not”—but we didn't care to say this out loud—so we invented a serious of ingenious ways of answering the question that were designed to give the impression that Erlang was (sort of) OO (If you waved your hands a lot) but not really (if you listened to what we actually said, and read the small print carefully).

曾今 Erlang 语言流行时我们问过“Erlang 是面向对象的吗” - 这个嘛，当然真实的答案是“不是，当然不是的” - 但我们并不在乎大声说出来 - 所以我们发明了一种巧妙的方式来回答这个问题，这些方法旨在给人留下这样的印象：Erlang 是（某种程度上）OO（如果你经常挥手），但不是真正的OO（如果你听了我们实际说的话，仔细阅读小字）。

At this point I am reminded of the keynote speech of the then boss of IBM in France who addressed the audience at the 7th IEEE Logic programming conference in Paris. IBM Prolog had added a lot of OO extensions. When asked why he replied:

在这一点上，我想起了当时 IBM 在法国的老板的主题演讲，他在巴黎举行的第七届IEEE Logic 编程大会上向观众致辞。IBM 在序言中增加了许多 OO 扩展。当被问到为什么这样做时他回答说：

*Our customers wanted OO Prolog so we made OO Prolog* 

*我们的客户想要 OO Perlog，所以我们推出了 OO Perlog*

I remember thinking “How simple, no qualms of conscience, no soul-searching, no asking ‘Is this the right thing to do’ …”

我记得我在想“太直白，太没良心，没有反省，没有提出‘做的是正确的事情吗’ ...”

## Why OO sucks 为什么面向对象编程糟透了

My principal objection to OOP goes back to the basic ideas involved, I will outline some of these ideas and my objections to them.

我对 OOP 持反对意见的原因可以归结到 OOP 的基本思想，我会列出一些 OOP 的基本思想以及我对它的反对意见。

### Objection 1.  Data structure and functions should not be bound together 

### 异议1. 数据结构和方法不应该绑定在一起

Objects bind functions and data structures together in indivisible units. I think this is a fundamental error since functions and data structures belong in totally different worlds. Why is this?

对象将方法和数据结构绑定在一个独立的单元中。我认为这是一个根本性的错误，因为函数和数据结构属于完全不同的世界。为什么是这样呢？

- Functions do things. They have inputs and outputs. The inputs and outputs are data structures, which get changed by the functions. In most languages functions are built from sequences of imperatives: “Do this and then that …”. To understand functions you have to understand the order in which things get done. (In lazy FPLs and logical languages this restriction is relaxed.)

- 方法用于执行具体操作。它们有输入和输出，输入和输出是属于数据结构，它们由方法来改变。在大多数编程语言中方法是根据命令序列组建的：“先做这个然后做那个...”。想要理解方法，你必须理解做事的先后顺序。（在惰性 FPL 和逻辑语言中，这种限制可以适当放松。）

- Data structures just are. They don't do anything. They are intrinsically declarative. “Understanding” a data structure is a lot easier than “understanding” a function.

- 数据结构就是这样。他们不做任何事，它们本质上是声明性的。“理解”数据结构比“理解”函数容易得多。

Functions are understood as black boxes that transform inputs to outputs. If I understand the input and the output then I have understood the function. *This does not mean to say that I could have written the function.*

方法可以被理解为将输入转换为输出的黑盒子。如果我理解输入和输出，那么我已经理解了这个方法。*这并不意味着我可以写出这个方法。*

Functions are usually “understood” by observing that they are the things in a computational system whose job is to transfer data structures of type T1 into data structure of type T2.

通常通过观察它们是计算系统中的东西来“理解”函数，该计算系统的工作是将类型 T1 的数据结构传输到 T2 类型的数据结构中。

**Since functions and data structures are completely different types of animal it is fundamentally incorrect to lock them up in the same cage.**

**由于功能和数据结构是完全不同类型的动物，因此将它们锁定在同一个笼子中是根本不正确的。**

### Objection 2.  Everything has to be an object. 

### 异议2. 一切皆对象

Consider “time”. In an OO language a “time” has to be an object. (In Smalltalk, even “3” is an object.) But in a non OO language a “time” is a instance of a data type. For example, in Erlang there are lots of different varieties of time, which can be clearly and unambiguously specified using type declarations, as follows:

来看一下“时间”。在面向对象语言中，“时间”必须是对象。(即使很短的“3”这个字符也是一个对象。)但是在非面向对象语言中，“时间”仅仅是日期类型的一个实例。比如在 Erlang 中有各种各样类型的“时间”变量，可以使用类型声明准确无误地指定，如下所示：

```
-deftype day()     = 1..31.
-deftype month()   = 1..12.
-deftype year()    = int().
-deftype hour()    = 1..24.
-deftype minute()  = 1..60.
-deftype second()  = 1..60.
-deftype abstime() = {abstime,year(),month(),day(),hour(),min(),sec()}.
-deftype hms()     = {hms,hour(),min(),sec()}.
…
```

Note that these definitions do not belong to any particular object. they are ubiquitous and data structures representing times can be manipulated by any function in the system.

注意，这些定义不属于任何特定对象，它们无处不在，表示时间的数据结构可以被系统中的任何函数调用。

There are no associated methods.

没有相关的方法。

### Objection 3.  In an OOPL data type definitions are spread out all over the place.

### 异议3. 在面向对象程序设计语言的数据类型中，定义遍布整个区域

In an OOPL data type definitions belong to objects. So I can't find all the data type definition in one place. In Erlang or C I can define all my data types in a single include file or data dictionary. In an OOPL I can't—the data type definitions are spread out all over the place.

在面向对象程序设计语言中数据定义归属与对象，所以我无法在一个地方获取到所有数据类型的定义。在 Erlang 和 C 中我可以在一个头文件或数据字典中定义我的数据类型，但是在面向对象程序设计语言中没办法这样做 - 数据类型的定义遍布这个工作区域。

Let me give an example of this. Suppose I want to define a ubiquitous data structure. A ubiquitous data type is a data type that occurs “all over the place” in a system.

我来举个例子，假如我想定义一个无处不在的数据结构，也就是系统中任意地方都可以出现的。

As Lisp programmers have know for a long time it is better to have a smallish number of ubiquitous data types and a large number of small functions that work on them, than to have a large number of data types and a small number of functions that work on them.

正如Lisp程序员很早就已经知道的，最好是拥有少量无处不在的数据类型和大量适用于它们的小函数，而不是拥有大量的数据类型和少量的功能。


A ubiquitous data structure is something like a linked list, or an array or a hash table or a more advanced object like a time or date or filename.

无处不在的数据结构类似于链表，数组或散列表或更高级的对象，如时间或日期或文件名。

In an OOPL I have to choose some base object in which I will define the ubiquitous data structure. All other objects that want to use this data structure must inherit this object. Suppose now I want to create some “time” object, where does this belong and in which object…

在OOPL中，我必须选择一些基础对象，在其中我将定义无处不在的数据结构。要使用此数据结构的所有其他对象必须继承此对象。假设现在我想创建一些“时间”对象，它属于哪个对象...

### Objection 4.  Objects have private state.

### 异议4. 对象有私有状态

State is the root of all evil. In particular functions with side effects should be avoided.

状态是万恶之源。特别是应该避免具有副作用的方法。

While state in programming languages is undesirable, in the real world state abounds. I am highly interested in the state of my bank account, and when I deposit or withdraw money from my bank I expect the state of my bank account to be correctly updated.

虽然编程语言中的状态是不受欢迎的，但在现实世界中状态比比皆是。我对我的银行账户状态非常感兴趣，当我从银行存款或取款时，我希望我的银行账户状态能够正确更新。

Given that state exists in the real world what facilities should programming language provide for dealing with state?

鉴于现实世界中存在状态，编程语言应该为处理状态提供哪些便利？

- OOPLs say “hide the state from the programmer”. The state is hidden and visible only through access functions.

- OOPL说“隐藏程序员的状态”。只有通过访问功能才能隐藏和显示状态。

- Conventional programming languages (C, Pascal) say that the visibility of state variables is controlled by the scope rules of the language.

- 传统的编程语言（C，Pascal）表示状态变量的可见性由语言的范围规则控制。

- Pure declarative languages say that there is no state. The global state of the system is carried into all functions and comes out from all functions. Mechanisms like monads (for FPLs) and DCGs (logic languages) are used to hide state from the programmer so they can program “as if state didn't matter” but have full access to the state of the system should this be necessary.

- 纯粹的声明性语言说没有状态。系统的全局状态被传入所有方法并从所有方法中传出。像monad（用于FPL）和DCG（逻辑语言）这样的机制用于隐藏程序员的状态，因此他们可以编写“好像状态无关紧要”的程序，但如果有必要，可以完全访问系统状态。

The “hide the state from the programmer” option chosen by OOPLs is the worst possible choice. Instead of revealing the state and trying to find ways to minimise the nuisance of state, they hide it away.

OOPL选择的“隐藏程序员状态”选项是最糟糕的选择。他们没有暴露状态并试图找到最小化干扰状态的方法，而是将其隐藏起来。

## Why was OO popular? 为什么面向对象会流行？

- Reason 1. It was thought to be easy to learn.

- 原因1. 它被认为很容易学习。

- Reason 2. It was thought to make code reuse easier.

- 原因2. 它被认为使代码重用更容易。

- Reason 3. It was hyped.

- 原因3. 它被炒作了。

- Reason 4. It created a new software industry.

- 原因4. 它创造了一个新的软件产业。

I see no evidence of 1 and 2. Reasons 3 and 4 seem to be the driving force behind the technology. If a language technology is so bad that it creates a new industry to solve problems of its own making then it must be a good idea for the guys who want to make money.

我没有看到1和2的证据。原因3和4似乎是该技术背后的驱动力。如果语言技术如此糟糕以至于它创造了一个新的行业来解决自己制造的问题，那么对于那些想要赚钱的人来说一定是个好主意。

This is is the real driving force behind OOPs.

这是OOP背后真正的推动力。

## See The Third Manifesto 参见第三份宣言

[This section added in 2000 for COSC345 by R. A. O'Keefe.]

The book “The Third Manifesto” by Date and Darwen is about how to design object data bases “right”. In the appendices, they treat SQL3 rather ungently, but no more ungently than the muddled giant deserves. They are even more critical of ODMG; if the industry cannot, after spending large amounts of time and money, produce a logically coherent object model for object data base systems, perhaps OO is not as simple or as usable as it seems.

Date和Darwen的书“The Third Manifesto”是关于如何“正确”的设计对象数据库。在附录中，他们对SQL3的处理相当不明智，但并不比混乱的巨人应得的更加不切实际。他们对ODMG更加批评;如果行业在花费大量时间和金钱之后不能为对象数据库系统生成逻辑上一致的对象模型，那么OO可能不像看起来那么简单或可用。