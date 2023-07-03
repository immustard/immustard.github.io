# 二分法查找


<!--more-->



> 前两天在[LeetCode](https://leetcode-cn.com/)做题的时候, 做到了二分查找(Binary Search). 现在来做一下梳理. 
>
> 地址: [LeetCode-Binary Search](https://leetcode-cn.com/problems/binary-search/)



二分查找作为程序员的一个基本技能, 也是面试的时候有可能面试官会问到的一种算法. 可以达到`O(log n)`的时间复杂度. 

一般来说, 当出现这几个条件的时候, 就应该用到二分查找: 

1. 待查找的数组是有序的或者是部分有序的
2. 要求时间复杂度低于`O(n)`, 或者直接说明时间复杂度是`O(log n)`



而且二分查找也有很多的变体. 在使用的时候, 要注意好**查找条件**, **判断条件**以及**左右边界的条件变更方式**. 这三个地方没有注意好, 很容易就会出现死循环或是遗漏. 今天来梳理一下这几种: 

1. 标准的二分查找
2. 二分查找左边界
3. 二分查找右边界
4. 二分查找的左右边界
5. 二分查找极值点



## 标准的二分查找

上面这个题目就用标准的二分查找来实现, 我们先来看标准的二分查找的模板: 

```java
public int search(int[] array, int target) {
  int left = 0, right = array.length-1;
  // 1. 因为循环中包含了 left == right 的条件, 所以每次循环的时候, left或right都要有变化
  while (left <= right) {
    // 这句话其实等同于 (right+left)/2, 但是这样的写法可以避免溢出
    int mid = ((right - left) >> 1) + left;
    
    if (array[mid] == target) {
      return mid;
    } else if (array[mid] < target) {
      // 左边界更新
      left = mid + 1;
    } else {
      // 右边界更新
      right = mid - 1;
    }
  }
  // 未查找到目标值
  return -1;
}
```



## 二分查找左边界

使用这种变体的时候通常有这几种特性: 

1. 数组有序, 包含重复元素
2. 数组部分有序, 包含重复元素
3. 数组部分有序, 不包含重复元素



### 二分查找左边界①

这种分类包含了上面的`1`,`3`两种情况. 

既然是查找左边界, 就要从右侧开始, 然后不断左移. 也就是说, 即使找到了`array[mid] == target`, 这个`mid`值也不见得就是要查找的左边界. 所以还是要继续收缩:

```java
public int search(int[] array, int target) {
  int left = 0, right = array.length-1;
  
  while (left < right) {
    int mid = ((right - left) >> 1) + left;
				
    if (array[mid] < target) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }
  return array[left] == target ? left : -1;
}
```

可以很明显的看出来和标准的有何不同: 

1. 查找条件变为了`left < right`

   因为在最后`left`与`right`相邻的时候, `mid`和`left`处在同一位置. 所以下一步, `left`, `mid`, `right`都会在同一位置. 也就是说, 如果判断条件还是`left <= right`的话, 可能最后就会进入死循环. 

2. 右边界更新为`right = mid`

   因为需要在查找到目标值之后继续向左移动. 



### 二分查找左边界②

这种分类包含了上面的`2`情况, 也就是数组部分有序, 包含重复元素. 这种条件下, 右边界在向左移动的时候, 不能简单的令`right = mid`. 因为有重复的元素, 这样就可能会造成遗漏: 

```java
public int search(int[] array, int target) {
  int left = 0, right = array.length-1;
  
  while (left < right) {
    int mid = ((right - left) >> 1) + left;
    if (array[mid] < target) {
      left = mid + 1;
    } else if (array[mid] > target) {
      right = mid;
    } else {
      --right;
    }
  }
  return array[left] == target ? left : -1;
}
```







---

> 作者:   
> URL: https://buli-home.cn/binarysearch/  

