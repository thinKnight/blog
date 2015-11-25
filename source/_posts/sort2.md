title: 排序算法（二）
date: 2014-03-04 22:20:39
tags: Algorithm
---
上一篇介绍了几种简单的排序算法，这一篇来谈谈**归并排序**和**快速排序**。  
<!-- more -->

# 归并排序（Merge Sort）
顾名思义，归并算法就是将两个或两个以上的有序表组合成一个新的有序表，所以归并排序算法是递归算法的一个很好的实例。

## 算法原理
归并操作的过程如下：
1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
4. 重复步骤3直到某一指针达到序列尾
5. 将另一序列剩下的所有元素直接复制到合并序列尾  

举个例子吧:  

>`[ 10 4 6 3 8 2 5 7 ]`   
>`[ 10 4 6 3 ]   [ 8 2 5 7 ]`  
>`[ 10 4 ]  [ 6 3 ]  [ 8 2 ]  [ 5 7 ]`  
>~~`[ 10 ] [ 4 ] [ 6 ] [ 3 ] [ 8 ] [ 2 ] [ 5 ] [ 7 ]`~~  
>`[ 4 10 ]  [ 3 6  ] [ 2 8 ]  [ 5 7 ]`  
>`[ 3 4 6 10 ]  [ 2 5 7 8 ]`  
>`[ 2 3 4 5 6 7 8 10 ]`  

## 算法分析
- 最差时间复杂度：O(nlogn)  

- 最优时间复杂度：O(n)  

- 平均时间复杂度：O(nlogn)  

## 算法稳定性
归并排序是一种稳定的排序方法  

## 算法实现
```C
//归并排序
typedef int ElemType;

void Merge( ElemType A[], ElemType TmpArray[], int Lpos, int Rpos, int RightEnd );
void MSort( ElemType A[], ElemType TmpArray[], int Left, int Right );
void Mergesort( ElemType A[], int N );

void Merge( ElemType A[], ElemType TmpArray[], int Lpos, int Rpos, int RightEnd )
{
    int i, LeftEnd, NumElements, TmpPos;
    LeftEnd = Rpos - 1;
    TmpPos = Lpos;
    NumElements = RightEnd - Lpos + 1;

    while( Lpos <= LeftEnd && Rpos <= RightEnd )
        if( A[ Lpos ] <= A[ Rpos ] )
            TmpArray[ TmpPos++ ] = A[ Lpos++ ];
        else
            TmpArray[ TmpPos++ ] = A[ Rpos++ ];

    while( Lpos <= LeftEnd )
        TmpArray[ TmpPos++ ] = A[ Lpos++ ];
    while( Rpos <= RightEnd )
        TmpArray[ TmpPos++ ] = A[ Rpos++ ];

    for( i = 0; i < NumElements; i++, RightEnd-- )
        A[ RightEnd ] = TmpArray[ RightEnd ];
}

void MSort( ElemType A[], ElemType TmpArray[], int Left, int Right )
{
    int Center;

    if( Left < Right )
    {
        Center = ( Left + Right ) / 2;
        MSort( A, TmpArray, Left, Center );
        MSort( A, TmpArray, Center + 1, Right );
        Merge( A, TmpArray, Left, Center + 1, Right );
    }
}

void Mergesort( ElemType A[], int N )
{
    ElemType *TmpArray;

    TmpArray = malloc( N * sizeof( ElemType ));
    if( TmpArray != NULL)
    {
        MSort( A, TmpArray, 0, N - 1 );
        free( TmpArray );
    }
    else
        printf("No space for tmp array!!!");
}
```  


虽然归并排序的运行时间是O（nlogn），但是它很难用于主存排序，主要问题在于合并两个排序的表需要线性附加内存，在整个算法中还要花费将数据拷贝到临时数组再拷贝回来这样一些附加的工作，其结果严重放慢了排序的速度。

归并排序不仅可以用分治的策略来递归处理，也可以用迭代的方法来表示，但即使这样，对于重要的内部排序而言，人们还是选择快速排序。  

# 快速排序（Quick Sort）
正如它的名字所标示的，快速排序是在实践中已知的最快排序算法，该算法特别快，主要是由于非常精炼和高度优化的内部循环。像归并排序一样，快速排序也是一种分治的递归算法。  


## 算法原理
将数组S排序的基本算法由下列简单的4步组成：
 1. 如果S中元素个数是0或1，则返回。
 2. 取S中任一元素，称之为枢纽元。
 3. 将A = S - {v}(S中其余元素)分成两个不相交的集合：S1 = { x属于A | x <= v } 和S2= { x属于A | x >= v }。
 4. 返回{quicksort(S1)后，继随v,继而quicksort(S2)}.
 

下面给出一个乱序的数组`{[ 50 10 90 30 70 40 80 60 20 }`，以第一个元素`50`为枢纽元，来展现快速排序的过程。

>`[ (50) 10 90 30 70 40 80 60 (20) ]` 第1次交换  

>`[ (20) 10 90 30 70 40 80 60 (50) ]` 第2次交换  

>`[ 20 10 (50) 30 70 40 80 60 (90) ]` 第3次交换  

>`[ 20 10 (40) 30 70 (50) 80 60 90 ]` 第4次交换  

>`[ 20 10 40 30 (50) (70) 80 60 90 ]` 第5次交换  

>`[ 20 10 40 30 （50）70 80 60 90 ]`  第6次交换  

 
## 算法分析
- 最差时间复杂度：O(n²)
- 最优时间复杂度：O(nlogn)
- 平均时间复杂度：O(nlogn)  

## 算法稳定性
快速排序是一种稳定的排序方法  

## 算法实现
```C
typedef int ElemType;

void swap(int A[], int a, int b );
void Quicksort( ElemType A[], int N );
void QSort(int A[], int low, int high);
int Partition(int A[], int low, int high );

void Quicksort( ElemType A[], int N )
{
    QSort( A, 0, N - 1 );
}

void swap(int A[], int a, int b )
{
    int Tmp;
    Tmp = A[ a ];
    A[ a ] = A[ b ];
    A[b] = Tmp;
}

void QSort(int A[], int low, int high)
{
    int pivot;

    if( low < high )
    {
        pivot = Partition( A, low, high );
        QSort( A, low, pivot-1 );
        QSort( A, pivot+1, high );
    }
}

int Partition(int A[], int low, int high )
{
    int pivotkey;
    pivotkey = A[ low ];
    while( low < high )
    {
        while( low < high && A[ high ] >= pivotkey )
            high--;
        swap(A, low, high);
        while( low < high && A[ low ] <= pivotkey)
            low++;
        swap(A, low, high);
    }
    return low;
}

```  

---

排序算法的内容就差不多先写到这里了，其实学习算法，光能用代码写出算法和理解原理只是初步的学习，最近看了《算法导论》才认识到，一个算法不光光只像这两篇文章里写的内容那么简单，还包含各自复杂的数学证明和进一步的优化，我的学习还有待继续深入！  


# 舞动的排序算法
如果光看文字还理解不了算法原理的话，这里有一组用舞蹈的形式来展现排序算法的原理的视频。  

 - [冒泡排序](http://v.youku.com/v_show/id_XMzMyOTAyMzQ0.html?f=16755664)
 - [归并排序](http://v.youku.com/v_show/id_XMzMyODk5Njg4.html?f=16755664)
 - [希尔排序](http://v.youku.com/v_show/id_XMzMyODk5MzI4.html?f=16755664)
 - [选择排序](http://v.youku.com/v_show/id_XMzMyODk5MDI0.html?f=16755664)
 - [快速排序](http://v.youku.com/v_show/id_XMzMyODk4NTQ4.html?f=16755664)
 - [插入排序](http://v.youku.com/v_show/id_XMzMyODk3NjI4.html?f=16755664)
 
 
 
 

