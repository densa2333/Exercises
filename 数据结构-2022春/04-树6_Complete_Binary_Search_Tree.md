# 04-树6_Complete_Binary_Search_Tree

## Description

A Binary Search Tree (BST) is recursively defined as a binary tree which has the following properties: 

- The left subtree of a node contains only nodes with keys less than the node's key.
- The right subtree of a node contains only nodes with keys greater than or equal to the node's key.
- Both the left and right subtrees must also be binary search trees.

A Complete Binary Tree (CBT) is a tree that is completely filled, with the possible exception of the bottom level, which is filled from left to right.

Now given a sequence of distinct non-negative integer keys, a unique BST can be constructed if it is required that the tree must also be a CBT. You are supposed to output the level order traversal sequence of this BST.



## Input Specification

Each input file contains one test case. For each case, the first line contains a positive integer *N* (≤1000). Then *N* distinct non-negative integer keys are given in the next line. All the numbers in a line are separated by a space and are no greater than 2000.



## Output Specification

For each test case, print in one line the level order traversal sequence of the corresponding complete binary search tree. All the numbers in a line must be separated by a space, and there must be no extra space at the end of the line.



## Sample Input

```
10
1 2 3 4 5 6 7 8 9 0

```



## Sample Output

```
6 3 8 1 5 7 9 0 2 4
```



## Solution

题意理解：给出一个可以构成二叉搜索树的序列（不规则），需要你输出其构造成完全二叉搜索树的层序遍历结果。

**分析**

- 数据结构的选取：由于是完全二叉树，用数组存储比较方便和节省空间
- 如何构造完全二叉搜索树：通过数学计算的方法计算出左右子树的元素个数进行递归处理
- 核心算法：构建完全二叉搜索树数组的过程是一个对该树先序遍历的过程
  - 在第一次调用`Solve()`前先将序列由小到大排序存到`A[]`中，以便先序遍历
  - 找到根节点，存入`T[TRoot]`
  - 对其左右子树分别递归调用`Solve()`
- 核心算法中需要用到一个求左子树元素个数的函数`GetLeftLength()`需要实现

详细代码：

```C
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int compare(const void *a, const void *b);
void Solve(int, int, int); //core algorithm
int GetLeftLength(int);

/* T[] is CBT  A[] is BST */
int T[1000], A[1000];

int main(void)
{
    int i, n;
    scanf("%d", &n);
    
    /* input BST A[] */
    for (i = 0; i < n; i++) {
        scanf("%d", &A[i]);
    }
    
    /* sort the sequence of BST */
    qsort(A, n, sizeof(int), compare);
    
    /* construct CBT by T[] */
    Solve(0, n - 1, 0);
    
    /* output CBT T[] */
    for (i = 0; i < n; i++) {
        if (i) printf(" %d", T[i]);
        else printf("%d", T[i]);
    }
    return 0;
}

/* parameter of qsort() */
int compare(const void *a, const void *b)
{ return *(int *)a - *(int *)b; }

/* recursively construct the CBT */
void Solve(int ALeft, int ARight, int TRoot)
{
    int n, L, LeftTRoot, RightTRoot;
    n = ARight - ALeft + 1; //element number of BST now
    
    /* recursion end condition */
    if (n == 0) return;
    
    /* get the element number of left subtree */
    L = GetLeftLength(n);
    
    /* construct CBT T[] */
    T[TRoot] = A[ALeft + L];
    LeftTRoot = TRoot * 2 + 1;
    RightTRoot = LeftTRoot + 1;
    
    /* construct left subtree by recursion */
    Solve(ALeft, ALeft + L - 1, LeftTRoot);
    
    /* construct right subtree by recursion */
    Solve(ALeft + L + 1, ARight, RightTRoot);
}

/* get the element number of left subtree */
int GetLeftLength(int n)
{
    int h, x;
    
    /* find the height of the tree except the bottom line */
    h = (int)(log(n + 1) / log(2));
    
    /* find the element number of the bottom line in left tree */
    x = n - (int)pow(2, h) + 1 < (int)pow(2, h - 1) ? n - (int)pow(2, h) + 1 : (int)pow(2, h - 1);
    
    /* formula: L = 2^(h - 1) - 1 + x */
    return (int)pow(2, h - 1) - 1 + x;
}
```

