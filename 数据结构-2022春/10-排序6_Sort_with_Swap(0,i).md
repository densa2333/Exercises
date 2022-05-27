# 10-排序6_Sort_with_Swap(0,i)

## Description

Given any permutation of the numbers {0, 1, 2,..., *N*−1}, it is easy to sort them in increasing order. But what if `Swap(0, *)` is the ONLY operation that is allowed to use? For example, to sort {4, 0, 2, 1, 3} we may apply the swap operations in the following way:

```
Swap(0, 1) => {4, 1, 2, 0, 3}
Swap(0, 3) => {4, 1, 2, 3, 0}
Swap(0, 4) => {0, 1, 2, 3, 4}
```

Now you are asked to find the minimum number of swaps need to sort the given permutation of the first *N* nonnegative integers.



## Input Specification

Each input file contains one test case, which gives a positive *N* (≤105) followed by a permutation sequence of {0, 1, ..., *N*−1}. All the numbers in a line are separated by a space.



## Output Specification

For each case, simply print in a line the minimum number of swaps need to sort the given permutation.



## Sample Input

```
10
3 5 7 2 6 4 9 0 8 1

```



## Sample Output

```
9
```



## Solution

N 个数字的排列由若干个独立的环组成

此题里环分三种：

- 只有一个元素，不需要交换  
- 环里 n~0~ 个元素，包括 0 ：需要 n~0~ - 1 次交换  
- 第 i 个环里有 n~i~ 个元素，不包括 0 ：先把 0 换到环里，再进行 (n~i~ + 1) - 1 次交换——一共是 n~i~ + 1 次交换

### 第二次尝试（AC）网上答案

```c
#include<stdio.h>
#include<stdlib.h>
int main()
{
    int i,n,tmp,num,*arr;
    scanf("%d",&n);
    arr = (int *)malloc(n*sizeof(int));
    
    for(i=0;i<n;i++) 
        scanf("%d",&arr[i]);
    num = 0;
    i = 0;
    if(arr[0] == 0) i=1;
    for(;i<n;++i) {
        if(arr[i] != i) {
            ++ num;
            while(arr[i] != i) {
                if(arr[i] == 0) // 环中含 0
                    --num;
                else            // 环中不含 0
                    ++num;
                tmp = arr[i];
                arr[i] = i;
                i = tmp;
            }
        }
    }
    printf("%d",num);
    free(arr);
    return 0;
}
```

### 第一次尝试（TLE）

先表排序，再模拟交换次数

```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int i, j, n, tmp, p, cnt, sum, *a, *t;
    scanf("%d", &n);
    a = (int*)malloc(sizeof(int) * n);
    t = (int*)malloc(sizeof(int) * n);
    for (i = 0; i < n; i++) scanf("%d", &a[i]);
    for (i = 0; i < n; i++) t[i] = i;
    // 插入排序
    for (i = 1; i < n; i++) {
        tmp = t[i];
        for (j = i; j > 0 && a[t[j - 1]] > a[tmp]; j--)
            t[j] = t[j - 1];
        t[j] = tmp;
    }
    // 计算交换次数
    sum = 0;
    for (i = 0; i < n; i++) {
        cnt = 0;
        if (t[i] != i) {
            // 是否含零
            if (a[t[i]] == 0) cnt--;
            else cnt++;
            //进入该环
            j = i;
            tmp = a[j];
            while (t[j] != j) {
                a[j] = a[t[j]];
                p = j;
                j = t[j];
                t[p] = p;
                cnt++;
            }
            a[p] = tmp;
        }
        sum += cnt;
    }
    printf("%d\n", sum);
    return 0;
}
```

