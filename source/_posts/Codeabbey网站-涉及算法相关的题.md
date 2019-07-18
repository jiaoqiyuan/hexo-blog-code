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

## [46. Tic-Tac-Toe](https://www.codeabbey.com/index/task_view/tic-tac-toe)

这道题有点像五子棋，只不过棋子是三个。

我的解题思路是将所有可能的情况列举出来，每走一步判断一下是否有三子共线。

```java
import java.util.*;

public class TicTacToe46 {
    static List<List<String>> allres;
    public static void format_wins() {
        List<String> w1 = Arrays.asList("1", "2", "3");
        List<String> w2 = Arrays.asList("4", "5", "6");
        List<String> w3 = Arrays.asList("7", "8", "9");
        List<String> w4 = Arrays.asList("1", "4", "7");
        List<String> w5 = Arrays.asList("2", "5", "8");
        List<String> w6 = Arrays.asList("3", "6", "9");
        List<String> w7 = Arrays.asList("1", "5", "9");
        List<String> w8 = Arrays.asList("3", "5", "7");
        allres = Arrays.asList(w1, w2, w3, w4, w5, w6, w7, w8);
    }

    public static boolean check(List<String> player) {

        for (List<String> wins : allres) {
            if (player.containsAll(wins)) {
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        input.nextLine();
        format_wins();
        while (input.hasNextLine()) {
            String inputs[] = input.nextLine().split(" ");
            List<String> playerA = new ArrayList<>();
            List<String> playerB = new ArrayList<>();
            boolean flag = false;
            for (int i = 0; i < inputs.length; i++) {
                if (i % 2 == 0) {
                    playerA.add(inputs[i]);
                } else {
                    playerB.add(inputs[i]);
                }
                if (i < 4) {
                    continue;
                } else {
                    if (check(playerA) || check(playerB)) {
                        System.out.print(i + 1 + " ");
                        flag = true;
                        break;
                    }
                }
            }
            if (!flag) {
                System.out.println("0 ");
            }
        }
    }
}
```

别人不错的方法，将表格映射成 char 数组，将棋子映射为 `X` 和 `O`，未放置棋子的映射为 `-` ，然后判断：

```java
package codeabby;
import java.util.Scanner;

public class TicTacToe46_good {
    private static Scanner in;

    public static void main(String[] args){
        in = new Scanner(System.in);
        int N=in.nextInt();
        for(int i=0;i<N;i++){
            int input[]=new int[9];
            char grid[]=new char[9];
            for(int j=0;j<9;j++){
                grid[j]='-';
                input[j]=in.nextInt();
            }
            int step=1;
            for(int j=0;j<9;j++){
                if((j+1)%2!=0) grid[input[j]-1]='X';
                else grid[input[j]-1]='O';
                if(!isTris(grid))step++;
            }
            if(!isTris(grid))step=0;
            System.out.print(step+" ");
        }

    }

    public static Boolean isTris(char[] g){
        for(int k=0;k<9;k=k+3){
            if(g[k]==g[k+1] && g[k]==g[k+2] && g[k]!='-'){
                return true;
            }
        }
        for(int k=0;k<3;k++){
            if(g[k]==g[k+3] && g[k]==g[k+6] && g[k]!='-'){
                return true;
            }
        }
        if(g[0]==g[4] && g[0]==g[8] && g[0]!='-') return true;
        if(g[2]==g[4] && g[2]==g[6] && g[2]!='-') return true;
        return false;
    }
}
```


[1]: https://www.codeabbey.com/