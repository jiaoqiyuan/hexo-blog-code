---
title: 抽象的代价
date: 2019-04-13 13:47:25
tags:
categories: 翻译
photos: https://github.com/jiaoqiyuan/pics/raw/master/blog/the-cost-of-abstraction/cost-of-abstaction.png
---

信息来源：[阮一峰每周分享](http://www.ruanyifeng.com/blog/2019/02/weekly-issue-43.html)

文章来源：[The Cost of Abstraction](http://250bpm.com/blog:86)

-----

<!--more-->

The cost of duplicating code, of accumulating technical debt, of not having tests and so on is often discussed and written about. The cost of abstracting things is almost never mentioned though, despite it being a major factor in keeping any project maintainable.

人们经常讨论和书写关于代码复制、技术债务积累、缺乏测试等方面的成本，尽管抽象是保持任何项目可维护的主要因素，但抽象的成本则几乎从未被提及过。

As an example, imagine that your program often increases two variables in sequence:

下面是个例子，设想你的程序中经常按顺序对两个变量的值：

```C
i++;
j++;
```

So you decide that the functionality deserves a dedicated function:

因此你决定应该将这个功能作为专用函数：

```C
template<typename T> inc_pair(T &i, T &j) {
    i++;
    j++;
}
```

By doing so you've removed some duplicated code but also — and that's a thing that's rarely mentioned — you've added a new abstraction called "inc_pair".

这样做后，你移除了重复的代码，但同时-这是一个很少提到的东西-你也增加了叫做 "inc_pair" 的新抽象。

In this particular case, almost everybody will agree that adding the abstraction was not worth it. But why? It was a tradeoff between code duplication and increased level of abstraction. But why would one decide that the well known cost of code duplication is lower than somewhat fuzzy "cost of abstraction"?

在这种特殊情况下，大部分人应该都会同意增加这个抽象是得不偿失的。为什么呢？这是代码重复和增加抽象级别之间的权衡问题。但是为什么人们会认为众所周知的代码重复成本低于模糊的“抽象的成本”？

To answer that question we have to look at what "cost of abstraction" really means.

为了回答这个问题，我们就不得不研究一下到底“抽象的成本”的含义是什么。

One obvious cost is that an abstraction is adding to the cognitive load of whoever works with the code: They will have to keep one more fact in mind, namely, that inc_pair is a function that increases both of its arguments by one.

一个很明显的成本就是抽象增加了使用该代码的人的认知负荷：他们必须记住一个事实，即 inc_pair 是一个将两个参数都加1的函数。

However, the main cost of abstraction is in separating the implementation from the specification, or, to put it in a different way, the letter of the function from the spirit of the function. The former being what the function does, the latter being what everybody believes it should do.

但是，最主要的成本还是抽象将实现和规范分开，或者说是抽象以不同的方式将函数的含义抽象成函数名。前者是函数所做的，后者是每个人认为函数应该做的。

The important word in the above sentence is "everybody". Once you make an abstraction, it's not longer about what YOU believe it should do. You are entering the domain of social consensus. It's about what EVERYBODY involved thinks it does. And, as everybody knows, social consensus is hard.

上面这句话中的重要词是“每个人”，一旦你做了抽象，它不再单单是你认为应该做的事情啦。你正在进入一个叫做社会共识的领域，也就是所有人都认为它做了什么，而且众所周知，形成社会共识很难。

Let's look at a practical example: What if type T for our inc_pair function is "Duration"? What will the function do? Will it increase the duration by one second? One day? One nanosecond? Individual users may disagree.

让我们看一个实际的例子：如果我们的 inc_pair 函数的类型 T 是 "持续时间" 的话怎么办？这个函数会做什么？它会增加一秒钟的持续时间吗？还是一天？一纳秒？个人用户可能不会同意这些说法。

Another example: What if operator ++ on type T throws an exception while doing j++ ? Sould the function leave i and j in inconsistent state? Or should it try to keep the change atomic by doing i— ? And what if i— throws an exception while doing that? Maybe the function is supposed to copy the old values of the variables and set them back in case of an error? Nobody really knows.

另一个例子：如果 ++ 运算符在执行 j++ 操作时抛出了异常该怎么办？该函数是否会使i和j处于不一致状态？或者它是否应该尝试通过执行 i- 操作以保证变化的原子性？要是执行 i- 的时候又出现异常的话该怎么办？也许该函数应该复制变量的旧值并在出现错误时将其设置回来？没有人真的知道。

In short, the decision about creating an abstraction should not be taken lightly. There's a large social cost to every abstraction and if you are churning them out without even thinking about it you are on the way towards making the project unmaintainable. If the abstractions leak to the user you are also making it unusable.

简而言之，关于创建抽象的决定不应该掉以轻心。每一个抽象都会产生巨大的社会成本，如果你在没有考虑它的情况下就混入它们，那么你就是在努力使项目无法维护。如果抽象泄漏给用户，你也会使其无法使用。

That being said, most of the projects out there are already so abstraction heavy that they are almost unmaintainable. Thus, the programmers are accustomed to the state of unmaintainability, consider it the normal state of affairs and they happily contribute to the mess by adding more and more abstraction.

话虽如此，许多现存的大项目已经严重抽象以至于它们几乎无法维护。因此，程序员习惯于不可维护的状态，将其视为正常状态，并通过添加越来越多的抽象来愉快地为混乱做出贡献。

I've already shown one example of adding a new abstraction for no particularly good reason: Programmer notices that two pieces of code are somewhat similar and creates a function to de-duplicate it. The cost of doing so is rarely taken into account.

我已经展示了一个添加新抽象的例子，没有特别好的理由：程序员注意到两段代码有些相似，并创建了一个去除它的函数。这样做的成本很少被考虑。

Another example, a systemic one rather than an accidental one, is the use of mocking in tests. While tests are definitely useful, they often require certain piece of code to be abstracted so that it can be mocked — for example by creating an interface, then having both actual and mock implementation of the interface. Creation of an additional abstraction is just an collateral damage.

另一个例子是系统的而不是偶然的，是在测试中使用模拟。虽然测试肯定是有用的，但它们通常需要抽象某些代码以便可以模拟它 - 例如通过创建接口，然后同时具有接口的实际和模拟实现。创建额外的抽象只是一种附带损害。

Yet another example are inheritance hierarchies. Say we want 4 classes: Egg laying fish, live bearing fish, egg laying mammals (echidna) and live-bearing mammals. The intuitive reaction is to create an inheritance hierarchy topped by class Animal, with intermediary nodes Fish and Mammal. It is often done that way even if there's no use case for directly working with Animal, Fish or Mammal class. Thus, future maintainers are left to scratch their heads about how should those abstract classes behave.

又一个例子是继承层次结构。假设我们想要4个课程：产蛋鱼，活鱼，产卵哺乳动物（针鼹）和活体哺乳动物。直观的反应是创建一个以Animal类为首的继承层次结构，中间节点为Fish和Mammal。即使没有直接使用Animal、Fish或Mammal类的用例，也经常这样做。因此，未来的维护者可能会对这些抽象类的行为方式感到头疼。

In the end I would like to look at what anti-abstraction tools we have at our disposal.

最后，我想看一下我们可以使用的反抽象工具。

First of all, we have scoping. If a C function is declared as "static" it is visible only within that file. That limits the amount of damage it can cause. It is still an abstraction but it is prevented from leaking to the wider audience. Same can be said of Java's "private" modifier. Of course, it's not a panacea. If source file is 10,000 lines long the scope of a static function is greatly extended and it becomes almost as dangerous as a non-static function.

首先，我们有范围。如果C函数声明为“静态”，则仅在该文件中可见。这限制了它可能造成的伤害量。它仍然是一种抽象，但它不会泄露给更广泛的受众。Java的“私有”修饰符也是如此。当然，它不是灵丹妙药。如果源文件长度为10,000行，则静态函数的范围会大大扩展，并且几乎与非静态函数一样危险。

We also have unnamed objects (e.g. lambdas). It turns out it's hard to treat something with no name as an abstraction. (I wonder what Plato would say about that!) If the only way to refer to a function is by writing it down the letter and the spirit of the function become much more entangled.

我们还有未命名的对象（例如lambdas）。事实证明，很难将没有名字的东西视为抽象。（我想知道柏拉图对此会说些什么！）如果引用一个函数的唯一方法是将它写在字母上，那么函数的意义就会变得更加纠结。

It also seems that Go's implicit interfaces were designed to avoid unnecessary abstraction. Given that interfaces are not defined when object is implemented but rather they can be created on the fly as needed by the user, their scope can be significantly limited thus keeping the number of people affected by the abstraction lower.

似乎Go的隐式接口旨在避免不必要的抽象。假定在实现对象时未定义接口，而是可以根据用户的需要动态创建接口，则它们的范围可能会受到很大限制，从而使受抽象影响的人数保持较低。

My final question is whether we are doing enough to limit the amount of abstraction we have to deal with. And do we have sufficient tools to do such limiting? While in many cases it seems that it's only a question of programmer's attitude, at least in the case of mocking it's the tooling itself that contributes to the problem.

我的最后一个问题是我们是否做得足以限制我们必须处理的抽象量。我们是否有足够的工具来进行此类限制？虽然在许多情况下，它似乎只是程序员态度的问题，至少在嘲笑它的情况下，工具本身会导致问题。