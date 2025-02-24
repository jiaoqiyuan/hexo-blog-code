---
title: 8 Reasons Python Sucks-译
date: 2019-01-11 07:35:51
tags:
categories: 翻译
thumbnail: https://getcodify.com/wp-content/uploads/2016/10/Python_logo.jpg
photos: https://getcodify.com/wp-content/uploads/2016/10/Python_logo.jpg
---


信息来源：[阮一峰每周分享][1]

文章来源：[8 Reasons Python Sucks][2]

------

## 我觉得Python辣鸡的8个理由


Occasionally I go out to lunch with some of my techie friends and we have a great time geeking together. We talk about projects, current events, and various tech-related issues. Inevitably, the discussion will turn to programming languages. One might lament "I have to modify some Java code. I hate Java. (Oh, sorry, Kyle.)" (It probably doesn't help that we gave Kyle the nickname "Java-boy" over a decade ago.) Another will gripe about some old monolithic shell code that nobody wants to rewrite.

<!--more-->

我偶尔出去跟我懂技术的朋友进行一些极客一点的聚会时，我们会讨论到一些项目、实时大事以及一些技术相关的问题。肯定的是，这些讨论最终都会落脚到编程语言上。其中有些人会哀嚎道“我有时不得不修改一些Java的底层代码，我讨厌Java，（额...，不还意思，开尔）”（十多年前，我们给凯尔起了个“Java-boy”的绰号，但这可能对避免我们吐槽Java无济于事）。另一个人可能会抱怨一些没人愿意重写的古老shell代码。

And me, well... I just blurted it out: I hate Python. I hate it with a passion. If I have the choice between using some pre-existing Python code or rewriting it in C, I'd rather rewrite it in C.

轮到我了，额... 我会脱口而出：我讨厌Python。我从个人感情上就讨厌Python。如果让我选择使用一些已经存在的Python代码或者用C语言重写它，我会毫不犹豫使用C语言重写这段Python代码。

When I finished shouting, Bill humorously added, "But what do you really think about Python, Neal?" So I'm dedicating this blog entry to Bill.

我吐槽完毕后，Bill幽默地补充道：“那么你觉得Python到底是什么？Neal”。所以我决定写下这篇博客给Bill。

Here's my list of "8 reasons Python sucks".

下面是我列举的“我不喜欢Python的8个原因”。

## Reason 1: Versions 版本问题

If you install a default Linux operating system, there's a really good chance that it will install multiple versions of Python. It will probably have Python2 and Python3, and maybe even some fractional versions like 3.5 or 3.7. There's a reason for this: Python3 is not fully compatible with Python2. Even some of the fractional versions are distinct enough to lack backwards compatibility.

如果你安装了一个Linux的默认系统，有很大的可能性这个Linux会带有多个版本的Python，会有Python2和Python3这两个版本，甚至可能会有一些像3.5或3.7的版本。这是因为：Python3与Python2不完全兼容。甚至一些小版本也不会向前兼容。

I'm all for adding new functionality to languages. I don't even mind if some old version becomes obsolete. However, Python installs in separate installations. My code for Python 3.5 won't work with the Python 3.7 installation unless I intentionally port it to 3.7. Enough Linux developers have decided that porting isn't worth the effort, so Ubuntu installs with both Python2 and Python3 -- because they are needed by different core functions.

我总是为变成语言添加它所支持的所有特性，我不在乎这个语言的一些老特性已经过时。但是Python的不同版本是安装在不同的地方的，是相互分离的，我使用Python3.5写的代码将不能在Python3.7的环境下执行，除非我将它移植到Python3.7环境下。相当多的Linux开发者认为Python程序的移植是不值得做的，所有Ubuntu同时安装了Python2和Python3 -- 因为不同的内核函数需要的Python版本不同。

This lack of backwards compatibility and split versions is usually a death knell. Commodore created one of the first home computers (long before the IBM PC or Apple). But the Commodore PET wasn't compatible with the subsequent Commodore CBM computer. And the CBM wasn't compatible with the VIC-20, Commodore-64, Amiga, etc. So either you spent a lot of time porting code from one platform to another, or you abandoned the platform. (Where's Commodore today? It died out as users abandoned the platform.)

缺乏向后兼容性、各个版本是分离的语言通常会死得很惨，Commodore创造了第一个家用计算机（比Apple和IBM PC
要早地多）。但是Commodore PET跟随后的Commodore CBM不兼容，而且CBM跟VIC-20、Commodore-64、Amiga等也是不兼容的。所以，你要么花所一大笔时间将程序从一个平台移植到另一个平台，要么抛弃一个平台。（看看Commodore今天还在吗？它在被人们抛弃后就消失了。）

Similarly, Perl used to be very popular. But when Perl3 came out, it wasn't fully backwards compatible with a lot of Perl2 code. The community griped, good code was ported, and the rest was abandoned. Then came Perl4 and the same thing happened. When Perl5 came out, a lot of people just switched to a different programming language that was more stable. Today, there's only a small community of people actively using Perl to maintain existing Perl projects. I haven't seen any major new projects based on Perl.

同样，Perl语言之前很流行，但是当Perl3问世时，Perl3不能完全兼容Perl2的很多很多代码。社区抱怨好的代码被移植到Perl3了，剩下的都被抛弃了。然后Perl4出现了，相同的事发生了。当Perl5出现的时候，相当大的一部分人已经从Perl转向了更加稳定的编程语言。今天，只有一小部分社区人员在维护现有的Perl项目，我再也没有见到过以Perl为主要语言的项目。

By the same means, Python has distinct silos of code for each version. And the community keeps dragging along the old versions. So you end up with a lot of old, dead Python code that keeps getting dragged along because nobody wants to spend the time porting it to the latest version. As far as I can tell, nobody creates new code for Python2, but we drag it along because nobody's ported the needed code to Python3.x. At the official [Python web site][3], their documentation is actively maintained and available for Python 2.7, 3.5, 3.6, and 3.7 -- because they can't decide to give up on the old code. Python is like the zombie of programming languages -- the dead just keep walking on.

相同地，Python的不同版本的代码之间就像独立的孤岛，而且社区还在拖着古老的代码前进。所以，你终将会有很多老旧的、无用的Python代码伴随终生，因为没人愿意花费来持续移植老旧代码到最新版本。据我所知，没有人再为Python2创建新的代码，但是我们会保留Python2的代码，因为没人将他们移植到Python3.x。在[Python官方网站][3]，他们的文档特别提到并提供了2.7,3.5,3.6和3.7版本 -- 因为他们没法作出放弃老本版代码的决心。Python就像个丧尸语言 -- 行尸走肉般地继续往前走。

## Reason 2: Installation 安装问题

With most software packages, you can easily run apt, yum, rpm, or some other install base and get the most recent code. That isn't the case with Python. If you install using 'apt-get install python', you don't know what version you're actually installing, and it may not be compatible with all of the code you need.

大部分的软件包，你可以很方便地通过apt、yum、rpm或者一些其他的安装手段进行安装，并且获得最新版本。这些方法都适合Python。如果你使用‘apt-get install python’，你不知道你安装的是那个版本的Python，而且安装后可能跟你代码需要的Python版本不兼容。

So instead, you install the version of Python you need. For one of the projects I was on, we used Python. But we had to use Python3.5 (the latest at that time). My computer ended up with Python2, Python2.6, Python3, and Python3.5 installed. Two were from the operating system, one was for the project, and one came in because of some unrelated software I installed for some other reason. Even though they are all "Python", they are not all the same.

所以，你需要安装你需要的Python版本。我参与过的一个项目中，我们使用Python语言开发，不过必须是使用Python3.5版本（当时最新的Python版本）。我的计算机最后安装了Python2, Python2.6, Python3和Python3.5，其中两个是操作系统自带的，有一个是项目开发需要的，还有一个是我安装的一些与项目不相关的软件需要的。尽管他们都是“Python”，他们并不相同。

If you want to install packages for Python, you're supposed to use "pip". (Pip stands for "Pip Installs Packages", because someone thinks recursive acronyms are still funny.) But since there's a bunch of versions of Python on the system, you have to remember to use the correct version of pip. Otherwise, 'pip' might run 'pip2' and not the 'pip3.7' that you need. (And you need to specify the actual path for pip3.7 if the name doesn't exist.)

如果你想为Python安装软件包，你可能会用到“pip”。（Pip代表“Pip Installs Package”，因为某人觉得使用递归式简写会比较有趣吧）但是因为操作系统上安装了多个版本的Python，你不得不牢记使用正确的pip版本，否则，'pip'可能表示的是'pip2'而不是你需要的'pip3.7'。（而且如果pip3.7命令不存在的情况下你需要指定pip3.7的运行路径）

I was advised by one teammate that I needed to [configure my environment][4] so that everything uses the Python 3.5 base. This worked great until I started on a second project that needed Python 3.6. Two concurrent projects with two different versions of Python -- no, that wasn't confusing. (What's the emoticon for sarcasm?)

我的一个同事建议我[配置一下环境变量][4]，这样所有的程序都使用的是Python3.5啦。这是个好建议，这在我开启另一个Python3.6版本的项目之前一切都很顺利。两个现存的项目需要不同的Python版本 -- 天啊，真让人头疼。（讽刺的表情符号是什么来着？）

The pip installer places files in the user's local directory. You don't use pip to install system-wide libraries. And Gawd forbid you make the mistake of running 'sudo pip', because that can screw up your entire computer! Running sudo might make some packages install at the system level, some install for the wrong version of Python, and some files in your home directory might end up being owned by root, so future non-sudo pip installs may fail due to permissions. Just don't do it.

pip安装程序将文件安装到用户的本地目录，你不会使用pip来安装系统方面的库，而且Gawd也禁止你犯这个错误：使用sudo pip运行安装程序，因为这可能会高砸你的整个电脑！使用sudo运行可能会把一些包安装到系统目录下，一些包可能安装到错误版本的Python包，另外一些安装到home目录下的文件可能最终所属用户可能是root，所以未来使用不带sudo的pip安装Python包时可能会因为权限的问题而失败。记住不要使用sudo来执行pip就行了。

By the way, who maintains these pip modules? The community. That is, no clear owner and no enforced chain of provenance or accountability. Earlier this year, a version of PyPI was found to have a [backdoor][5] that stole SSH credentials. This doesn't surprise me at all. (I don't use Node.js and npm for the same reason; I don't trust their [community repositories][6].)

顺便说一下，谁在维护这些pip模块呢？Python社区。是的，没有清晰的决策者，也咩有强制性的责任链和问责机制。今年早些时候，PyPI的一个版本中被发现存在一个偷取SSH证书的[后门][5]。我一点也不惊奇。（我不使用Node.js和npm也是相同的原因;我不信任他们的[社区库][6]）

## Reason 3: Syntax 语法问题

I'm a strong believer in readable code. And at first glance, Python seems very readable. That is, until you start making large code bases.

我是代码易读的忠诚信徒。第一眼看到Python时，Python的语法似乎非常易读，在我用它来构建大型代码库前，确实是这样的。

Most programming languages use some kind of notation to identify scope -- where a function begins and ends, actions contained in a conditional statement, range of a variable's definition, etc. With C, Java, JavaScript, Perl, and PHP, braces {...} define the scope. Lisp uses parenthesis (...). And Python? It uses spaces. If you need to define a scope for complex code, then you indent the next few lines. The scope ends when the indent ends.

大多数变成语言使用某种类型的符号来标识代码范围 -- 函数的开始位置和结束位置，执行动作包含在一个条件语句的里面，变量的定义范围等等。在C语言，Java，JavaScript，Perl和PHP使用{...}定义范围。Lisp使用插入语(...)。那么Python呢？它使用空格。如果你要为复杂代码定义一个作用范围，那么就将接下来需要包含在作用范围内的代码缩进吧。缩进结束了，作用范围也就结束了。

The Python manual says that you can use any number of spaces or tabs for defining the scope. However, ALWAYS USE FOUR SPACES PER INDENT! If you want to indent twice for nesting, use eight spaces! The Python community has standardized on this nomenclature, even though it isn't in the Python manual. Forget the fact that the examples in the documentation use tabs, tabs + 1 space, and other indents. The community is rabid about using four spaces. So unless you plan to never show your code to anyone else, always use four spaces for each indent.

Python手册里说你可以使用任意数量的空格或Tab键来定义范围。然而，我们最终都是使用四个空格来进行缩进。如果你想在缩进中嵌套缩进，那就用8个空格吧！Python社区有标准化的命名法，尽管这些命名法则不包含在Python手册中。忘记官方文档中使用tab、tab+1以及其他的一些缩进方式吧，社区正在疯狂地地使用四个空格进行。所以除非你永远不会将你的代码展示给逼人看，否则就老老实实使用四个空格进行代码缩进吧。

When I first saw Python code, I thought that using indents to define the scope seemed like a good idea. However, there's a huge downside. Deep nesting is permitted, but lines can get so wide that they wrap lines in the text editor. Long functions and long conditional actions may make it hard to match the start to the end. And I pity anyone who miscounts spaces and accidentally puts in three spaces instead of four somewhere -- this can take hours to debug and track down.

我第一眼看到Python代码时，我觉得使用缩进来定义范围似乎是个好主意。但是这有一个致命的缺点，深层嵌套是允许的，这就导致一行代码可能会占据很多字符，导致代码在编辑器中查看时看起来会占据好几行。长一点的函数和长一点的条件语句可能很难匹配好开始和结束的缩进。我非常可怜那些误将四个空格写成三个空格的人 -- 这个错误可能要话费数个小时来跟踪和调试。

For other languages, I've picked up the habit of putting debug code without any indents. This way, I can quickly browse the code and easily identify and remove debugging code when I'm done. But with Python? Anything not indented properly generates an indention error. This means debugging code must blend in to the active code.

就其他编程语言来说，我养成了忽略debug语句缩进的习惯，这样的话，我就可以快速地浏览代码并且在我不需要的时候很容易地移除调试代码。但是使用Python呢？任何不当的缩进都会导致缩进错误，这意味着调试代码也必须跟执行代码一样要进行缩进。

## Reason 4: Includes 包含错误

Most programming languages have some way to include other chunks of code. For C, it's "#include". For PHP, there's 'include', 'include_once', 'require', and 'require_once'. And for Python, there's "import".

大多数编程语言会有包含其他代码块的方法。就C语言来说，它有"#include"，PHP有'include'、‘include_once’和'require_once'。Python语言呢，有'import'。

Python's import permits including an entire module, part of a module, or a specific function from a module. Finding a list of what can be imported is non-intuitive. With C, you can just look in /usr/include/*.h. But with Python? It's best to use 'python -v' to list all of the places it looks, and then search every file in every directory and subdirectory from that list. I have friends who love Python and I've seen them grep through standard modules as they look for the thing they want to import. Seriously.

python的import允许包含整个模块、部分模块或者一个模块的一个特定的函数。查看哪些内容可以被导入进来是不直观的，使用C语言时，你可以看一眼/usr/include/*.h文件就可以了，但是使用Python时，最好使用python -v命令列举出所有可查找的位置，然后从显示的列表中在每个目录以及它的子目录中搜索每个文件。我的有些朋友喜爱Python，而且我看到他们为了寻找他们需要导入的内容，在标准库文件路径下使用grep进行搜索。我是认真的！

The import function also allows users to rename the imported code. They basically define a custom namespace. At first glance, this might seem nice, but it ends up impacting readability and long-term support. Renaming modules is great for small scripts, but really bad for long programs. People who use 1-2 letter namespaces, like "import numpy as n" should be shot (or forced to convert all of their code to Perl5).

import函数同时允许用户重命名被导入的代码，这让用户能从代码基础层面定义一个自己习惯的命名。初看之下，似乎很不错，但是这最终可能会影响代码的可读性以及长期可维护性。模块重命名对小的脚本文件来说可能很有用，但是对大型代码来说真的不适合。使用1到2个字母进行作为命名的人，比如‘import numpy as n’，应该被枪毙！（或者被强制要求将他们所有的代码转换成Perl5）

But that's not the worst part. With most languages, including code just includes the code. A few languages, like object-oriented C++, may execute code if there's a global object with a constructor. Similarly, some PHP code may define global variables, so an import could run code -- but that's typically considered a bad practice. In contrast, many Python modules include initialization functions that run during the import. You don't know what's running, you don't know what it does, and you might not notice. Unless there's a namespace conflict, in which case you get to spend many fun hours tracking down the cause.

这还不是糟糕的部分。大多数编程语言，包括那些只有代码的语言，一些像面向对象的编程语言比如c++，可能只有存在拥有构造函数的全局对象的情况下才会执行。同样地，一些PHP代码可能会定义全局变量，所以一个import就能运行代码 -- 但是这通常被认为是一个糟糕的做法。作为对比，许多Python模块包含的初始化函数在模块被import时会被执行。你不知道什么代码正在执行，你不知道它做了些什么，你甚至都注意不到这段代码的执行。除非有命名空间冲突，你才会花几个小时时间跟踪解决这个问题。

## Reason 5: Nomenclature 命名方法问题

In every other language, arrays are called 'arrays'. In Python, they are called 'lists'. And an associative array is sometimes called a 'hash' (Perl), but Python calls it a 'dictionary'. Python seems to go out of it's way to not use the common terms found throughout the computer and information science field.

在所有其他编程语言内，数组都叫’arrays‘，在python中，数组叫做'list'。而且一个关联数组有时会叫‘hash’（Perl），但是在Python中它叫做'dictionary'。Python似乎没有使用在计算机和信息科学领域发明的常用术语。

And then there are the names of libraries. PyPy, PyPi, NumPy, SciPy, SymPy, PyGtk, Pyglet, PyGame... (Yes, those first two are pronounced the same way, but they do very different things.) I understand that the 'py' is for Python. But couldn't they be consistent about whether it comes first or second?

然后还有一些库的名字。PyPy, PyPi, NumPy, SciPy, SymPy, PyGtk, Pyglet, PyGame...（是的，前两个的发音都一模一样，但是它们做的事情完全不同。）我理解Py代表的是Python，但是难道不应该考虑一下他们是出现在第一个或第二个位置的情况吗？

Some common libraries just gave up on the pun-like "Py" naming convention. This includes, matplotlib, nose, Pillow, and SQLAlchemy. And while some of the names may give you a hint to the purpose (e.g., "SQLAlchemy" contains SQL, so it's probably an SQL interface), others are just random words. If you didn't know what "BeautifulSoup" did, could you tell from the name that it's an HTML/XML parser? (As an aside, BeautifulSoup is well documented and easy to use. If every Python module was like this, I wouldn't be complaining so much. Unfortunately, this is the exception and not the norm. Most Python libraries seriously suck at documentation.)

一些通用库甚至直接放弃了Py这个双关语义的命名惯例，这些库包括，matplotlib, nose, Pillow以及SQLAlchemy。虽然有些库的名字会暗示你它的目的（比如，“SQLAlchemy”包含SQL，所以它很可能是SQL接口库），其他的就是一些随机字拼接的名字。如果你不知道‘BeautifulSoup’是做什么的，你能从名字判断出它是一个HTML/XML解析库吗？（配上旁白，BeautifulSoup的文档很棒而且很容易使用。如果所有的Python模块都像BeautifulSoup这样，我也不会抱怨这么多。不幸的是，BeautifulSoup是个例外而不是常规，大多数Python库的文档都相当糟糕）。

Overall, I view Python as a collection of libraries with horrible and inconsistent naming conventions. I have a [standing gripe][7] that open source projects typically have horrible names. Unless you know the project, you'll never figure out what it does by the name. And unless you know what to look for, you'll probably only find it by accidentally stumbling across someone who mentions it in passing. Most of Python's libraries reinforce this negative criticism.

总体来说，我认为Python是充满了可怕而且与命名管理不符的标准库的集合。我有一个[观点][7]就是开源项目的命名都很糟糕。除非你了解这个项目，否则你永远不会通过名字了解它做了什么。而且除非你知道要找什么，否则你很可能只有在那些已经被这个问题绊倒并且解决了问题的人那里了解到你想要的东西。大多数Python的库印证了这种负面评论。

## Reason 6: Quirks 怪癖问题

Every language has its quirks. With C, there's the weird nomenclature of using & and * for accessing address space and values. C also has that increment/decrement shortcut using ++ and --. With Bash, there's the whole "when to use a backslash" when quoting special characters like parenthesis and periods for regular expressions. And JavaScript has issues around compatibility (not every browser supports every useful function). However, Python has more quirks than any other language I've ever seen. Consider strings:

每个语言都有自己的怪癖，比如C语言，访问地址空间和值时的使用&和*的怪异命名法。C语言也存在使用++和--来表示自增/自减的用法。对Bash来说，在插入特殊字符和使用正则表达式的时候，存在“何时使用反斜杠”的用法。而且JavaScript在兼容性方面存在问题（不是所有的浏览器都支持每一个有用的函数）。但是Python的怪癖用法比其他语言多得多，就拿字符串来说：

- In C, double quotes enclose strings. Single quotes enclose characters.

- 在C语言中，使用双引号包裹字符串，单引号包裹字符。

- In PHP and Bash, both types of quotes can enclose strings. However, a double quote can have variables embedded in the string. In contrast, single quoted strings are literals; any embedded variable-like names are not expanded.

- 在PHP和Bash中，不论单引号还是双引号都可以包裹字符串，但是双引号可以在字符串中嵌入变量。相反，单引号字符串是文字;任何类似嵌入变量的名称都不会被展开。

- In JavaScript, there's really no difference between single quotes and double quotes.

- 在JavaScript中，单引号和双引号没有任何区别。

- In Python, there's no difference between single quotes and double quotes. However, if you want your string to span lines, then you need to use triple quotes """string""" or '''string'''. And if you want to use binary, then you need to preference the string with b (b'binary') or r (r'raw'). And sometimes you need to cast your strings as strings using str(string), or convert it to utf8 using string.encode('utf-8').

- 在Python中，单引号和双引号没有区别。但是，如果你想要将字符串分行显示，你就要使用三个双引号或单引号比如"""string"""或者‘’‘string’‘’。如果你想要使用二进制，你需要添加为字符串前面加个b(b表示‘binary’)或者r(r表示‘raw’)。而且，有些情况下你需要使用str(string)将字符串转换成字符串，或者使用string.encode('utf-8')将它转换成utf8格式。

If you thought that =, ==, and === was initially a little weird in PHP and JavaScript, wait until you play with quotes in Python.

如果你原来认为PHP和JavaScript中的=, ==和===很奇怪，使用了python中的引号你就知道什么叫怪异了。


## Reason 7: Pass By Object Reference 传递对象引用

Most programming languages pass function parameters by value. If the function alters the value, the results are not passed back to the calling code. But as I've already explained, Python goes out of its way to be different. Python defaults to doing functions with pass-by-object-reference parameters. This means that changing the source variable may end up changing the value.

多数编程语言都是通过值来传递函数参数，如果函数改变了这个值，改变后的值不会返回给调用这个函数的代码。但是就像我之前说的，Python总是特立独行。Python默认通过传递对象引用的方式来传递函数参数，这意味着修改了源变量的值后最终会改变所有调用这个变量的代码得到的值。

This is one of the big differences between procedural, functional, and object-oriented programming languages. If every variable is passed by object reference, and any change to the variable changes the reference everywhere, then you might as well use globals for everything. Calling the same object by different names doesn't change the object, so it is effectively global. And as C programmers learned long ago, g[lobal variables are evil][8] and [should not be used][9].

上面说的函数传递方式是过程式，函数式和面向对象式编程语言之间的一个很大的不同。如果没一个变量都是通过引用传递，那么对这个变量的任何改变都会影响所有引用这个变量的地方，那么你还不如全部使用全局变量呢。调用相同对象的不同名字不改变这个对象，实际上它是全局的。就像C程序员很早之前就学过的，[全局变量就是魔鬼][8]，[千万别用][9]。

In Python, you have to work to pass variables by value. Saying "a=b" just assigns another name to the same object space; this doesn't copy the value of b into a. If you actually meant to copy the value, then you need to use a copy function. Usually this is "a=b.copy()". However, notice that I said "usually". Not all data types have a 'copy' prototype. Or maybe the copy function is incomplete. In those cases, there is a separate library called 'copy' that you can use: "a=copy.deepcopy(b)".

在Python中，你必须使用使用值来传递变量，‘a=b’仅仅是将另一个名字分配到同样的对象内存空间，这不是将b的值拷贝给变量a，如果你确实想要拷贝值，那么你就需要使用拷贝函数。通常是这样'a=b.copy()'。但是注意，我说的是通常，不是所有的数据都有一个叫做copy的原型，也或许这个数据的拷贝函数是残缺的。这种情况下，有一个叫做copy的库可以供你使用：'a=copy.deepcopy(b)'

## Reason 8: Local Names 本地名称

It's a common programming technique to name the program after the library or function being used. For example, if I'm testing a screen capture program with a C library called "libscreencapture.so", I would call my program "screencapture.c" and compile into "screencapture.exe".

有个通用的编程技巧就是将使用了相关函数或者库的程序命名为这个库或函数的名字，比如，如果我使用C语言写了一个测试屏幕特性的程序叫做“libscreencapture.so”，那么我就会把我的程序命名为“screencapture.c”，然后将它编译为“screencapture.exe”。

```
gcc -o screencapture.exe screencapture.c -lscreencapture
```

With C, Java, JavaScript, Perl, PHP, etc., this works fine because the language can easily distinguish resource libraries from the local program; they have different paths. But with Python? Don't do this. Never do this. Why? Python assumes you want to import the local code first. If I have a program called "screencapture.py" that uses "import screencapture", then it will import itself rather than the system library. At minimum, you should call your local program "myscreencapture.py" instead.

使用C语言，Java，JavaScript，Perl，PHP等等语言时，上面说的命名技巧很适用，因为在本地程序中很容易区分哪些是源码哪些是库文件，他们存在于不同的路径下。但是使用Python呢，别这么做！永远你不要这么做。为什么？Python会假定你首先要导入本地代码，如果你有一个叫做screencapture.py的程序，它使用了"import screencapture"，那么它就会导入自身而不是系统库。至少，你应该把你的成为改名为myscreencapture.py。

## Not All Bad 并非一无是处

Python is a very popular language and has a huge following. I even have a handful of friends who really like Python -- it's their preferred programming language. Over the years, I've discussed these issues with them, and each time they nod their heads and agree. They don't disagree that these are problems with Python; they just think it's not bad enough for them to stop loving the language.

Python是一门非常流行的编程语言，而且有大量的追随者。我甚至也有一大票喜爱Python的朋友 -- Python是他们最爱的编程语言。这些年来，我跟他们也讨论过这些问题，每次他们也都点头表示同意。他们并非不承认Python存在问题，他们只是觉得这些问题不足以影响他们热爱这门语言。

My friends often cite all of the really cool Python libraries that exist. And I agree that some of the libraries are really useful. For example, BeautifulSoup is one of the best HTML parsers I've ever used, NumPy makes multidimensional arrays and complex mathematics easier to implement, and TensorFlow is very useful for machine learning. However, I'm not going to make a monolithic program in Python just because I like TensorFlow or SciPy. I'm not going to give up readability and maintainability for a [free pony][10]; it's not worth the effort.

我的朋友经常引用那些现有的非常酷的Python库，而且我承认有些库确实很有用。比如，BeautifulSoup库就是我所用过的最好的HTML解析库之一，NumPy使得多维数组和复杂数学计算变得很轻易就能实现，而且TensorFlow对于机器学习来说也很有用。但是，我不会因为喜爱TensorFlow或者SciPy就在一个程序中仅仅就使用Python来编程。我不会放弃[free pony][10]的可读性和可维护性;这不值得我的努力。

Usually when I write negative criticisms about a topic, I also try to write something positive. I followed my blog entry on "Open Source Sucks" with "Open Source Rocks". And when I wrote about limitations with FFmpeg, I explicitly mentioned how it's the best video processing library out there. But I can't make a list of good things about Python because I really think that Python sucks.

通常情况下，当我针对一个主题写下负面的批评内容时，我也会试着写一些积极的内容。我会持续更新我的博客“Open Source Sucks”和"Open Source Rocks"。当我写了FFMepeg的局限性时，我也明确有提到它是如何将视频处理库做到最好的。但是我无法很好地列举出Python的优点，因为我真的觉得Python辣鸡！

(Hey Bill, does this answer your question?)

(嘿，Bill，对我的回答还满意吗？)

[1]: http://www.ruanyifeng.com/blog/2019/01/weekly-issue-39.html
[2]: https://www.hackerfactor.com/blog/index.php?/archives/825-8-Reasons-Python-Sucks.html
[3]: https://docs.python.org/3/
[4]: https://docs.python-guide.org/dev/virtualenvs/
[5]: https://www.bleepingcomputer.com/news/security/backdoored-python-library-caught-stealing-ssh-credentials/
[6]: https://www.infoq.com/news/2018/05/npm-getcookies-backdoor
[7]: https://www.hackerfactor.com/blog/index.php?/archives/415-Open-Source-Sucks.html
[8]: https://www.learncpp.com/cpp-tutorial/4-2a-why-global-variables-are-evil/
[9]: https://www.quora.com/Is-it-a-bad-practice-to-use-global-variables-in-any-programming-language
[10]: https://www.urbandictionary.com/define.php?term=Free%20Pony