# 03-树3_Tree_Traversals_Again

## Description

An inorder binary tree traversal can be implemented in a non-recursive way with a stack. For example, suppose that when a 6-node binary tree (with the keys numbered from 1 to 6) is traversed, the stack operations are: push(1); push(2); push(3); pop(); pop(); push(4); pop(); pop(); push(5); push(6); pop(); pop(). Then a unique binary tree (shown in Figure 1) can be generated from this sequence of operations. Your task is to give the postorder traversal sequence of this tree.

[Figure 1](https://github.com/densa2333/Exercises/blob/main/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84-2022%E6%98%A5/03-%E6%A0%913TreeTraversalsAgainF1.png)



## Input Specification

Each input file contains one test case. For each case, the first line contains a positive integer *N* (≤30) which is the total number of nodes in a tree (and hence the nodes are numbered from 1 to *N*). Then 2*N* lines follow, each describes a stack operation in the format: "Push X" where X is the index of the node being pushed onto the stack; or "Pop" meaning to pop one node from the stack.



## Output Specification

For each test case, print the postorder traversal sequence of the corresponding tree in one line. A solution is guaranteed to exist. All the numbers must be separated by exactly one space, and there must be no extra space at the end of the line.



## Sample Input

```
6
Push 1
Push 2
Push 3
Pop
Pop
Push 4
Pop
Pop
Push 5
Push 6
Pop
Pop

```



## Sample Output

```
3 4 2 6 5 1
```



## Solution

题意理解：

给出的是中序遍历的出入栈顺序，由二叉树遍历的知识知所有入栈的顺序即为先序遍历的顺序，已知先序遍历和中序遍历，我们就可以构造出该树。

这里我们不需要真的构造一棵树，而是利用核心算法来递归构造后序遍历的顺序数组。

详见注释。

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct LNode * List;
struct LNode {
    int Data;
    List Next;
};

typedef struct LNode * Stack;	// stack of linked list implementation

void Push(int, Stack *);
int Pop(Stack *);
void Solve(int, int, int, int);	// core algorithm

int pre[30], in[30], post[30];

int main()
{
    int i, n, num, idxpre, idxin;
    char s[10];
    Stack Pre = NULL;
    
    /* remember getchar(); */
    scanf("%d", &n);
    getchar();
    
    /* input */
    idxpre = 0;
    idxin = 0;
    for (i = 0; i < n * 2; i++) {
        gets(s);
        if (strncmp(s, "Push", 4) == 0) {
            if (s[6]) num = 10 * (s[5] - '0') + s[6] - '0'; // tow bits
            else num = s[5] - '0';							// one bits
            Push(num, &Pre);
            pre[idxpre] = num;	//preoder traversal
            idxpre++;
        }
        else if (strncmp(s, "Pop", 3) == 0) {
            in[idxin] = Pop(&Pre);	// inorder traversal
            idxin++;
        }
    }
    
    /* void Solve(int preL, int inL, int postL, int n); */
    Solve(0, 0, 0, n);
    
    /* output */
    for (i = 0; i < n; i++) {
        if (i) printf(" %d", post[i]);
        else printf("%d", post[i]);
    }
    
    return 0;
}

void Push(int x, Stack *S)	// push of linked list implementation
{
    if (*S == NULL) {
        (*S) = malloc(sizeof(struct LNode));
        (*S)->Data = x;
        (*S)->Next = NULL;
    } else {
        Stack s = malloc(sizeof(struct LNode));
        s->Data = x;
        s->Next = *S;
        *S = s;
    }
}

int Pop(Stack *S)	// pop of linked list implementation
{
    int ret = -1;
    List p;
    if (*S == NULL) printf("Stack is empty");
    else {
        ret = (*S)->Data;
        p = *S;
        *S = (*S)->Next;
        free(p);
    }
    return ret;
}

/* construct postorder by recursion implementation */
void Solve(int preL, int inL, int postL, int n)
{
    int i, root, L, R;
    
    /* recursion end condition */
    if (n == 0) return;
    if (n == 1) {
        post[postL] = pre[preL];
        return;
    }
    
    /* recursion operation */
    root = pre[preL];
    post[postL + n - 1] = root;	// preFirst -> root -> postLast
    
    for (i = 0; i < n; i++) {
        if (in[inL + i] == root) break;	// find root in inorder traversal
    }
    
    L = i;			// elements number of left subtree
    R = n - L - 1;	// elements number of right subtree
    
    /* construct left subtree by recursion */
    Solve(preL + 1, inL, postL, L);
    
    /* construct right subtree by recur */
    Solve(preL + L + 1, inL + L + 1, postL + L, R);
}
```

