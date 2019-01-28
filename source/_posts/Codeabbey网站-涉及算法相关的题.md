---
title: Codeabbey网站-涉及算法相关的题
date: 2019-01-25 14:15:22
tags: [算法, codeabbey]
categories: 算法
photos: https://www.codeabbey.com/img/facade.gif
thumbnail: https://www.codeabbey.com/img/facade.gif
---

> 网站：[Codeabbey][1]

> 声明：这是算法相关的题，难度开始增加了。

> 补充：从这里开始全部是Java来写，以后的工作肯定是写Java多一些，所以也先练习一下Java。

------

<!--more-->

## [#120. Selection Sort](https://www.codeabbey.com/index/task_view/selection-sort)

基于选择排序，遍历多遍数据，每次选出一个最大值，放到最后，该题的目的是找出每次找到最大值后，输出最大值在列表中的索引位置。

```java
package codeabby;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class SelectionSort120 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        String[] numbers = input.nextLine().split(" ");
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < numbers.length; i++) {
            list.add(Integer.parseInt(numbers[i]));
        }
        int index = 0;
        while (list.size() != 1) {
            int max = 0;
            for (int i = 0; i < list.size(); i++) {
                if (list.get(i) > max) {
                    max = list.get(i);
                    index = i;
                }
            }
            //将列表最后一个数放入最大值的索引位置
            list.set(index, list.get(list.size() - 1));
            //删除最后一个元素
            list.remove(list.size() - 1);
            System.out.println(index + "   " + max);
        }
    }
}
```

## [#53. King and Queen](https://www.codeabbey.com/index/task_view/king-and-queen)

题意是给出一个8 x 8的棋谱，在上面放置King和Queen，看给出的坐标的King能不能被Queen吃掉，规则是只要King和Queen在同一行或同一列甚至是在一条对角线上，King就可以被Queen吃掉。

这里做法是把给出的坐标先解析，分解出King(x1, y1)和Queen(x2,y2)，如果x1==x2或者y1 == y2，或者|x1 - x2| == |y1 - y2|，King就能被Queen吃掉，否则不能。

```java
package codeabby;

import java.util.Scanner;

/**
 * 考察java获取char，绝对值的获取等知识。
 */
public class KingAndQueen53 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        while (input.hasNextLine()) {
            String[] words = input.nextLine().split(" ");
            char x1 = words[0].charAt(0);
            int y1 = Character.getNumericValue(words[0].charAt(1));
            char x2 = words[1].charAt(0);
            int y2 = Character.getNumericValue(words[1].charAt(1));
            if ((Math.abs(x1 - x2) == Math.abs(y1 - y2)) || x1 == x2 || y1 == y2) {
                System.out.println("Y ");
            } else {
                System.out.println("N ");
            }

        }
    }
}
```

## [#45. Cards Shuffling](https://www.codeabbey.com/index/task_view/cards-shuffling)

这里就是洗牌的操作，给出一串数字，将对应位置上的卡牌移动到相应位置上。

```java
package codeabby;

import java.util.ArrayList;
import java.util.Scanner;

public class CardShuffling45 {
    //创建固定列表存放花色和纸牌数值
    private static final ArrayList<String> ranks = new ArrayList<String>(){{add("A"); add("2"); add("3"); add("4"); add("5");
        add("6"); add("7"); add("8"); add("9"); add("T"); add("J"); add("Q"); add("K");}};
    private static final ArrayList<String> suits = new ArrayList<String>(){{add("C"); add("D"); add("H"); add("S");}};

    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        String[] words = input.nextLine().split(" ");
        String[] output = new String[52];
        //初始化output，按顺序将纸牌放入列表中
        for (int i = 0; i < output.length; i++) {
            int rank_index = i % 13;
            int suit_index = i / 13;
            String card_name = suits.get(suit_index) + ranks.get(rank_index);
            output[i] = card_name;
        }
        //根据指定数字洗牌
        for (int i = 0; i < words.length; i++) {
            int index = Integer.parseInt(words[i]) % 52;
            //交换两个数
            String tmp = output[index];
            output[index] = output[i];
            output[i] = tmp;
        }
        for (String card :
                output) {
            System.out.print(card + " ");
        }
    }
}
```


[1]: https://www.codeabbey.com/