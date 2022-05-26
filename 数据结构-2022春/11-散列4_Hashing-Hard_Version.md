# 11-散列4 Hashing - Hard Version

## Description

Given a hash table of size *N*, we can define a hash function *H*(*x*)=*x*%*N*. Suppose that the linear probing is used to solve collisions, we can easily obtain the status of the hash table with a given sequence of input numbers.

However, now you are asked to solve the reversed problem: reconstruct the input sequence from the given status of the hash table. Whenever there are multiple choices, the smallest number is always taken.



## Input Specification

Each input file contains one test case. For each test case, the first line contains a positive integer *N* (≤1000), which is the size of the hash table. The next line contains *N* integers, separated by a space. A negative integer represents an empty cell in the hash table. It is guaranteed that all the non-negative integers are distinct in the table.



## Output Specification

For each test case, print a line that contains the input sequence, with the numbers separated by a space. Notice that there must be no extra space at the end of each line.



## Sample Input

```
11
33 1 13 12 34 38 27 22 32 -1 21
```



## Sample Output

```
1 13 12 21 33 34 38 27 22 32
```



## Solution

**拓扑排序**

- 首先是散列表的输入方式，我们必须清楚。按照 H = X % N 的 公式存入，并且已知散列表的大小。
- 那么当元素 x 被映射到 H(x) 位置，发现这个位置已经有y了，则y一定是在x之前被输入的。并且在 y 的前面的所有的数都一定比 y 要先输入，所以 y 前面的数的个数间接的可以表示为 y 的入度。
- 在设立结构体的时候，可以如此表示
  ```
  struct LinkNode {
      int data;
      struct LinkNode *Next;
  }; 
  struct listNode{
      int count;
      struct LinkNode *Next;
  };
  ```
- 用途是将散列表的每一个位置的数都用一个指针结构体数组表示，然后链表的头为空，后面接该位置元素的前面的所有的数。
- 每次将度为零并且最小的数输出并且令该位置的数的入度为-1，表示不再考虑该数
- 然后遍历入度不为零的数，将其中存在被删除的min的数组中，入度减一，因为少了个数

算法图解：

| `num` | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|       | 33   | 1    | 13   | 12   | 34   | 38   | 27   | 22   | 32   | -1   | 21   |

| `Toplist` | 0      | 1      | 2      | 3      | 4      | 5      | 6      | 7      | 8      | 9      | 10     |
| --------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| Degree    | 0      | 0      | 0      | 2      | 3      | 0      | 1      | 7      | 9      | 0      | 0      |
|           | `NULL` | `NULL` | `NULL` | 13     | 12     | `NULL` | 38     | 27     | 22     | `NULL` | `NULL` |
|           |        |        |        | 1      | 13     |        | `NULL` | 38     | 27     |        |        |
|           |        |        |        | `NULL` | 1      |        |        | 34     | 38     |        |        |
|           |        |        |        |        | `NULL` |        |        | 12     | 34     |        |        |
|           |        |        |        |        |        |        |        | 13     | 12     |        |        |
|           |        |        |        |        |        |        |        | 1      | 13     |        |        |
|           |        |        |        |        |        |        |        | 33     | 1      |        |        |
|           |        |        |        |        |        |        |        | `NULL` | 33     |        |        |
|           |        |        |        |        |        |        |        |        | 21     |        |        |
|           |        |        |        |        |        |        |        |        | `NULL` |        |        |

源码：

```C
#include <stdio.h>
#include <stdlib.h>
#define Maxsize 10001

struct LNode {
    int Data;
    struct LNode *Next;
};
typedef struct LNode *Link; //散列表元素

struct lNode {
    int Degree;
    struct lNode *Next;
};
typedef struct lNode *list; //邻接表，头结点为空

int main()
{
    int i, j, N, num[Maxsize], min = Maxsize, minp = 0;

    scanf("%d", &N);
    list Toplist = (list)malloc(sizeof(struct lNode) * N);
    for (i = 0; i < N; i++) { //初始化散列表头
        scanf("%d", &num[i]);
        Toplist[i].Degree = 0;
        Toplist[i].Next = NULL;
    }
    for (i = 0; i < N; i++) {
        if (num[i] >= 0 && num[i] % N != i) {
            for (j = i; j != num[i] % N; ) {
                j = (j + N - 1) % N; //寻找当前元素前面的冲突元素
                Link Node = (Link)malloc(sizeof(struct LNode));
                Node->Data = num[j];
                //插入当前元素前面的所有冲突元素
                Node->Next = Toplist[i].Next;
                Toplist[i].Next = Node;
                //计算入度
                Toplist[i].Degree++;
            }
        }
    }
    int cnt = N;
    while (cnt) {
        min = 100000;
        minp = 0;
        for (i = 0; i < N; i++) {
            //寻找入度为零且最小的数
            if (Toplist[i].Degree == 0 && num[i] < min) {
                min = num[i];
                minp = i;
            }
        }
        Toplist[minp].Degree = -1; //下次不再考虑该位置的数
        cnt--;
        if (min >= 0) {
            printf("%d", min);
            if (cnt != 0) printf(" "); //行尾空格特判
        }
        for (i = 0; i < N; i++) {
            if (Toplist[i].Degree > 0) { //对所有剩下的数
                Link Node = Toplist[i].Next;
                while (Node != NULL) {
                    if (Node->Data == min) {
                        Toplist[i].Degree--; //如果表中有前面删除的元素就调整入度
                        break;
                    }
                    Node = Node->Next;
                }
            }
        }
    }
    return 0;
}
```

