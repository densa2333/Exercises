# 09-排序2_Insert_or_Merge

## Description

According to Wikipedia:

**Insertion sort** iterates, consuming one input element each repetition, and growing a sorted output list. Each iteration, insertion sort removes one element from the input data, finds the location it belongs within the sorted list, and inserts it there. It repeats until no input elements remain.

**Merge sort** works as follows: Divide the unsorted list into N sublists, each containing 1 element (a list of 1 element is considered sorted). Then repeatedly merge two adjacent sublists to produce new sorted sublists until there is only 1 sublist remaining.

Now given the initial sequence of integers, together with a sequence which is a result of several iterations of some sorting method, can you tell which sorting method we are using?



## Input Specification

Each input file contains one test case. For each case, the first line gives a positive integer *N* (≤100). Then in the next line, *N* integers are given as the initial sequence. The last line contains the partially sorted sequence of the *N* numbers. It is assumed that the target sequence is always ascending. All the numbers in a line are separated by a space.



## Output Specification

For each test case, print in the first line either "Insertion Sort" or "Merge Sort" to indicate the method used to obtain the partial result. Then run this method for one more iteration and output in the second line the resuling sequence. It is guaranteed that the answer is unique for each test case. All the numbers in a line must be separated by a space, and there must be no extra space at the end of the line.



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
3 1 2 8 7 5 9 4 0 6
1 3 2 8 5 7 4 9 0 6

```



## Sample Output

```
Merge Sort
1 2 3 8 4 5 7 9 0 6
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
void Merge_Sort(int [], int[], int);
bool IsSame(int [], int [], int);

int main()
{
    int i, n, initial[MAX], target[MAX];
    scanf("%d", &n);
    for (i = 0; i < n; i++) scanf("%d", &initial[i]);
    for (i = 0; i < n; i++) scanf("%d", &target[i]);
    
    if (Insertion_Sort(initial, target, n));
    else Merge_Sort(initial, target, n);

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

void Merge(int A[], int TmpA[], int L, int R, int RightEnd)
{ /* 将有序的A[L]~A[R-1]和A[R]~A[RightEnd]归并成一个有序序列 */
    int LeftEnd, NumElements, Tmp;
    int i;

    LeftEnd = R - 1; /* 左边终点位置 */
    Tmp = L;         /* 有序序列的起始位置 */
    NumElements = RightEnd - L + 1;
     
    while( L <= LeftEnd && R <= RightEnd ) {
        if ( A[L] <= A[R] )
            TmpA[Tmp++] = A[L++]; /* 将左边元素复制到TmpA */
        else
            TmpA[Tmp++] = A[R++]; /* 将右边元素复制到TmpA */
    }

    while( L <= LeftEnd )
        TmpA[Tmp++] = A[L++]; /* 直接复制左边剩下的 */
    while( R <= RightEnd )
        TmpA[Tmp++] = A[R++]; /* 直接复制右边剩下的 */

}

void Merge_pass(int A[], int TmpA[], int N, int length)
{ /* 两两归并相邻有序子列 */
    int i, j;
      
    for ( i=0; i <= N-2*length; i += 2*length )
        Merge( A, TmpA, i, i+length, i+2*length-1 );
    if ( i+length < N ) /* 归并最后2个子列*/
        Merge( A, TmpA, i, i+length, N-1);
    else /* 最后只剩1个子列*/
        for ( j = i; j < N; j++ ) TmpA[j] = A[j];
}

void Merge_Sort(int A[], int B[], int N)
{ 
    int length, i; 
    int *TmpA;
     
    length = 1; /* 初始化子序列长度*/
    TmpA = malloc(N * sizeof(int));
    if ( TmpA != NULL ) {
        while( length < N ) {
            Merge_pass(A, TmpA, N, length);
            length *= 2;
            if (IsSame(TmpA, B, N)) {
                Merge_pass(TmpA, A, N, length);
                length *= 2;
                printf("Merge Sort\n");
                for (i = 0; i < N; i++)
                    if (i) printf(" %d", A[i]);
                    else printf("%d", A[i]);
                break;
            }

            Merge_pass(TmpA, A, N, length);
            length *= 2;
            if (IsSame(A, B, N)) {
                Merge_pass(A, TmpA, N, length);
                length *= 2;
                printf("Merge Sort\n");
                for (i = 0; i < N; i++)
                    if (i) printf(" %d", TmpA[i]);
                    else printf("%d", TmpA[i]);
                break;
            }
        }
        free( TmpA );
    }
    else printf("空间不足");
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

