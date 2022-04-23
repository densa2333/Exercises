# 07-图5_Saving_James_Bond-Hard_Version

## Description

This time let us consider the situation in the movie "Live and Let Die" in which James Bond, the world's most famous spy, was captured by a group of drug dealers. He was sent to a small piece of land at the center of a lake filled with crocodiles. There he performed the most daring action to escape -- he jumped onto the head of the nearest crocodile! Before the animal realized what was happening, James jumped again onto the next big head... Finally he reached the bank before the last crocodile could bite him (actually the stunt man was caught by the big mouth and barely escaped with his extra thick boot).

Assume that the lake is a 100 by 100 square one. Assume that the center of the lake is at (0,0) and the northeast corner at (50,50). The central island is a disk centered at (0,0) with the diameter of 15. A number of crocodiles are in the lake at various positions. Given the coordinates of each crocodile and the distance that James could jump, you must tell him a shortest path to reach one of the banks. The length of a path is the number of jumps that James has to make.



## Input Specification

Each input file contains one test case. Each case starts with a line containing two positive integers *N* (≤100), the number of crocodiles, and *D*, the maximum distance that James could jump. Then *N* lines follow, each containing the (*x*,*y*) location of a crocodile. Note that no two crocodiles are staying at the same position.



## Output Specification

For each test case, if James can escape, output in one line the minimum number of jumps he must make. Then starting from the next line, output the position (*x*,*y*) of each crocodile on the path, each pair in one line, from the island to the bank. If it is impossible for James to escape that way, simply give him 0 as the number of jumps. If there are many shortest paths, just output the one with the minimum first jump, which is guaranteed to be unique.



## Sample Input 1

```
17 15
10 -21
10 21
-40 10
30 -50
20 40
35 10
0 -10
-25 22
40 -40
-30 30
-10 22
0 11
25 21
25 10
10 10
10 35
-30 10

```



## Sample Output 1

```
4
0 11
10 21
10 35
```



## Sample Input 2

```
4 13
-12 12
12 12
-12 -12
12 -12

```



## Sample Output 2

```
0
```



## Solution

解法:
- 设一个空的起点，设一个空的终点
- 把第一跳能跳到的点和起点连起来
- 把最后能上岸的点和终点连起来
- 然后用 Dijkstra 算法求起点到终点的最短路径

**最后一句话决定了是用 Dijkstra 算法而不是 Unweighted**

> If there are many shortest paths, just output the one with the minimum first jump, which is guaranteed to be unique.

Dijkstra

```C

```

Unweighted

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#define MAX 102
#define ERROR -1

typedef struct {
    int x, y;
} coordinate;

typedef int ElementType;
typedef struct QNode *PtrToQNode;
struct QNode {
    ElementType Data;
    PtrToQNode Next;
};
typedef struct LinkQueue {
    PtrToQNode front;
    PtrToQNode rear;
} *Queue;

int n, d, dist[MAX], path[MAX];
coordinate croc[MAX];
int G[MAX][MAX];

void Input();
bool FirstJump(int);
bool Jump(int, int);
bool IsSafe(int);
void Save007();
void Unweighted(int);
Queue CreateQueue();
void EnQueue(ElementType, Queue);
ElementType DeQueue(Queue);
bool IsEmpty(Queue);
void Print(int);

int main()
{
    Input();
    Save007();
    return 0;
}

void Input()
{
    int i, j;
    scanf("%d%d", &n, &d);
    for (i = 1; i <= n; i++) {
        scanf("%d%d", &croc[i].x, &croc[i].y);
        if (FirstJump(i)) {
            G[0][i] = 1;
            G[i][0] = 1;
        }
        if (IsSafe(i)) {
            G[i][n + 1] = 1;
            G[n + 1][i] = 1;
        }
    }
    for (i = 1; i <= n; i++) {
        for (j = 1; j <= n; j++) {
            if (Jump(i, j)) {
                G[i][j] = 1;
                G[j][i] = 1;
            }
        }
    }
}

bool FirstJump(int v)
{
    return croc[v].x * croc[v].x + croc[v].y * croc[v].y <= (15 + d) * (15 + d);
}

bool IsSafe(int v)
{
    return croc[v].x + d >= 50 || croc[v].x - d <= -50 || croc[v].y + d >= 50 || croc[v].x - d <= -50;
}

bool Jump(int v, int w)
{
    return (croc[v].x - croc[w].x) * (croc[v].x - croc[w].x) + (croc[v].y - croc[w].y) * (croc[v].y - croc[w].y) <= d * d;
}

void Save007()
{
 
    Unweighted(0);
    printf("%d\n", dist[n + 1]);
    Print(n + 1);
}

void Unweighted(int S)
{
    for (int i = 0; i <= n + 1; i++) {
        dist[i] = -1;
        path[i] = -1;
    }
    Queue Q = CreateQueue();
    EnQueue(S, Q);
    dist[S] = 0;
    while (!IsEmpty(Q)) {
        int V = DeQueue(Q);
        for (int W = 0; W <= n + 1; W++) {
            if (G[V][W] && dist[W] == -1) {
                dist[W] = dist[V] + 1;
                path[W] = V;
                EnQueue(W, Q);
            }
        }
    }
}

void Print(int v)
{
    if (path[v] == -1) {

        return;
    }
    Print(path[v]);
    if (v != n + 1) printf("%d %d %d\n", croc[v].x, croc[v].y, v);   
}

Queue CreateQueue()
{
    PtrToQNode Qnode = malloc(sizeof(struct QNode));
    Queue Q = malloc(sizeof(struct LinkQueue));
    if (Qnode && Q) {
        Qnode->Data = 0;
        Qnode->Next = NULL;
        Q->front = Qnode;
        Q->rear = Qnode;
        return Q;
    } else
        return NULL;
}

void EnQueue(ElementType V, Queue Q)
{
    PtrToQNode Qnode = malloc(sizeof(struct QNode));
    Qnode->Data = V;
    Qnode->Next = NULL;
    Q->rear->Next = Qnode;
    Q->rear = Qnode;
    return;
}

ElementType DeQueue(Queue Q)
{
    ElementType V;
    PtrToQNode tmp;
    if (!IsEmpty(Q)) {
        V = Q->front->Next->Data;
        tmp = Q->front;
        Q->front = tmp->Next;
        free(tmp);
        return V;
    } else {
        printf("Queue is empty\n");
        return ERROR;
    }
}

bool IsEmpty(Queue Q)
{
    if (Q->front == Q->rear) return true;
    else return false;
}
```

