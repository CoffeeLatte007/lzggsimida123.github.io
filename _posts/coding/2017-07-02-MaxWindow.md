---
layout: post
title: "生成窗口最大值数组"
date: 2017-06-28 09:55:18 +0800
categories: 语言
tag: 算法
---

* content
{:toc}


    有一个整型数组arr和一个大小为w的窗口从数组的最左边滑到最右边，窗口每次向右边滑一个位置。
　　    <!-- more -->
  
# 题目
有一个整型数组arr和一个大小为w的窗口从数组的最左边滑到最右边，窗口每次向右边滑一个位置。

例如，数组为[4,3,5,4,3,3,6,7]，窗口大小为3时:

[4 3 5] 4 3 3 6 7    　max = 5

4 [3 5 4] 3 3 6 7    　max = 5

4 3 [5 4 3] 3 6 7    　max = 5

4 3 5 [4 3 3] 6 7    　max = 4

4 3 5 4 [3 3 6] 7    　max = 6

4 3 5 4 3 [3 6 7]    　max = 7

如果数组长度为n,窗口大小为w，则一共产生n-w+1个窗口的最大值。
请实现一个函数。

input:整型数组arr,窗口大小为w。

out:一个长度为n-w+1的数组res,res[i]表示每一种窗口状态下的最大值。

上面的结果应该返回{5,5,5,4,6,7}

# 思路

假定数组长度为N，窗口大小为w，我们可以通过两层for循环完成O(NxW)的时间复杂度，这种方法比较简单不予简述。

本题有时间复杂度为O(n)的方法，只要记录了以前的值就可以完成，使用双端队列的方式，即可完成。现有一个双向队列名为qmax，在qmax中保存下标，在java中使用LinkedList可以完成双向队列的工作，在qmax的队头用来保存我们当前的最大值的坐标，当队头的下标等于i-w的时候，就过期，直接弹出,这也是我们的最大值。qmax插入的规则如下:

当我们遍历到数组的第i个元素的时候，有如下规则:

1. 如果qmax为空，直接插入i。
2. 如果qmax不为空，则获取qmax队尾的坐标，记为j。
3. 如果a[j]>a[i],说明队尾的值比他数组大，则直接插入队列。
4. 如果a[j]<=a[i]，需要把j从qmax弹出，然后会到第二步，再决定插入的策略。
5. 过期计算:如果qmax队头的下标等于i-w，说明过期，直接弹出。
6. 计算res值，当i-w>=0的时候就可以记录res的值了。

qmax在其中的责任是负责维护窗口为w的子数组的最大值更新。

# 代码

```
public static int[] getMaxWindow(int[] arr,int w){
        if (arr == null || arr.length < w || w < 1){
            return null;
        }
        LinkedList<Integer> qmax = new LinkedList<Integer>();//用来记录最大值的更新
        int[] res = new int[arr.length-w+1];//用来记录结果
        int index = 0;
        for (int i = 0; i < arr.length; i++) {
            //1到4步骤的过程
            while (!qmax.isEmpty() && arr[qmax.peekLast()] <= arr[i]){
                qmax.pollLast();
            }
            qmax.addLast(i);

            //第5步:过期计算
            if (qmax.peekFirst() == i-w){
                qmax.pollFirst();
            }
            //第6步:记录res值
            if (i-w >= -1){
                res[index++]=arr[qmax.peekFirst()];
            }
        }
        return res;
    }

    public static void main(String[] args) {
        int[] arr = new int[]{4,3,5,4,3,3,6,7};
        int[] res = getMaxWindow(arr,3);
        for(int r : res){
            System.out.println(r);
        }
    }
```






  
   

<br>