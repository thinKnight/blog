title: 简单分治
date: 2014-03-24 17:54:45
tags: Algorithm
---
在之前的排序算法中，我们提到了一种叫归并排序的算法，就是通过把待排序的数列分成一个个小的数列并排序成有序表，再将这些小的有序表组合成一个新的有序表来完成排序，所以归并排序算法就是利用分治策略的一个很好的典型。  
<!-- more -->

# 概念阐述  

在分治策略中我们用递归的方法来求解一个问题，在每一层的递归中应用如下的三个步骤：
- 分解(Dived)步骤将问题划分为一些子问题，子问题的形式与原问题一样，只是规模更小。
- 解决(Conquer)步骤递归地求解出子问题。如果子问题的规模足够小，则停止递归，直接求解。
- 合并(Combine)步骤将子问题的解组合成原问题的解。  

简单通俗地来说，所谓的**分治策略**就是把很复杂的问题用递归的方式细化成一个个小的，能够解决子问题，再把所有子问题的解合并起来，就得到原问题的解。  

下面就通过一个例子来简单介绍分治策略。  

# 实例问题

这里有一个数组A：
> `[13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7]`  

现在要求寻找数组A中和最大的非空连续子数组（我们称这样的连续子数组为**最大子数组**）。

## 暴力求解法

乍一看到这个问题，首先想到的当然就是简单粗暴地套上双重循环，尝试每对可能的组合，再进行比较，得出最终结果，这就是我们常说的暴力求解法，算法实现如下：  


```C
//最大子数组问题：暴力求解法
#include <stdio.h>
int main()
{
    int A[] = { 13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5,     22, 15, -4, 7 };
    int length = sizeof( A ) / sizeof( int );
    int max = -1000;
    int i, j, k, sum, step, left;
    for( i = 1; i <= length; i++ ) 
    {
        for( j = 0; ( j + i -1 ) <= length-1; j++ )  
        {
            sum = 0;
            for( k = j; k <= j + i - 1; k++ ) 
                sum += A[ k ];
            if( sum > max )
            {
                max = sum;
                left = j;
                step = i;
            }
        }
    }
    printf("length:%d, max:%d, left:%d, right:%d, step:%d\n", length, max, left, left+step-1, step);
    getchar();
    return 0;
}
```  

---

但是套上两个循环后，算法的时间复杂度就变为了O(n²)，我们还有更好的方法。  

---

## 分治策略求解法

现在我们假设要寻找子数组`A[low..high]`的最大子数组，使用分治就意味着要把数组A分成两个规模尽量相等的子数组，也就是说找到数组的中央位置mid，然后考虑求解两个子数组`A[low..mid]`和`A[mid+1..high]`。这时的最大子数组`A[i..j]`所在的位置必然是如下三处：
- 完全处于子数组`A[low..mid]`中，因此 `low ≤ i ≤ j ≤ mid`。
- 完全处于子数组`A[mid+1..high]`中，因此 `mid+1 ≤ i ≤ j ≤ high`。
- 跨越了中点mid，因此 `low≤ i ≤ mid < j ≤ high`。  

接下来我们就可以递归地求解数组`A[low..mid]`和`A[mid+1..high]`的最大子数组，因为这两个子问题仍然是最大子数组问题，只是规模更小，因此剩下的子工作就是在这三个子数组的最大子数组中找出最大的一个。

任何跨中点的最大子数组都由两个子数组`A[i..mid]`和`A[mid+1..j]`组成，其中low≤ i ≤ mid且mid < j ≤ high，因此只要找到形如`A[i..mid]`和`A[mid+1..j]`的最大子数组，然后将其合并即可，对此算导给出了如下的伪代码：  

```
FIND-MAX-CROSSING-SUBARRRY( A, low, mid, high )
    left-sum = -100000
    sum = 0
    for i = mid downto low
        sum = sum + A[i]
        if sum > left-sum
            left-sum = sum
            max-left = i
	
    right-sum = -100000
    sum = 0
    for j = mid + 1 to high
        sum = sum + A[i]
        if sum > right-sum
            right-sum = sum
            max-right = j

    return (max-left, max-right, left-sum + right-sum)
```  

---

有了线性时间的FIND-MAX-CROSSING-SUBARRRY，我们就可以设计求解最大子数组问题的分治算法的伪代码：  

---

```
FIND-MAXIMUM-SUBARRAY(A, low, high)
if high == low
    return ( low, high, A[low] ); 
else 
    mid = ( low + high ) / 2
    ( left-low, left-high, left-sum ) = 
        FIND-MAXIMUM-SUBARRAY( A, low, mid )
    ( right-low, right-high, right-sum ) = 
        FIND-MAXIMUM-SUBARRAY( A, mid + 1, high )
    ( cross-low, cross-high, cross-sum ) = 
        FIND-MAX-CROSSING-SUBARRRY( A, low, mid, high )
    if left-sum >= right-sum and left-sum >= cross-sum
        return ( left-low, left-high, left-sum )
    elseif right-sum >= left-sum and right-sum >= cross-sum
        return ( right-low, right-high, right-sum )
    else
        return ( cross-low, cross-high, cross-sum )
```  

---

具体实现代码如下：  

---

```
//最大子数组问题：分治策略法
#include <stdio.h>
int last_low, last_high;

int
find_max_cro_arr( int A[], int low, int mid, int high )
{
    int i, j;
    int left_sum = -100000;
    int sum = 0;
    for( i = mid; i >= low; i-- ){
        sum += A[ i ]; 
        if( sum > left_sum )
        {
            left_sum = sum;
            last_low = i;
        }
    }

    int right_sum = -100000;
    sum = 0;
    for( j = mid + 1; j <= high; j++ ){
        sum = sum + A[ j ];
        if( sum > right_sum )
        {
            right_sum = sum;
            last_high = j;
        }	
    }
    return left_sum + right_sum;
}

int
find_max_arr( int A[], int low, int high )
{
    int left_sum, right_sum, cross_sum;
    if( high == low )
        return A[ low ];
    else
    {
        int mid = ( low + high ) / 2;
        left_sum = find_max_arr( A, low, mid );
        right_sum = find_max_arr( A, mid + 1, high );
        cross_sum = find_max_cro_arr( A, low, mid, high );
        if ( left_sum >= right_sum && left_sum >= cross_sum )	
            return left_sum;
        else if ( right_sum >= left_sum && right_sum >= cross_sum )	
                return right_sum;
            else
                return cross_sum;	
				
    }
}

int 
main()
{
    int A[] = { 13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7 };
    int length = sizeof( A ) / sizeof( int );
    int sum = find_max_arr( A, 0, length - 1 );
    printf("max_sum:%d, low:%d, high:%d\n", sum, last_low, last_high);

    getchar();
    return 0;
}
```  

参考资料：《算法导论：第三版》。







