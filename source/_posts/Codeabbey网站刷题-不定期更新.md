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


...


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
