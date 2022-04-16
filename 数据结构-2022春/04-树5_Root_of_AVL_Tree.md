# 04-树5_Root_of_AVL_Tree

## Description

An AVL tree is a self-balancing binary search tree. In an AVL tree, the heights of the two child subtrees of any node differ by at most one; if at any time they differ by more than one, rebalancing is done to restore this property. Figures 1-4 illustrate the rotation rules.

![Figure1](https://github.com/densa2333/Exercises/blob/main/assets/04F1.jpg)

![Figure2](https://github.com/densa2333/Exercises/blob/main/assets/04F2.jpg)

![Figure3](https://github.com/densa2333/Exercises/blob/main/assets/04F3.jpg)

![Figure4](https://github.com/densa2333/Exercises/blob/main/assets/04F4.jpg)

Now given a sequence of insertions, you are supposed to tell the root of the resulting AVL tree.



## Input Specification

Each input file contains one test case. For each case, the first line contains a positive integer *N* (≤20) which is the total number of keys to be inserted. Then *N* distinct integer keys are given in the next line. All the numbers in a line are separated by a space.



## Output Specification

For each test case, print the root of the resulting AVL tree in one line.



## Sample Input 1:

```
5
88 70 61 96 120

```



## Sample Output 1:

```
78
```



## Sample Input 2:

```
7
88 70 61 96 120 90 65

```



## Sample Output 2:

```
88
```



## Solution

平衡二叉搜索树的操作集练习

```C
#include <stdio.h>
#include <stdlib.h>
#define ElementType int

typedef struct AVLNode *Position;
typedef Position AVLTree;
struct AVLNode {
    ElementType Data;
    AVLTree Left, Right;
    int Height;
};

int Max(int a, int b) { return a > b ? a : b; }

AVLTree SingleLeftRotation(AVLTree);
AVLTree DoubleLeftRightRotation(AVLTree);
AVLTree SingleRightRotation(AVLTree);
AVLTree DoubleRightLeftRotation(AVLTree);
int GetHeight(AVLTree);
AVLTree Insert(AVLTree, ElementType);

int main()
{
    int i, n, item;
    AVLTree T = NULL;
    scanf("%d", &n);
    while (n--) {
        scanf("%d", &item);
        T = Insert(T, item);
    }
    printf("%d", T->Data);
    return 0;
}

AVLTree SingleLeftRotation(AVLTree A)
{
    AVLTree B = A->Left;
    A->Left = B->Right;
    B->Right = A;
    
    A->Height = Max(GetHeight(A->Left), GetHeight(A->Right)) + 1;
    B->Height = Max(GetHeight(B->Left), A->Height) + 1;
    
    return B;
}

AVLTree DoubleLeftRightRotation(AVLTree A)
{
    A->Left = SingleRightRotation(A->Left);
    return SingleLeftRotation(A);
}

AVLTree SingleRightRotation(AVLTree A)
{
    AVLTree B = A->Right;
    A->Right = B->Left;
    B->Left = A;
    
    A->Height = Max(GetHeight(A->Left), GetHeight(A->Right)) + 1;
    B->Height = Max(GetHeight(B->Right), A->Height) + 1;
    
    return B;
}

AVLTree DoubleRightLeftRotation(AVLTree A)
{
    A->Right = SingleLeftRotation(A->Right);
    return SingleRightRotation(A);
}

int GetHeight(AVLTree T)
{
    if (!T) return 0;
    else return T->Height;
}

AVLTree Insert(AVLTree T, ElementType item)
{
    if (!T) {
        T = (AVLTree)malloc(sizeof(struct AVLNode));
        T->Data = item;
        T->Left = NULL;
        T->Right = NULL;
        T->Height = 0;
    }
    
    else if (T->Data < item) {
        T->Right = Insert(T->Right, item);
        if (GetHeight(T->Left) - GetHeight(T->Right) == -2) {
            if (item > T->Right->Data)
                T = SingleRightRotation(T);
            else
                T = DoubleRightLeftRotation(T);
        }
    }
    
    else if (T->Data > item) {
        T->Left = Insert(T->Left, item);
        if (GetHeight(T->Right) - GetHeight(T->Left) == -2) {
            if (item < T->Left->Data)
                T = SingleLeftRotation(T);
            else
                T = DoubleLeftRightRotation(T);
        }
    }
    
    T->Height = Max(GetHeight(T->Left), GetHeight(T->Right)) + 1;
    
    return T;
}
```

