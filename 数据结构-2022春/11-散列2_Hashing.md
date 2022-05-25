# 11-散列2_Hashing

## Description

The task of this problem is simple: insert a sequence of distinct positive integers into a hash table, and output the positions of the input numbers. The hash function is defined to be *H*(*key*)=*key*%*TSize* where *TSize* is the maximum size of the hash table. Quadratic probing (with positive increments only) is used to solve the collisions.

Note that the table size is better to be prime. If the maximum size given by the user is not prime, you must re-define the table size to be the smallest prime number which is larger than the size given by the user.



## Input Specification

Each input file contains one test case. For each case, the first line contains two positive numbers: *MSize* (≤10^4^) and *N* (≤*MSize*) which are the user-defined table size and the number of input numbers, respectively. Then *N* distinct positive integers are given in the next line. All the numbers in a line are separated by a space.



## Output Specification

For each test case, print the corresponding positions (index starts from 0) of the input numbers in one line. All the numbers in a line are separated by a space, and there must be no extra space at the end of the line. In case it is impossible to insert the number, print "-" instead.



## Sample Input

```
4 4
10 6 4 15
```



## Sample Output

```
0 1 4 -
```



## Solution

1.平方探测法以增量序列循环试探下一个地址。我的叫停方法是出现死循环即退出。因为平方探测法可能出现循环  
2.最小值测试点出错，没有考虑给出的MSize小于等于2的情况。MSize为1或者2时素数应该选择为 2（如果不特判就会赋值为 3）  
3.枚举类型enum格式上和struct类似。元素是int或者unsigned int类型。默认情况下从0开始赋值。

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <math.h>
#define MAXTABLESIZE 100000     /* 允许开辟的最大散列表长度 */

typedef int ElementType;    /* 关键词类型用整型 */
typedef int Index;          /* 散列地址类型 */
typedef Index Position;     /* 数据所在位置与散列地址是同一类型 */
/* 散列单元状态类型，分别对应：有合法元素、空单元、有已删除元素 */
typedef enum { Legitimate, Empty, Deleted } EntryType;

typedef struct HashEntry Cell; /* 散列表单元类型 */
struct HashEntry{
    ElementType Data; /* 存放元素 */
    EntryType Info;   /* 单元状态 */
};

typedef struct TblNode *HashTable; /* 散列表类型 */
struct TblNode {   /* 散列表结点定义 */
    int TableSize; /* 表的最大长度 */
    Cell *Cells;   /* 存放散列单元数据的数组 */
};

HashTable CreateTable( int TableSize );
int NextPrime( int N );
int Hash( int Key, int P );
Position Find( HashTable H, ElementType Key );
bool Insert( HashTable H, ElementType Key, int i );
void DestroyTable( HashTable H );

int main()
{
    int i, MSize, N;
    ElementType key;
    HashTable H;

    scanf("%d %d", &MSize, &N);
    H = CreateTable(MSize);
    for (i = 0; i < N; i++) {
        scanf("%d", &key);
        Insert(H, key, i); // 输出放在 Insert 里一并进行
    }
    DestroyTable(H);

    return 0;
}

HashTable CreateTable( int TableSize )
{
    HashTable H;
    int i;

    H = (HashTable)malloc(sizeof(struct TblNode));
    /* 保证散列表最大长度是素数 */
    H->TableSize = NextPrime(TableSize);
    /* 声明单元数组 */
    H->Cells = (Cell *)malloc(H->TableSize * sizeof(Cell));
    /* 初始化单元状态为“空单元” */
    for( i = 0; i < H->TableSize; i++ )
        H->Cells[i].Info = Empty;

    return H;
}

/* 返回大于N且不超过MAXTABLESIZE的最小素数 */
int NextPrime( int N )
{ 
    int i, p = (N % 2) ? N + 2 : N + 1; /*从大于N的下一个奇数开始 */
    /* 最小值用例未过，原因是MSize=1时没有特判，会把p赋为3 */
    if (N == 1) p = 2;
    
    while( p <= MAXTABLESIZE ) {
        for( i = (int)sqrt(p); i > 2; i-- )
            if ( !(p % i) ) break; /* p不是素数 */
        if ( i == 2 ) break; /* for正常结束，说明p是素数 */
        else  p += 2; /* 否则试探下一个奇数 */
    }
    return p;
}

/* 除留余数法散列函数 */
int Hash( int Key, int P )
{
    return Key % P;
}

Position Find( HashTable H, ElementType Key )
{
    Position CurrentPos, NewPos, OriPos;
    int CNum = 0; /* 记录冲突次数 */
    /* 记录初始散列位置，如果死循环永远找不到合法位置就退出并返回找不到 */
    OriPos = NewPos = CurrentPos = Hash( Key, H->TableSize ); /* 初始散列位置 */
    /* 当该位置的单元非空，并且不是要找的元素时，发生冲突 */
    while( H->Cells[NewPos].Info != Empty && H->Cells[NewPos].Data != Key ) {
        /* 统计1次冲突，这题不是双向探测 */
        ++CNum;
        NewPos = CurrentPos + CNum*CNum; /* 增量为CNum*CNum */
        while ( NewPos >= H->TableSize )
            NewPos = NewPos % H->TableSize; /* 调整为合法地址 */
        if (NewPos == OriPos) break; /* 进死循环直接退出初识散列地址 */
    }
    return NewPos; /* 此时NewPos或者是Key的位置，或者是一个空单元的位置（表示找不到）*/
}

bool Insert( HashTable H, ElementType Key, int i )
{
    Position Pos = Find( H, Key ); /* 先检查Key是否已经存在 */

    if( H->Cells[Pos].Info != Legitimate ) { /* 如果这个单元没有被占，说明Key可以插入在此 */
        H->Cells[Pos].Info = Legitimate;
        H->Cells[Pos].Data = Key;
        /*字符串类型的关键词需要 strcpy 函数!! */
        if (i) printf(" %d", Pos);
        else printf("%d", Pos);
        return true;
    }
    else {
        if (i) printf(" -");
        else printf("-");
        return false;
    }
}

void DestroyTable( HashTable H )
{
    free( H->Cells ); /* 释放头结点数组 */
    free( H );        /* 释放散列表结点 */
}
```

