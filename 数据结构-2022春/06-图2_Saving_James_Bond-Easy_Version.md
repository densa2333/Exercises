# 06-图2_Saving_James_Bond-Easy_Version

## Description

This time let us consider the situation in the movie "Live and Let Die" in which James Bond, the world's most famous spy, was captured by a group of drug dealers. He was sent to a small piece of land at the center of a lake filled with crocodiles. There he performed the most daring action to escape -- he jumped onto the head of the nearest crocodile! Before the animal realized what was happening, James jumped again onto the next big head... Finally he reached the bank before the last crocodile could bite him (actually the stunt man was caught by the big mouth and barely escaped with his extra thick boot).

Assume that the lake is a 100 by 100 square one. Assume that the center of the lake is at (0,0) and the northeast corner at (50,50). The central island is a disk centered at (0,0) with the diameter of 15. A number of crocodiles are in the lake at various positions. Given the coordinates of each crocodile and the distance that James could jump, you must tell him whether or not he can escape.



## Input Specification

Each input file contains one test case. Each case starts with a line containing two positive integers *N* (≤100), the number of crocodiles, and *D*, the maximum distance that James could jump. Then *N* lines follow, each containing the (*x*,*y*) location of a crocodile. Note that no two crocodiles are staying at the same position.



## Output Specification

For each test case, print in a line "Yes" if James can escape, or "No" if not.



## Sample Input 1

```
14 20
25 -15
-25 28
8 49
29 15
-35 -2
5 28
27 -29
-8 -28
-20 -35
-25 -20
-13 29
-30 15
-35 40
12 12

```



## Sample Output 1

```
Yes
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
No
```



## Solution

核心算法：列出连通图的基础上修改

- 对每一条鳄鱼进行 DFS 之前先判断能不能从岛上跳到，对能第一条的鳄鱼进行 DFS ，如果有一条鳄鱼能逃脱（一个与岸连通的图）则跳出返回 Yes

DFS算法：（先序遍历）

- 先标记为踩过
- 如果可以直接到对岸就修改 answer 的值
- 不然就对每一条鳄鱼递归调用DFS，DFS 返回 answer 的值

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#define MAX 100

typedef struct {
    int x, y;
    bool visited;
} Vertex;

int n, d;
bool answer;
Vertex croc[MAX];

void Input();
bool FirstJump(int);
bool Jump(int, int);
bool IsSafe(int);
void Save007();
bool DFS(int);

int main()
{
    Input();
    Save007();
    return 0;
}

void Input()
{
    int i, x, y;
    scanf("%d%d", &n, &d);
    for (i = 0; i < n; i++) {
        scanf("%d%d", &croc[i].x, &croc[i].y);
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
    int i;
    for (i = 0; i < n; i++) {
        if (!croc[i].visited && FirstJump(i)) {
            answer = DFS(i);
            if (answer) break;
        }
    }
    if (answer) printf("Yes");
    else printf("No");
}

bool DFS(int v)
{
    int i;
    croc[v].visited = true;
    if (IsSafe(v)) answer = true;
    else {
        for (i = 0; i < n; i++) {
            if (!croc[i].visited && Jump(i, v)) {
                answer = DFS(i);
                if (answer) break;
            }
        }
    }
    return answer;
}
```

