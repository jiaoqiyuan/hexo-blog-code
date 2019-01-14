---
title: Codeabbey网站刷题-不定期更新
date: 2019-01-14 16:05:10
tags: [算法, codeabbey]
categories: 算法
photos: https://www.codeabbey.com/img/facade.gif
thumbnail: https://www.codeabbey.com/img/facade.gif
---

> 网站：[Codeabbey][1]

> 声明：这里仅仅是自娱自乐刷的一些题，很简单，基本是使用Python，后期准备转用java。

----

## [#1. Sum "A+B"][2]

```python
input_split = input().split()
print(int(input_split[0]) + int(input_split[1]))
```

## [#2. Sum in Loop][3]

```python
nums = int(input())   # 获取第一行输入
input_str = input()   # 获取第二行输入

print(sum(int(x) for x in input_str.split(' ')))
```

## [#3. Sums in Loop][4]

```python
nums = int(input()) #获取数据量

for i in range(nums):
    sum = 0
    input_split = [int(x) for x in input().split(' ')]
    for sub_str in input_split:
        sum += sub_str
    
    print(str(sum) + ' ')
```

## [#4. min-of-two][5]

```python
nums = int(input())

for i in range(nums):
    input_split = [int(x) for x in input().split(' ')]
    print(str(min(input_split)) + ' ')
```

## [#5. min-of-three][6]

```python
nums = int(input())

str_list = []

for i in range(nums):
    input_split = [int(x) for x in input().split(' ')]
    print(str(min(input_split)) + ' ')
```

## [#15. Maximum of array][7]

```python
input_str = input()

str_list = [int(x) for x in input_str.split(' ')]
print(str(max(str_list)) + ' ' + str(min(str_list)))
```

## [#6. Rounding][8]

```python
nums = int(input())

for i in range(nums):
    input_list = [int(x) for x in input().split(' ')]
    print(str(round(input_list[0]/input_list[1], 0))[:-2] + ' ')
```

## [#7. fahrenheit-celsius][9]

```python
input_str = input()
input_list = [int(x) for x in input_str.split(' ')][1:]   # 切分字符串，因为第一个字符是表示数字个数，所以去掉

for ele in input_list:
    print(str(round((ele - 32)/1.8, 0))[:-2] + ' ')
```



[1]: https://www.codeabbey.com/
[2]: https://www.codeabbey.com/index/task_view/sum-of-two
[3]: https://www.codeabbey.com/index/task_view/sum-in-loop
[4]: https://www.codeabbey.com/index/task_view/sums-in-loop
[5]: https://www.codeabbey.com/index/task_view/min-of-two
[6]: https://www.codeabbey.com/index/task_view/min-of-three
[7]: https://www.codeabbey.com/index/task_view/maximum-of-array
[8]: https://www.codeabbey.com/index/task_view/rounding
[9]: https://www.codeabbey.com/index/task_view/fahrenheit-celsius