# 09-排序3_Insertion_or_Heap_Sort

## Description

According to Wikipedia:

**Insertion sort** iterates, consuming one input element each repetition, and growing a sorted output list. Each iteration, insertion sort removes one element from the input data, finds the location it belongs within the sorted list, and inserts it there. It repeats until no input elements remain.

**Heap sort** divides its input into a sorted and an unsorted region, and it iteratively shrinks the unsorted region by extracting the largest element and moving that to the sorted region. it involves the use of a heap data structure rather than a linear-time search to find the maximum.

Now given the initial sequence of integers, together with a sequence which is a result of several iterations of some sorting method, can you tell which sorting method we are using?



## Input Specification

Each input file contains one test case. For each case, the first line gives a positive integer *N* (≤100). Then in the next line, *N* integers are given as the initial sequence. The last line contains the partially sorted sequence of the *N* numbers. It is assumed that the target sequence is always ascending. All the numbers in a line are separated by a space.



## Output Specification

For each test case, print in the first line either "Insertion Sort" or "Heap Sort" to indicate the method used to obtain the partial result. Then run this method for one more iteration and output in the second line the resulting sequence. It is guaranteed that the answer is unique for each test case. All the numbers in a line must be separated by a space, and there must be no extra space at the end of the line.



## Sample Input 1

```
10
3 1 2 8 7 5 9 4 6 0
1 2 3 7 8 5 9 4 6 0

```



## Sample Output 1

```
Insertion Sort
1 2 3 5 7 8 9 4 6 0
```



## Sample Input 2

```
10
3 1 2 8 7 5 9 4 6 0
6 4 5 1 0 3 2 7 8 9

```



## Sample Output 2

```
Heap Sort
5 4 3 1 0 2 6 7 8 9
```



## Solution

题意理解：

- 给出两行，第一行是待排序数组，第二行是排序到某一步的数组
- 需要你判断第二行数组的顺序是属于哪一种排序，并且输出再按这种排序方式迭代一次后的数组顺序

分析：

- 在每一次迭代末尾增加判断函数，如果符合就选定这种排序方式，再迭代一次后输出顺序

细节：

- 两种排序方式的判断，第一次排序不能直接在待排序数组上操作，不然第二次判断没有原数组可操作
- 归并排序算法里每次互相倒腾都是一次迭代，需要分别判断是否符合所给顺序

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#define MAX 100

bool Insertion_Sort(int [], int [], int);
void Swap(int *, int *);
void PercDown(int [], int, int);
void Heap_Sort(int [], int[], int);
bool IsSame(int [], int [], int);

int main()
{
    int i, n, initial[MAX], target[MAX];
    scanf("%d", &n);
    for (i = 0; i < n; i++) scanf("%d", &initial[i]);
    for (i = 0; i < n; i++) scanf("%d", &target[i]);
    
    if (Insertion_Sort(initial, target, n));
    else Heap_Sort(initial, target, n);

    return 0;
}

bool Insertion_Sort(int A[], int B[], int N)
{
    int P, i, Tmp, tmp[MAX];
    bool flag;
    for (i = 0; i < N; i++) tmp[i] = A[i];
     
    for (P = 1; P < N; P++) {
        Tmp = tmp[P];
        for (i = P; i > 0 && tmp[i - 1] > Tmp; i--)
            tmp[i] = tmp[i - 1]; 
        tmp[i] = Tmp;
        flag = IsSame(tmp, B, N);
        if (flag) {
            P++;
            Tmp = tmp[P];
            for (i = P; i > 0 && tmp[i - 1] > Tmp; i--)
                tmp[i] = tmp[i - 1];
            tmp[i] = Tmp;
            break;
        }
    }
    if (flag) {
        printf("Insertion Sort\n");
        for (i = 0; i < N; i++)
            if (i) printf(" %d", tmp[i]);
            else printf("%d", tmp[i]);
    }

    return flag;
}

void Swap( int *a, int *b )
{
    int t = *a; *a = *b; *b = t;
}
 
void PercDown( int A[], int p, int N )
{ /* 改编代码4.24的PercDown( MaxHeap H, int p )    */
  /* 将N个元素的数组中以A[p]为根的子堆调整为最大堆 */
    int Parent, Child;
    int X;

    X = A[p]; /* 取出根结点存放的值 */
    for( Parent = p; (Parent*2+1) < N; Parent = Child ) {
        Child = Parent * 2 + 1;
        if( (Child != N-1) && (A[Child] < A[Child+1]) )
            Child++;  /* Child指向左右子结点的较大者 */
        if( X >= A[Child] ) break; /* 找到了合适位置 */
        else  /* 下滤X */
            A[Parent] = A[Child];
    }
    A[Parent] = X;
}

void Heap_Sort( int A[], int B[], int N )
{ /* 堆排序 */
    int i;
      
    for ( i=N/2-1; i>=0; i-- )/* 建立最大堆 */
        PercDown( A, i, N );
     
    for ( i=N-1; i>0; i-- ) {
        /* 删除最大堆顶 */
        Swap( &A[0], &A[i] );
        PercDown( A, 0, i );
        if (IsSame(A, B, N)) {
            printf("Heap Sort\n");
            i--;
            Swap( &A[0], &A[i] );
            PercDown( A, 0, i );
            for (i = 0; i < N; i++)
                if (i) printf(" %d", A[i]);
                else printf("%d", A[i]);
            break;
        }
    }
}

bool IsSame(int A[], int B[], int n)
{
    int i;

    bool flag = true;
    for (i = 0; i < n && flag; i++)
        if (A[i] != B[i]) flag = false;

    return flag;
}
```

