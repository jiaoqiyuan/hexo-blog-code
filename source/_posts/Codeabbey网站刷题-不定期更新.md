---
title: Codeabbey网站刷题-不定期更新
date: 2019-01-21 09:05:10
tags: [算法, codeabbey]
categories: 算法
photos: https://www.codeabbey.com/img/facade.gif
thumbnail: https://www.codeabbey.com/img/facade.gif
---

> 网站：[Codeabbey][1]

> 声明：这里仅仅是自娱自乐刷的一些题，很简单，基本是使用Python，后期准备转用java。

> 补充：这里前面的题目都比较简单，还没有碰到需要特别高深算法的题目，偶尔做一下主要是为了熟悉一些Java中复杂数据类型的使用方法。

> 补充2019-01-18：题目前面序号是题目对应的编号，这里没有按编号顺序是因为每道题的难度分值不同，这里是按难度分值进行排序的。

----

<!--more-->

## [#1. Sum "A+B"][2]

```python
# python
input_split = input().split()
print(int(input_split[0]) + int(input_split[1]))
```

## [#2. Sum in Loop][3]

```python
# python
nums = int(input())   # 获取第一行输入
input_str = input()   # 获取第二行输入

print(sum(int(x) for x in input_str.split(' ')))
```

## [#3. Sums in Loop][4]

```python
# python
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
# python
nums = int(input())

for i in range(nums):
    input_split = [int(x) for x in input().split(' ')]
    print(str(min(input_split)) + ' ')
```

## [#5. min-of-three][6]

```python
# python
nums = int(input())

str_list = []

for i in range(nums):
    input_split = [int(x) for x in input().split(' ')]
    print(str(min(input_split)) + ' ')
```

## [#15. Maximum of array][7]

```python
# python
input_str = input()

str_list = [int(x) for x in input_str.split(' ')]
print(str(max(str_list)) + ' ' + str(min(str_list)))
```

## [#6. Rounding][8]

```python
# python
nums = int(input())

for i in range(nums):
    input_list = [int(x) for x in input().split(' ')]
    print(str(round(input_list[0]/input_list[1], 0))[:-2] + ' ')
```

## [#7. fahrenheit-celsius][9]

```python
# python
input_str = input()
input_list = [int(x) for x in input_str.split(' ')][1:]   # 切分字符串，因为第一个字符是表示数字个数，所以去掉

for ele in input_list:
    print(str(round((ele - 32)/1.8, 0))[:-2] + ' ')
```

## [#20. Vowel Count][10]

```python
# python
nums = int(input())

for i in range(nums):
    input_str = input()
    print(str(input_str.count('a') + input_str.count('e') + input_str.count('i') + input_str.count('o') + input_str.count('u') + input_str.count('y')) + ' ')
```

## [#11. sum-of-digits][11]

```python
# python
nums = int(input())

for i in range(nums):
    input_split = [int(x) for x in input().split(' ')]
    sum = input_split[0] * input_split[1] + input_split[2]
    result = 0
    for char_sum in str(sum):
        result += int(char_sum)
    print(str(result) + ' ')
```

## [#41. Median of Three][12]

```python
# python
nums = int(input())

for i in range(nums):
    input_split = [int(x) for x in input().split(' ')]
    for ele_ele in input_split:
        if (ele_ele > min(input_split)) and (ele_ele < max(input_split)):
            print(str(ele_ele) + ' ')
```

## [#8. Arithmetic Progression](https://www.codeabbey.com/index/task_view/arithmetic-progression)

```python
input_str = '''11 11 48
26 10 55
23 18 63
20 1 35
6 16 45
4 11 91
3 10 40
9 4 51'''

input_list = [x for x in input_str.split('\n')]

for ele in input_list:
    ele_sp = [int(x) for x in ele.split(' ')]
    result = 0
    i = 0
    while(i < ele_sp[2]):
        result += (ele_sp[0] + i * ele_sp[1])
        i += 1
        
    print(str(result) + ' ')
```

## [#28. Body Mass Index](https://www.codeabbey.com/index/task_view/body-mass-index)

```python
input_str = '''59 1.48
47 1.62
105 2.01
119 2.38
77 2.13
107 2.51
63 2.12
94 2.29
49 1.32
103 2.27
114 2.14
48 1.19
42 1.15
50 1.54
65 1.65
69 1.66
49 1.46
106 1.74
42 1.15
66 1.66
44 1.71
98 2.21
63 1.73
89 2.01
43 1.43
102 1.68
116 1.85
106 1.69
92 1.60
68 1.36
76 1.59
81 1.77
54 1.64
79 2.22
51 1.26'''

input_list = [x for x in input_str.split('\n')]

for ele in input_list:
    ele_sp = [x for x in ele.split(' ')]
    BMI = float(ele_sp[0]) / pow(float(ele_sp[1]), 2)
    if BMI < 18.5:
        print('under ')
    elif BMI >= 18.5 and BMI < 25.0:
        print('normal ')
    elif BMI >= 25.0 and BMI < 30.0:
        print('over ')
    else:
        print('obese ')
```

## [#9. Triangles](https://www.codeabbey.com/index/task_view/triangles)

```python
input_str = '''1148 467 638
616 1461 699
413 260 808
314 646 250
918 1166 2694
865 1055 713
2407 859 1365
2171 741 1446
1521 866 1402
1187 608 367
131 251 271
3324 1496 941
2845 961 1851
601 708 977
1306 749 1772
704 1643 655
907 705 2250
629 2277 1046
598 813 1631
1434 1444 976
662 1290 1491
1311 595 402
890 1690 1737
292 425 609
200 129 265
559 233 139
363 1386 608'''

input_list = [x for x in input_str.split('\n')]

for ele in input_list:
    ele_sp = [int(x) for x in ele.split(' ')]
    MAX = max(ele_sp)
    MIN = min(ele_sp)
    for ele_ele in ele_sp:
        if ele_ele > MIN and ele_ele < MAX:
            if ele_ele + MIN > MAX:
                print('1 ')
            else:
                print('0 ')
```

## [#13. Weighted sum of digits](https://www.codeabbey.com/index/task_view/weighted-sum-of-digits)

```python
input_str = '128 56 204984 325441473 71 10345 301662 253890 111 431670 2414456 40 6 21 339 149936 11972 15940 24552 186086 3076312 1068 483595 58251 0 3049439 2032872 24726155 967463 7883 16 66041 4062011 7108 114 73178230 2 21948283 36'

input_list = [x for x in input_str.split(' ')]

for ele in input_list:
    result = 0
    for i in range(len(ele)):
        result += int(ele[i]) * (i+1)
    print(str(result) + ' ')
```

## [#43. Dice Rolling](https://www.codeabbey.com/index/task_view/dice-rolling)

```python
input_str = '''0.577175040264
0.120321465656
0.481802093331
0.201835676562
0.793378914241
0.930038275197
0.727960446849
0.244160681497
0.0613653254695
0.823915954679
0.651526081376
0.528674236499
0.125569326803
0.34707117267
0.82844642736
0.49360822048
0.242098119576
0.337716317736
0.399298793636
0.293447557371
0.905004166067
0.884278282057
0.0873124250211
0.369706542697
0.611792732496
0.314362816978
0.881133318413
0.422235912643'''

input_list = [float(x) for x in input_str.split('\n')]

for ele in input_list:
    result = int(ele * 6 + 1)
    print(str(result) + ' ')
```

## [#16. Average of an array](https://www.codeabbey.com/index/task_view/average-of-array)

```python
input_str = '''1288 3485 1164 2104 335 3465 4031 629 1993 0
57 46 105 52 79 29 981 246 717 195 576 0
1523 1156 885 730 934 1796 2016 1578 1490 550 582 1657 234 0
9843 14352 4323 10754 15073 5988 11582 0
681 243 96 875 819 871 404 556 425 847 921 891 720 905 656 0
474 123 136 99 104 444 35 239 268 0
1879 8061 1838 7325 1808 2602 6133 161 1374 1170 4605 4766 7939 3775 3700 0
94 105 507 16 350 779 213 556 642 282 9 153 0
3096 2014 518 2229 3822 3120 169 0
335 99 267 272 1042 1192 1648 1746 1380 1856 0
1012 439 3672 3235 3004 700 3267 3613 0
1064 2780 2866 2179 594 330 2263 2585 2576 0
3109 3475 770 1396 2673 165 59 2288 1587 2882 3300 2026 0
933 3157 1609 449 833 119 1513 3613 2985 3691 110 0'''

input_list = [x for x in input_str.split('\n')]

for ele in input_list:
    ele_sp = [int(x) for x in ele.split(' ')]
    print(str(round(float(sum(ele_sp)/(len(ele_sp) - 1)) + 0.01)) + ' ')
```

## [#17. Array Checksum](https://www.codeabbey.com/index/task_view/array-checksum)

```python
input_str = '443975 811736101 25 87 401394351 857 505 18139 958615 2 522677 63689046 19908 216620803 465631 149281971 880164875 82420890 15 66833495 178851 838798168 4618324 2 1827 990 1012971'

input_list = [int(x) for x in input_str.split(' ')]

result = 0
for ele in input_list:
    result = ((result + ele) * 113) % 10000007
print(str(result))
```

## [#30. Reverse String](https://www.codeabbey.com/index/task_view/reverse-string)

```python
input_str = 'white jeopardy supper clown emperor till turn make and interrogative'

print(input_str[::-1])
```

## [#21. Array Counters](https://www.codeabbey.com/index/task_view/array-counters)

```python
inpt_str = '2 8 2 6 7 10 2 5 2 6 3 1 7 2 10 10 4 3 9 9 8 7 9 6 4 6 1 2 8 5 5 10 3 7 5 9 7 7 4 8 2 7 9 9 8 8 9 1 1 7 10 8 4 8 4 7 4 5 9 1 9 4 10 1 10 4 10 6 1 3 4 3 9 3 2 6 10 10 7 1 7 6 8 10 3 2 7 6 6 5 6 5 8 6 5 8 10 4 4 10 6 7 2 5 10 4 10 9 3 6 9 10 1 7 9 4 9 6 9 4 1 5 9 8 10 3 6 9 7 9 8 3 6 10 7 5 3 7 4 6 3 3 5 3 10 4 7 8 10 5 2 10 9 10 8 8 3 3 6 9 2 4 2 7 4 9 2 7 5 6 3 7 8 8 10 7 1 6 5 10 1 6 10 9 6 7 7 8 9 3 7 1 7 9 7 10 7 9 6 2 4 8 8 1 5 8 8 6 4 3 6 4 9 5 3 4 2 9 2 1 2 9 1 8 7 8 7 4 6 3 5 9 1 3 10 5 10 8 1 3 10 6 6 8 1 9 2 2 8 4 3 9 2 3 6 9 10 3 2 6 6 7 4 6 9 4 1 8 2 1 10 2 7 6 9 8 4 1 10 1 4 2 10 6 5 6 4 4 9 6 9 4 2 3 9 10 7 10 7 8 1 7 9 8 3'

input_list = [x for x in inpt_str.split(' ')]

for i in range(10):
    i += 1
    print(str(input_list.count(str(i))) + ' ')
```

## [#48. Collatz Swquence](https://www.codeabbey.com/index/task_view/collatz-sequence)

```python
input_str = '616 27 71 942 65 22 34513 4000 81 10 17 383 1732 6905 410 93 27729 25'

input_list = [int(x) for x in input_str.split(' ')]

step = 0
for ele in input_list:
    while(ele != 1):
        step += 1
        if ele % 2 == 0:
            ele = ele / 2
        else:
            ele = ele * 3 + 1
    print(str(step) + ' ')
    step = 0
```

## [#10. Linear Function](https://www.codeabbey.com/index/task_view/linear-function)

```python
def find_slope(a, b, c, d):
    return (d - b) / (c - a)

def find_intercept(a, b, m):
    return (b - m * a)

def linear_function():
    answer = []
    test_cases = int(input())
    for test_case in range(test_cases):
        a, b, c, d = [int(x) for x in input().split()]
        m = int(find_slope(a, b, c, d))
        g = int(find_intercept(a, b, m))
        answer.append('({0} {1})'.format(m, g))
    print(' '.join(answer))
    
linear_function()
```

## [#12. Modulo and time difference](https://www.codeabbey.com/index/task_view/modulo-and-time-difference)

```python
inputs = int(input())

for ele in range(inputs):
    day1, hour1, min1, sec1, day2, hour2, min2, sec2 = [int(x) for x in input().split(' ')]
    timestamp1 = sec1 + min1 * 60 + hour1 * 60 * 60 + day1 * 24 * 60 * 60
    timestamp2 = sec2 + min2 * 60 + hour2 * 60 * 60 + day2 * 24 * 60 * 60
    timestamp_desc = timestamp2 - timestamp1
    day = int( timestamp_desc/ (24 * 60 * 60))
    hour = int(timestamp_desc % (24 * 60 * 60) / (60 * 60))
    min = int(timestamp_desc % (24 * 60 * 60) % (60 * 60) / 60)
    sec = int(timestamp_desc % (24 * 60 * 60) % (60 * 60) % 60)
    print('(' + str(day) + ' ' + str(hour) + ' ' + str(min) + ' ' + str(sec) + ') ')
```

## [#14. Modular Calculator](https://www.codeabbey.com/index/task_view/modular-calculator)

```python
result = int(input())
flag = True
lastval = 0

while flag:
    sign, val = [x for x in input().split()]
    if sign == '*':
        result *= int(val)
    elif sign == '+':
        result += int(val)
    elif sign == '%':
        flag = False
        lastval = int(val)
        
print(str(result % lastval))    
```

## [#27. Bubble Sort](https://www.codeabbey.com/index/task_view/bubble-sort)

```python
len = int(input())
input_list = [int(x) for x in input().split()]
count1 = 0
count2 = 0
flag = True
while flag:
    flag = False
    count1 += 1
    for i in range(1, len):
        if input_list[i] < input_list[i - 1]:
            input_list[i], input_list[i - 1] = input_list[i - 1], input_list[i]
            flag = True
            count2 += 1
print(str(count1) + ' ' + str(count2))
```

## [#26. Greatest Common Divisor](https://www.codeabbey.com/index/task_view/greatest-common-divisor)

```python
len = int(input())

def gcd(a,b):  
    a, b = (a, b) if a >=b else (b, a)
    if a%b == 0:  
        return b  
    else :  
        return gcd(b,a%b)

def lcm(a,b):
    return a*b//gcd(a,b)

for i in range(len):
    input_list = [int(x) for x in input().split()]
    print('(' + str(gcd(input_list[0], input_list[1])) + ' ' + str(lcm(input_list[0], input_list[1])) + ') ')
```

## [#29. Sort With Indexes](https://www.codeabbey.com/index/task_view/sort-with-indexes)

```python
from copy import deepcopy
len = int(input())

input_list = [int(x) for x in input().split()]

sorted_list = deepcopy(input_list)
sorted_list.sort()

for ele in sorted_list:
    print(str(input_list.index(ele)+1) + ' ')
```

## [#18. Sequare Root](https://www.codeabbey.com/index/task_view/square-root)

```python
len = int(input())

for ele in range(len):
    result = 1
    input_list = [int(x) for x in input().split()]
    if input_list[1] == 0:
        print('1 ')
    else :
        for i in range(input_list[1]):
            result = round((result + input_list[0]/result)/2, 7)
        print(str(result) + ' ')
```

## [#23. Bubble in Array](https://www.codeabbey.com/index/task_view/bubble-in-array)

```python
input_list = [int(x) for x in input().split()]

count = 0

for j in range(1, len(input_list)-1):
    i = j - 1
    if input_list[i] > input_list[j]:
        input_list[i], input_list[j] = input_list[j], input_list[i]
        count += 1
        
result = 0
for ele in input_list:
    if ele == -1:
        break
    else:
        result = ((result + ele) * 113) % 10000007

print(str(count) + ' ' + str(result) + ' ')
```

## [#50. Palindromes](https://www.codeabbey.com/index/task_view/palindromes)

```python
import string
len = int(input())

def format_str(s):
    exclude = set(string.punctuation + ' ')
    s = ''.join(ch for ch in s if ch not in exclude)
    return s.lower()

for i in range(len):
    input_str = format_str(input())
    if input_str == input_str[::-1]:
        print('Y ')
    else:
        print('N ')
```

## [#31. Rotate String](https://www.codeabbey.com/index/task_view/rotate-string)

```python
len = int(input())

for i in range(len):
    input_list = [x for x in input().split()]
    num = int(input_list[0])
    print(input_list[1][num:] + input_list[1][:num] + ' ')
```

## [#24. Neumann's Random Generator](https://www.codeabbey.com/index/task_view/neumanns-random-generator)

```python
nums = int(input())

input_list = [x for x in input().split()]

def get_num(x):
    result = str(int(x) * int(x))
    if len(result) < 8:
        result = ' ' * (8 - len(result)) + result
    return result[2:6]
    
def get_check_num(x):
    result = []
    count = 1
    num = get_num(x)
    while(True):
        if num in result:
            return str(count)
        else:
            result.append(num)
            count = count + 1
            num = get_num(num)
    

for i in range(nums):
    print(get_check_num(input_list[i]) + ' ')
```

## [#67. Fibonacci Sequence](https://www.codeabbey.com/index/task_view/fibonacci-sequence)

```python
cases = int(input())

def fibonacci_counter(cases):
    
    for i in range(cases):
        value = int(input())
        counter = 0
        a, b = 0, 1
        
        while a != value:
            a, b = b, a + b
            counter += 1
            
        print(counter, end=' ')
        
fibonacci_counter(cases)
```

## [#52. Pythagorean Theorem](https://www.codeabbey.com/index/task_view/pythagorean-theorem)

```python
nums = int(input())

for i in range(nums):
    input_list = [int(x) for x in input().split()]
    input_list.sort()
    if (pow(input_list[0],2) + pow(input_list[1],2)) == pow(input_list[2],2):
        print('R ')
    elif (pow(input_list[0],2) + pow(input_list[1],2)) > pow(input_list[2],2):
        print('A ')
    elif (pow(input_list[0],2) + pow(input_list[1],2)) < pow(input_list[2],2):
        print('O ')
```

## [#68. Bicycle Race](https://www.codeabbey.com/index/task_view/bicycle-race)

```python
nums = int(input())

for i in range(nums):
    input_list = [int(x) for x in input().split()]
    value = (input_list[0] * input_list[1]) / (input_list[1] + input_list[2]) 
    print(str(round(value, 8)), end = ' ')
```

## [#32. Josephus Problem](https://www.codeabbey.com/index/task_view/josephus-problem)

```python
input_list = [int(x) for x in input().split()]

s = 0

for i in range(2, input_list[0]+1):
    s = (s + input_list[1]) % i
    
print(str(s + 1))
```

## [#35. Savings Calculator](https://www.codeabbey.com/index/task_view/savings-calculator)

```python
nums = int(input())

for i in range(nums):
    input_list = [int(x) for x in input().split()]
    money = input_list[0]
    year = 0
    while(money < input_list[1]):
        money = money + money*input_list[2]/100
        year += 1
    print(str(int(year)), end = ' ')
```

## [#25. Linear Congruential Generator](https://www.codeabbey.com/index/task_view/linear-congruential-generator)

```python
nums = int(input())

def get_num(a, c, m, x, n):
    result = x
    for i in range(n):
        result = (a * result + c) % m
    return result

for i in range(nums):
    input_list = [int(x) for x in input().split()]
    res = get_num(input_list[0], input_list[1], input_list[2], input_list[3], input_list[4])
    print(str(res), end = ' ')
```

## [#44. Double Dice Roll](https://www.codeabbey.com/index/task_view/double-dice-roll)

```python
nums = int(input())


def get_double_dice_roll():
    DICE = 6
    for i in range(nums):
        input_list = [int(x) for x in input().split()]
        first_dice = (input_list[0] % 6) + 1
        second_dice = (input_list[1] % 6) + 1
        total = first_dice + second_dice
        print(str(total) + ' ')

get_double_dice_roll()
```

## [#19. Matching Brackets](https://www.codeabbey.com/index/task_view/matching-brackets)

```python
nums = int(input())

brackets_start = ['<', '(', '{', '[']
brackets_end = ['>', ')', '}', ']']


def check_brackets_end(tmp_brackets, test_char):
    if len(tmp_brackets) == 0:
        return False
    if brackets_end.index(test_char) == brackets_start.index(tmp_brackets[-1]) :
        tmp_brackets.pop()
        return True
    else :
        return False

def check_brackets(test_str):
    tmp_brackets = []
    end_flag = 0
    for i in range(len(test_str)):
        #print(test_str[i])
        if test_str[i] in brackets_start:
            #print(test_str[i])
            tmp_brackets.append(test_str[i])
            #print(tmp_brackets)
        elif test_str[i] in brackets_end:
            #print(test_str[i])
            if check_brackets_end(tmp_brackets, test_str[i]) == False:
                end_flag = 1
                break
            
    if end_flag==1:
        print('0 ')
    elif len(tmp_brackets):
        print('0 ')
    else :
        print('1 ')

for i in range(nums):
    input_str = str(input())
    check_brackets(input_str)
```

## [#81. Bit Count](https://www.codeabbey.com/index/task_view/bit-count)

其原理是不断清除n的二进制表示中最右边的1，同时累加计数器，直至n为0。如果n的二进制表示中有k个1，那么这个方法只需要循环k次即可。代码如下：

```java
// Java
import java.util.Scanner;

public class BitCount_81 {
    public static void main(String[] args) {
        BitCount_81 s = new BitCount_81();
        Scanner input = new Scanner(System.in);
        int numberOfLines = input.nextInt();

        while(input.hasNext()) {
            int tmp = input.nextInt();
            int c = 0;
            //利用减一操作，去除低位第一个为1的位，循环执行，直到结果最终为0
            for (c = 0; tmp != 0; ++c) {
                tmp &= (tmp - 1);
            }
            System.out.println(c + " ");
        }
    }
}
```

为什么n &= (n – 1)能清除最右边的1呢？因为从二进制的角度讲，n相当于在n - 1的最低位加上1。举个例子，8（1000）= 7（0111）+ 1（0001），所以8 & 7 = （1000）&（0111）= 0（0000），清除了8最右边的1（其实就是最高位的1，因为8的二进制中只有一个1）。再比如7（0111）= 6（0110）+ 1（0001），所以7 & 6 = （0111）&（0110）= 6（0110），清除了7的二进制表示中最右边的1（也就是最低位的1）。

## [#47. Caesar Shift Cipher](https://www.codeabbey.com/index/task_view/caesar-shift-cipher)

```java
//Java
import java.util.Scanner;

public class CaesarShiftCipher47 {

    public static void main(String[] args) {

        String tmp = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

        Scanner input = new Scanner(System.in);
        int numberOfLines = input.nextInt();
        int count = input.nextInt();

        while(input.hasNext()) {
            String line = input.nextLine();
            StringBuilder sb = new StringBuilder(line);
            for (int i = 0; i < line.length(); i++) {
                if(line.charAt(i) == ' ' || line.charAt(i) == '.') {
                    continue;
                }
                int index = tmp.indexOf(line.charAt(i));
                sb.setCharAt(i, tmp.charAt((index + 26 * (count/26+1) - count) % 26));
            }
            System.out.println(sb.toString() + " ");
        }
    }

}
```

## [#104. Triangle Area](https://www.codeabbey.com/index/task_view/triangle-area)

```java
//Java
import java.text.DecimalFormat;
import java.util.Scanner;

public class TriangleArea104 {

    public static void main(String[] args) {

        Scanner input = new Scanner(System.in);
        int numberOfLines = input.nextInt();

        while(input.hasNext()) {
            int x1 = input.nextInt();
            int y1 = input.nextInt();
            int x2 = input.nextInt();
            int y2 = input.nextInt();
            int x3 = input.nextInt();
            int y3 = input.nextInt();


            double a = Math.sqrt((x1 - x2) * (x1 - x2) + (y1 - y2) * (y1 - y2));
            double b = Math.sqrt((x1 - x3) * (x1 - x3) + (y1 - y3) * (y1 - y3));
            double c = Math.sqrt((x2 - x3) * (x2 - x3) + (y2 - y3) * (y2 - y3));

            double s = (a + b + c) / 2;

            double result = Math.sqrt(s * (s - a) * (s - b) * (s - c));
            DecimalFormat df = new DecimalFormat("0.0");
            System.out.println(df.format(result) + " ");
        }
    }
}

```

## [#55. Matching Words](https://www.codeabbey.com/index/task_view/matching-words)

```java
//Java
import java.util.*;

public class MatchingWords55 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        String line = input.nextLine();
        String line_split[] = line.split(" ");
        List<String> list = new ArrayList<>();
        for(String word : line_split){
            String tmp[] = line.split(word);
            //只添加重复出现两次以上的单词
            if (tmp.length - 1 > 1) {
                if (list.indexOf(word) == -1)
                list.add(word);
            }
        }
        //对列表进行排序
        Collections.sort(list);
        //method1: not ok in jdk7
//        System.out.println(String.join(" ", list));

        //method2: ok
        for (String word : list) {
            System.out.print(word + " ");
        }
        //method3: ok
//        StringBuilder sb = new StringBuilder();
//        for (String word: list) {
//            sb.append(word + " ");
//        }
//        System.out.println(sb.toString());
    }
}
```

## [#58. Card Names](https://www.codeabbey.com/index/task_view/card-names)

```java
// Java
package codeabby;

import java.util.ArrayList;
import java.util.Scanner;

public class CardNames58 {
    static final ArrayList<String> suits = new ArrayList<String>(){{add("Clubs"); add("Spades"); add("Diamonds"); add("Hearts");}};
    static final ArrayList<String> ranks = new ArrayList<String>(){{add("2"); add("3"); add("4"); add("5");
        add("6"); add("7"); add("8"); add("9"); add("10"); add("Jack"); add("Queen"); add("King"); add("Ace");}};

    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        int nums = input.nextInt();
        input.nextLine();
        String line[] = input.nextLine().split(" ");
        for (String word: line) {
            int number = Integer.parseInt(word);
            int suit_value = number / 13;
            int rank_value = number % 13;
            System.out.print(ranks.get(rank_value) + "-of-" + suits.get(suit_value) + " ");
        }
    }
}
```

## [#61. Prime Numbers Generation](https://www.codeabbey.com/index/task_view/prime-numbers-generation)

参考文章：[使用java语言产生素数-译](https://jiaoqiyuan.cn/2019/01/18/%E4%BD%BF%E7%94%A8java%E8%AF%AD%E8%A8%80%E4%BA%A7%E7%94%9F%E7%B4%A0%E6%95%B0-%E8%AF%91/)

```java
package codeabby;

import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;
import java.util.Scanner;

public class PrimeNumberGeneration61 {

    private static List<Integer> primeList(int n) {
        boolean prime[] = new boolean[n + 1];
        //先将数组全部填为true
        Arrays.fill(prime, true);
        //精华所在，将范围n之内的数字判断之后标记，如果是素数就为true，如果是非素数就为false
        for (int i = 2; i * i < n; i++) {
            if (prime[i]) {
                for (int j = i + i; j < n; j = i + j) {
                    prime[j] = false;
                }
            }
        }

        List<Integer> primeNumbers = new LinkedList<>();
        for (int i = 2; i <= n; i++) {
            if (prime[i]) {
                primeNumbers.add(i);
            }
        }

        return primeNumbers;
    }

    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        int nums = input.nextInt();
        input.nextLine();
        String line = input.nextLine();
        List<Integer> list = primeList(10000000);
        for (String word : line.split(" ")) {
            int number = Integer.parseInt(word);
            System.out.print(list.get(number - 1) + " ");
        }

    }
}
```

## [#94. Fool's Day 2014](https://www.codeabbey.com/index/task_view/fools-day-2014)

作者给出了一串数字让自己找规律，很简单，就是每行数字的平方和。

```java
package codeabby;

import java.util.Scanner;

public class FoolsDay94 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();

        while (input.hasNextLine()) {
            String line = input.nextLine();
            int sum = 0;
            for (String word : line.split(" ")) {
                sum += Integer.parseInt(word) * Integer.parseInt(word);
            }
            System.out.println(sum + " ");
        }
    }
}
```

## [#59. Bulls and Cows](https://www.codeabbey.com/index/task_view/bulls-and-cows)

考察关于字符串与字符之间转换的问题，逻辑很简单，比较重视基础。

```java
package codeabby;

import java.util.Scanner;

public class BullsAndCows59 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        String standardNumber = input.next();
        input.nextLine();
        String line = input.nextLine();
        for (String word : line.split(" ")) {
            int correctPosition = 0;
            int wrongPosition = 0;
            for (int i = 0; i < word.length(); i++) {
                //如果相同位置的数字相等，就将第一个值加1
                if (word.charAt(i) == standardNumber.charAt(i)) {
                    correctPosition += 1;
                //如果相同位置的数字不等，就判断一下这个数字在给出的standardNumber中是否存在，如果存在就将第二个值加1
                } else if (standardNumber.indexOf(word.charAt(i)) != -1) {
                    wrongPosition += 1;
                }
            }
            System.out.println(correctPosition + "-" + wrongPosition + " ");
        }
    }
}
```

## [#22. Two Printers](https://www.codeabbey.com/index/task_view/two-printers)

题意是有两个打印机，二者打印速度不同，打印机1的打印速度是X秒每张，打印机2的速度是Y秒每张，两台打印机共同打印N页值，求最短打印时间。这里采用的是穷举法，找出每种情况下打印的时间，取最小值。

```java
package codeabby;

import java.util.Scanner;

public class TwoPrinters22 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        while (input.hasNextLine()) {
            String words[] = input.nextLine().split(" ");
            int x_printer = Integer.parseInt(words[0]);     //X值
            int y_printer = Integer.parseInt(words[1]);     //Y值
            int pages = Integer.parseInt(words[2]);         //打印页数
            int time = 0;                                   //打印总时间
            //穷举法，将x打印页数从0到pages的情况全部列出来，找出时间最短的那种情况
            for (int i = 0; i < pages; i++) {
                int tmp_time = 0;
                //如果x打印机最终花费了较长的时间，就以x打印机的时间为准
                if (i * x_printer >= (pages - i) * y_printer) {
                    tmp_time = i * x_printer;
                //如果y打印机花费了较长时间，就以y打印机的时间为准
                } else {
                    tmp_time = (pages - i) * y_printer;
                }
                //如果是第一次循环（time=0）或者当前情况下时间比之前记录的最小时间短，就更新最短时间.
                if (time == 0 || tmp_time < time) {
                    time = tmp_time;
                }
            }
            System.out.println(time + " ");

        }
    }
}
```

## [#128. Combinations Counting](https://www.codeabbey.com/index/task_view/combinations-counting)

使用Java的BigInteger计算，不然存不下。

```java
package codeabby;

import java.math.BigInteger;
import java.util.*;

public class CombinationsCounting128 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        while (input.hasNextLine()) {
            String[] words = input.nextLine().split(" ");
            int N = Integer.parseInt(words[0]);
            int K = Integer.parseInt(words[1]);
            BigInteger njiecheng = new BigInteger("1");
            BigInteger kjiecheng = new BigInteger("1");
            BigInteger n_kjiecheng = new BigInteger("1");
            for (int i = 1; i <= N; i++) {
                njiecheng = njiecheng.multiply(new BigInteger(String.valueOf(i)));
            }

            for (int i = 1; i <= K; i++) {
                kjiecheng = kjiecheng.multiply(new BigInteger(String.valueOf(i)));
            }

            for (int i = 1; i <= (N - K); i++) {
                n_kjiecheng = n_kjiecheng.multiply(new BigInteger((String.valueOf(i))));
            }
            System.out.println(njiecheng.divide(kjiecheng.multiply(n_kjiecheng)) + " ");
        }
    }
}
```


## [#42. Blackjack Counting](https://www.codeabbey.com/index/task_view/blackjack-counting)

```java
package codeabby;

import java.util.Scanner;

public class BlackjackCounting42 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        while (input.hasNextLine()) {
            String[] words = input.nextLine().split(" ");
            int sum = 0;
            int countA = 0;     //记录A的个数
            for (String word:
                 words) {
                if (word.compareTo("2") == 0 || word.compareTo("3") == 0 || word.compareTo("4") == 0
                        || word.compareTo("5") == 0 || word.compareTo("6") == 0
                        || word.compareTo("7") == 0 || word.compareTo("8") == 0
                        || word.compareTo("9") == 0) {
                    sum += Integer.parseInt(word);
                } else if (word.compareTo("T") == 0 || word.compareTo("J") == 0
                || word.compareTo("Q") == 0 || word.compareTo("K") == 0) {
                    sum += 10;
                } else if (word.compareTo("A") == 0) {
                   countA ++;
                }
            }
            if (sum > 21) {
                System.out.println("Bust ");
            } else {
                //处理A的情况，看最多能有几个A等于11，前提条件是A等于11后不能让总和大于21
                int n = 0;
                for (int i = 1; i <= countA; i++) {
                    if (sum + i*11 <= 21) {
                        n ++;
                    } else {
                        break;
                    }
                }
                sum += n * 11 + (countA - n) * 1;
                System.out.println(sum);
                if (sum > 21) {
                    System.out.println("Bust ");
                } else {
                    System.out.println(sum + " ");
                }
            }
        }
    }
}
```
## [#38. Quadratic Equation](https://www.codeabbey.com/index/task_view/quadratic-equation)

本题是求二次方程的解，需要考虑虚数，套公式就可以了，注意最后打印时整形数字与字符串的转换就行了

```java
package codeabby;

import java.util.Scanner;

public class QuadraticEquation38 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        while (input.hasNextLine()) {
            String[] words = input.nextLine().split(" ");
            int a = Integer.parseInt(words[0]);
            int b = Integer.parseInt(words[1]);
            int c = Integer.parseInt(words[2]);

            if ((b * b - 4 * a * c) < 0) {
                int tmp = (b * b - 4 * a * c) * -1;
                System.out.print(-b/(2*a) + "+" + (int)Math.sqrt(tmp)/(2*a) + "i " + -b/(2*a) + "-" + (int)Math.sqrt(tmp)/(2*a) + "i");
                if (input.hasNextLine()) {
                    System.out.print("; ");
                }

            } else {
                double tmp = Math.sqrt((double) (b * b - 4 * a * c));
                int x1 = (int)(-b + tmp) / (2*a);
                int x2 = (int)(-b - tmp) / (2*a);
                System.out.print(x1 + " " + x2);
                if (input.hasNextLine()) {
                    System.out.print("; ");
                }
            }
        }
    }
}

```

## [#34. Binary Search](https://www.codeabbey.com/index/task_view/binary-search)

这一题考察的是二分法查询算法，以及使用小数处理时精度范围的选取。

```java
package codeabby;

import java.util.Scanner;

public class BinarySearch34 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        while (input.hasNextLine()) {
            String[] numbers = input.nextLine().split(" ");
            double a = Float.parseFloat(numbers[0]);
            double b = Float.parseFloat(numbers[1]);
            double c = Float.parseFloat(numbers[2]);
            double d = Float.parseFloat(numbers[3]);

            double start = 0;
            double end  = 100;
            double x = 100;
            double result = 0;

            while (true) {
                result = (a * x + b * Math.sqrt(x*x*x) - c * Math.exp(-x / 50) - d);

                if (result > 0.00000001) {
                    end = x;
                    x = (start + end) / 2;
                } else if (result < -0.00000001) {
                    start = x;
                    x = (start + end) / 2;
                } else {
                    System.out.println(x + " ");
                    break;
                }
            }
        }
    }
}
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
[10]: https://www.codeabbey.com/index/task_view/vowel-count
[11]: https://www.codeabbey.com/index/task_view/sum-of-digits
[12]: https://www.codeabbey.com/index/task_view/median-of-three
