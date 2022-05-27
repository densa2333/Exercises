# 10-排序5_PAT_Judge

## Description

The ranklist of PAT is generated from the status list, which shows the scores of the submissions. This time you are supposed to generate the ranklist for PAT.



## Input Specification

Each input file contains one test case. For each case, the first line contains 3 positive integers, *N* (≤104), the total number of users, *K* (≤5), the total number of problems, and *M* (≤105), the total number of submissions. It is then assumed that the user id's are 5-digit numbers from 00001 to *N*, and the problem id's are from 1 to *K*. The next line contains *K* positive integers `p[i]` (`i = 1, ..., K`), where `p[i]` corresponds to the full mark of the i-th problem. Then *M* lines follow, each gives the information of a submission in the following format:

```
user_id problem_id partial_score_obtained
```

where `partial_score_obtained` is either −1 if the submission cannot even pass the compiler, or is an integer in the range [0, `p[problem_id]`]. All the numbers in a line are separated by a space.



## Output Specification

For each test case, you are supposed to output the ranklist in the following format:

```
rank user_id total_score s[1] ... s[K]
```

where `rank` is calculated according to the `total_score`, and all the users with the same `total_score` obtain the same `rank`; and `s[i]` is the partial score obtained for the `i`-th problem. If a user has never submitted a solution for a problem, then "-" must be printed at the corresponding position. If a user has submitted several solutions to solve one problem, then the highest score will be counted.

The ranklist must be printed in non-decreasing order of the ranks. For those who have the same rank, users must be sorted in nonincreasing order according to the number of perfectly solved problems. And if there is still a tie, then they must be printed in increasing order of their id's. For those who has never submitted any solution that can pass the compiler, or has never submitted any solution, they must NOT be shown on the ranklist. It is guaranteed that at least one user can be shown on the ranklist.



## Sample Input

```
7 4 20
20 25 25 30
00002 2 12
00007 4 17
00005 1 19
00007 2 25
00005 1 20
00002 2 2
00005 1 15
00001 1 18
00004 3 25
00002 2 25
00005 3 22
00006 4 -1
00001 2 18
00002 1 20
00004 1 15
00002 4 18
00001 3 4
00001 4 2
00005 2 -1
00004 2 0

```



## Sample Output

```
1 00002 63 20 25 - 18
2 00005 42 20 0 22 -
2 00007 42 - 25 - 17
2 00001 42 18 18 4 2
5 00004 40 15 0 25 -
```



## Solution

详见注释

```C

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

enum flag {EXIST = 1, NEVER = 0, CE = -1};

/* 选手结构体定义 */
typedef struct {
    int id;         // 选手ID
    int rank;       // 选手排名，在Output函数中处理
    int total;      // 选手总分，在Input函数中处理
    //（total=-1时说明不参与排序，在qsort中自动被扔到最后，同时在Output中作为输出控制）
    int partial[6]; // 选手各小题得分
    int flag[6];    // 选手各小题提交状态（提交有效答案，从来没提交，提交了但编译错误）
    int perfect;    // 选手拿到满分的题数
} entrant;

int n, k; //选手总数与题目总数
int full[6]; //各题满分
entrant user[10001]; //选手数组

int cmp(const void *a, const void *b);
void Input();
bool PerfectlySolved(int, int);
bool NeverSubmmit(int);
void Output();

int main()
{
    Input();    // 处理输入
    qsort(user + 1, n, sizeof(entrant), cmp); // 数组排序
    Output();   // 处理输出
    return 0;
}

/* 排序规则 */
int cmp(const void *a, const void *b)
{
    // 总分高的排在前面
    if (((entrant *)a)->total > (((entrant *)b)->total)) return -1;
    else if (((entrant *)a)->total == (((entrant *)b)->total)) // 如果总分相同
        // 满分题数多的排在前面
        if (((entrant *)a)->perfect > (((entrant *)b)->perfect)) return -1;
        else if (((entrant *)a)->perfect == (((entrant *)b)->perfect)) // 如果满分题数也相同
            // ID小的排在前面
            if (((entrant *)a)->id < (((entrant *)b)->id)) return -1;
            else return 1;
        else return 1;
    else return 1;
}

/* 输入处理 */
void Input()
{
    int i, j, m, id, pnum, score;

    // 第一行3个数分别为选手总数、题目总数、提交总数
    scanf("%d %d %d", &n, &k, &m);

    // 第二行k个数为各题满分
    for (i = 1; i <= k; i++)
        scanf("%d", &full[i]);
    
    // 接下来m行为所有的提交记录，读入并处理
    for (i = 0; i < m; i++) {
        scanf("%d %d %d", &id, &pnum, &score);
        // 先按ID顺序读入ID
        user[id].id = id;
        // 更新i选手pnum号题的提交状况，注意同一题有多次提交的情况，只能有效提交覆盖无效提交
        if (score >= 0) user[id].flag[pnum] = EXIST; //分数大于等于零为有效提交
        else if (user[id].flag[pnum] == NEVER) user[id].flag[pnum] = CE; //分数-1为编译错误
        // 更新i选手pnum号题的分数（每次读入都高分覆盖低分）
        if (score > user[id].partial[pnum]) user[id].partial[pnum] = score;
    }

    // 计算每个选手的总分
    for (i = 1; i <= n; i++)
        if (NeverSubmmit(i)) // 如果不存在任何有效提交
            user[i].total = -1; // total=-1
        else // 如果存在有效提交，计算总分
            for (j = 1; j <= k; j++)
                user[i].total += user[i].partial[j];

    // 计算每个选手的满分题数
    for (i = 1; i <= n; i++) {
        for (j = 1; j <= k; j++) {
            if (PerfectlySolved(i, j)) user[i].perfect++;
        }
    }
}

/* 判断id选手pid号题是否满分 */
bool PerfectlySolved(int id, int pid)
{
    return user[id].partial[pid] == full[pid];
}

/* 判断是否不存在任何有效提交 */
bool NeverSubmmit(int id)
{
    int i;
    bool ret = true;
    // 若存在有效提交就跳出并返回false
    for (i = 1; i <= k && ret; i++) if (user[id].flag[i] == EXIST) ret = false;
    return ret;
}

/* 输出处理 */
void Output()
{
    int i, j, rank = 1;
    
    // 迭代计算每个选手排名
    user[1].rank = rank;
    i = 2;
    while (i <= n) {
        if (user[i].total != user[i - 1].total) rank = i; //不并列则排名跳转
        user[i].rank = rank;
        i++;
    }

    // 输出结果，后面所有total=-1的都不用输出
    for (i = 1; i <= n && user[i].total != -1; i++) {
        printf("%d %05d %d", user[i].rank, user[i].id, user[i].total);
        for (j = 1; j <= k; j++) {
            // 输出i选手j号题小分时，只要提交了就显示分数，编译错误算0分，从未提交才输出"-"
            if (user[i].flag[j] != NEVER) printf(" %d", user[i].partial[j]);
            else printf(" -");
        }
        printf("\n");
    }
}
```

