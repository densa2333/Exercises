# 11-KMP_串的模式匹配

## 题目描述

给定两个由英文字母组成的字符串 String 和 Pattern，要求找到 Pattern 在 String 中第一次出现的位置，并将此位置后的 String 的子串输出。如果找不到，则输出“Not Found”。

本题旨在测试各种不同的匹配算法在各种数据情况下的表现。各组测试数据特点如下：

- 数据0：小规模字符串，测试基本正确性；
- 数据1：随机数据，String 长度为 10^5^，Pattern 长度为 10；
- 数据2：随机数据，String 长度为 10^5^，Pattern 长度为 10^2^；
- 数据3：随机数据，String 长度为 10^5^，Pattern 长度为 10^3^；
- 数据4：随机数据，String 长度为 10^5^，Pattern 长度为 10^4^；
- 数据5：String 长度为 106，Pattern 长度为 10^5^；测试尾字符不匹配的情形；
- 数据6：String 长度为 106，Pattern 长度为 10^5^；测试首字符不匹配的情形。



## 输入格式

输入第一行给出 String，为由英文字母组成的、长度不超过 106 的字符串。第二行给出一个正整数 *N*（≤10），为待匹配的模式串的个数。随后 *N* 行，每行给出一个 Pattern，为由英文字母组成的、长度不超过 105 的字符串。每个字符串都非空，以回车结束。



## 输出格式

对每个 Pattern，按照题面要求输出匹配结果。



## 样例输入

```
abcabcabcabcacabxy
3
abcabcacab
cabcabcd
abcabcabcabcacabxyz
```



## 样例输出

```
abcabcacabxy
Not Found
Not Found
```



## 解

KMP 算法

```C
#include <stdio.h>
#include <string.h> 
#include <stdlib.h>
#define MAXSIZE 1000001
#define NotFound -1
typedef int Position;

Position KMP(char *string, char *pattern);
void BuildMatch(char *pattern, int *match);

int main()
{
    char string[MAXSIZE], pattern[MAXSIZE];
    int n;
    scanf("%s", string);
    getchar();
    scanf("%d", &n);
    while (n--) {
        getchar();
        scanf("%s", pattern);
        Position p = KMP(string, pattern);
        if (p == NotFound) printf("Not Found\n");
        else printf("%s\n", string + p);
    }
    return 0;
}

Position KMP(char *string, char *pattern)
{
    int n = strlen(string);
    int m = strlen(pattern);
    int s, p, *match;

    match = (int *)malloc(sizeof(int) * m);
    BuildMatch(pattern, match);
    s = p = 0;
    while (s < n && p < m) {
        if (string[s] == pattern[p]) { s++; p++; }
        else if (p > 0) p = match[p - 1] + 1;
        else s++;
    }
    return (p == m) ? (s - m) : NotFound;
}

/* 动态规划 */
void BuildMatch(char *pattern, int *match)
{
    int i, j;
    int m = strlen(pattern);
    match[0] = -1; //初始条件
    for (j = 1; j < m; j++) {
        i = match[j - 1];
        while ((i >= 0) && (pattern[i + 1] != pattern[j]))
            i = match[i];
        if (pattern[i + 1] == pattern[j])
            match[j] = i + 1;
        else
            match[j] = -1;
    }
}
```

