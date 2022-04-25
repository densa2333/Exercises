# 08-图8_How_Long_Does_It_Take

## Description

Given the relations of all the activities of a project, you are supposed to find the earliest completion time of the project.



## Input Specification

Each input file contains one test case. Each case starts with a line containing two positive integers *N* (≤100), the number of activity check points (hence it is assumed that the check points are numbered from 0 to *N*−1), and *M*, the number of activities. Then *M* lines follow, each gives the description of an activity. For the `i`-th activity, three non-negative numbers are given: `S[i]`, `E[i]`, and `L[i]`, where `S[i]` is the index of the starting check point, `E[i]` of the ending check point, and `L[i]` the lasting time of the activity. The numbers in a line are separated by a space.



## Output Specification

For each test case, if the scheduling is possible, print in a line its earliest completion time; or simply output "Impossible".



## Sample Input 1

```
9 12
0 1 6
0 2 4
0 3 5
1 4 1
2 4 1
3 5 2
5 4 0
4 6 9
4 7 7
5 7 4
6 8 2
7 8 4

```



## Sample Output 1

```
18
```



## Sample Input 2

```
4 5
0 1 1
0 2 2
2 1 3
1 3 4
3 2 5

```



## Sample Output 2

```
Impossible
```



## Solution

### 第二次尝试（AC）：输出各顶点最短时间里最大者

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#define MaxVertexNum 100
#define ERROR -1

typedef int ElementType;
typedef int Vertex;
typedef int WeightType;

typedef struct SNode *PtrToSNode;
struct SNode {
    ElementType Data;
    PtrToSNode Next;
};
typedef PtrToSNode Stack;

typedef struct ENode *PtrToENode;
struct ENode{
    Vertex V1, V2;
    WeightType Weight;
};
typedef PtrToENode Edge;

typedef struct AdjVNode *PtrToAdjVNode; 
struct AdjVNode{
    Vertex AdjV;
    WeightType Weight;
    PtrToAdjVNode Next;
};

typedef struct Vnode{
    PtrToAdjVNode FirstEdge;
} AdjList[MaxVertexNum];

typedef struct GNode *PtrToGNode;
struct GNode{  
    int Nv;
    int Ne;
    AdjList G;
};
typedef PtrToGNode LGraph;

LGraph CreateGraph(int);
void InsertEdge(LGraph, Edge);
LGraph BuildGraph();
Stack CreateStack();
bool IsEmpty (Stack);
bool Push(Stack, ElementType);
ElementType Pop(Stack);

void FindIndegree(LGraph);
bool TopOrder(LGraph);
void PrintEarliest(LGraph);

int Indegree[MaxVertexNum],
    Earliest[MaxVertexNum];

int main()
{
    int ret;
    LGraph Graph = BuildGraph();
    if (!TopOrder(Graph)) printf("Impossible");
    else PrintEarliest(Graph);
    return 0;
}

/* ---------------- Core Algorithm BEGIN ---------------- */
void FindIndegree(LGraph Graph)
{
    int i;
    PtrToAdjVNode V;

    for (i = 0; i < Graph->Nv; i++) Indegree[i] = 0;
    
    for (i = 0; i < Graph->Nv; i++) {
        for (V = Graph->G[i].FirstEdge; V; V = V->Next) {
            Indegree[V->AdjV]++;
        }
    }
}

bool TopOrder(LGraph Graph)
{
    int i, count, finishtime;
    Vertex V;
    PtrToAdjVNode W;
    Stack S = CreateStack();

    FindIndegree(Graph);
    for (i = 0; i < Graph->Nv; i++) if(Indegree[i] == 0) Push(S, i);
    for (i = 0; i < Graph->Nv; i++) Earliest[i] = 0;
    count = 0;

    while (!IsEmpty(S)) {
        V = Pop(S);
        count++;
        for (W = Graph->G[V].FirstEdge; W; W = W->Next) {
            if (--Indegree[W->AdjV] == 0)
                Push(S, W->AdjV);
            if (Earliest[V] + W->Weight > Earliest[W->AdjV])
                Earliest[W->AdjV] = Earliest[V] + W->Weight;
        }
    }
    if (count != Graph->Nv)
        return false;
    else
        return true;
}

void PrintEarliest(LGraph Graph)
{
    int i, max = Earliest[0];
    for (i = 0; i < Graph->Nv; i++) {
        if (Earliest[i] > max)
            max = Earliest[i];
    }
    printf("%d", max);
}
/* ---------------- Core Algorithm END ---------------- */

LGraph CreateGraph(int VertexNum)
{
    Vertex V;
    LGraph Graph;
    
    Graph = (LGraph)malloc(sizeof(struct GNode));
    Graph->Nv = VertexNum;
    Graph->Ne = 0;

    for (V=0; V<Graph->Nv; V++)
        Graph->G[V].FirstEdge = NULL;

    return Graph; 
}

void InsertEdge(LGraph Graph, Edge E)
{
    PtrToAdjVNode NewNode;
    
    NewNode = (PtrToAdjVNode)malloc(sizeof(struct AdjVNode));
    NewNode->AdjV = E->V2;
    NewNode->Weight = E->Weight;

    NewNode->Next = Graph->G[E->V1].FirstEdge;
    Graph->G[E->V1].FirstEdge = NewNode;
}

LGraph BuildGraph()
{
    LGraph Graph;
    Edge E;
    Vertex V;
    int Nv, i;
    
    scanf("%d", &Nv);
    Graph = CreateGraph(Nv);
    
    scanf("%d", &(Graph->Ne));
    if (Graph->Ne != 0) {
        E = (Edge)malloc(sizeof(struct ENode)); 
        for (i=0; i<Graph->Ne; i++) {
            scanf("%d %d %d", &E->V1, &E->V2, &E->Weight);
            InsertEdge(Graph, E);
        }
    }

    return Graph;
}

Stack CreateStack() 
{
    Stack S;
    S = (Stack)malloc(sizeof(struct SNode));
    S->Next = NULL;
    return S;
}

bool IsEmpty (Stack S)
{
    return ( S->Next == NULL );
}

bool Push(Stack S, ElementType X)
{
    PtrToSNode TmpCell;
    TmpCell = (PtrToSNode)malloc(sizeof(struct SNode));
    TmpCell->Data = X;
    TmpCell->Next = S->Next;
    S->Next = TmpCell;
    return true;
}

ElementType Pop(Stack S)  
{
    PtrToSNode FirstCell;
    ElementType TopElem;
    if( IsEmpty(S) ) {
        printf("Stack is empty"); 
        return ERROR;
    }
    else {
        FirstCell = S->Next; 
        TopElem = FirstCell->Data;
        S->Next = FirstCell->Next;
        free(FirstCell);
        return TopElem;
    }
}
```



### 第一次尝试（WA）：有多个终点的情况没考虑

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#define MaxVertexNum 100
#define ERROR -1

typedef int ElementType;
typedef int Vertex;
typedef int WeightType;

typedef struct SNode *PtrToSNode;
struct SNode {
    ElementType Data;
    PtrToSNode Next;
};
typedef PtrToSNode Stack;

typedef struct ENode *PtrToENode;
struct ENode{
    Vertex V1, V2;
    WeightType Weight;
};
typedef PtrToENode Edge;

typedef struct AdjVNode *PtrToAdjVNode; 
struct AdjVNode{
    Vertex AdjV;
    WeightType Weight;
    PtrToAdjVNode Next;
};

typedef struct Vnode{
    PtrToAdjVNode FirstEdge;
} AdjList[MaxVertexNum];

typedef struct GNode *PtrToGNode;
struct GNode{  
    int Nv;
    int Ne;
    AdjList G;
};
typedef PtrToGNode LGraph;

LGraph CreateGraph(int);
void InsertEdge(LGraph, Edge);
LGraph BuildGraph();
Stack CreateStack();
bool IsEmpty (Stack);
bool Push(Stack, ElementType);
ElementType Pop(Stack);

void FindIndegree(LGraph);
bool TopOrder(LGraph);

int Indegree[MaxVertexNum],
    Earliest[MaxVertexNum];

int main()
{
    int ret;
    LGraph Graph = BuildGraph();
    if (!TopOrder(Graph)) printf("Impossible");
    else printf("%d", Earliest[Graph->Nv - 1]); // IS NOT NECESSARILY THE LAST
    return 0;
}

/* ---------------- Core Algorithm BEGIN ---------------- */
void FindIndegree(LGraph Graph)
{
    int i;
    PtrToAdjVNode V;

    for (i = 0; i < Graph->Nv; i++) Indegree[i] = 0;
    
    for (i = 0; i < Graph->Nv; i++) {
        for (V = Graph->G[i].FirstEdge; V; V = V->Next) {
            Indegree[V->AdjV]++;
        }
    }
}

bool TopOrder(LGraph Graph)
{
    int i, count, finishtime;
    Vertex V;
    PtrToAdjVNode W;
    Stack S = CreateStack();

    FindIndegree(Graph);
    for (i = 0; i < Graph->Nv; i++) if(Indegree[i] == 0) Push(S, i);
    for (i = 0; i < Graph->Nv; i++) Earliest[i] = 0;
    count = 0;

    while (!IsEmpty(S)) {
        V = Pop(S);
        count++;
        for (W = Graph->G[V].FirstEdge; W; W = W->Next) {
            if (--Indegree[W->AdjV] == 0)
                Push(S, W->AdjV);
            if (Earliest[V] + W->Weight > Earliest[W->AdjV])
                Earliest[W->AdjV] = Earliest[V] + W->Weight;
        }
    }
    if (count != Graph->Nv)
        return false;
    else
        return true;
}
/* ---------------- Core Algorithm END ---------------- */

LGraph CreateGraph(int VertexNum)
{
    Vertex V;
    LGraph Graph;
    
    Graph = (LGraph)malloc(sizeof(struct GNode));
    Graph->Nv = VertexNum;
    Graph->Ne = 0;

    for (V=0; V<Graph->Nv; V++)
        Graph->G[V].FirstEdge = NULL;

    return Graph; 
}

void InsertEdge(LGraph Graph, Edge E)
{
    PtrToAdjVNode NewNode;
    
    NewNode = (PtrToAdjVNode)malloc(sizeof(struct AdjVNode));
    NewNode->AdjV = E->V2;
    NewNode->Weight = E->Weight;

    NewNode->Next = Graph->G[E->V1].FirstEdge;
    Graph->G[E->V1].FirstEdge = NewNode;
}

LGraph BuildGraph()
{
    LGraph Graph;
    Edge E;
    Vertex V;
    int Nv, i;
    
    scanf("%d", &Nv);
    Graph = CreateGraph(Nv);
    
    scanf("%d", &(Graph->Ne));
    if (Graph->Ne != 0) {
        E = (Edge)malloc(sizeof(struct ENode)); 
        for (i=0; i<Graph->Ne; i++) {
            scanf("%d %d %d", &E->V1, &E->V2, &E->Weight);
            InsertEdge(Graph, E);
        }
    }

    return Graph;
}

Stack CreateStack() 
{
    Stack S;
    S = (Stack)malloc(sizeof(struct SNode));
    S->Next = NULL;
    return S;
}

bool IsEmpty (Stack S)
{
    return ( S->Next == NULL );
}

bool Push(Stack S, ElementType X)
{
    PtrToSNode TmpCell;
    TmpCell = (PtrToSNode)malloc(sizeof(struct SNode));
    TmpCell->Data = X;
    TmpCell->Next = S->Next;
    S->Next = TmpCell;
    return true;
}

ElementType Pop(Stack S)  
{
    PtrToSNode FirstCell;
    ElementType TopElem;
    if( IsEmpty(S) ) {
        printf("Stack is empty"); 
        return ERROR;
    }
    else {
        FirstCell = S->Next; 
        TopElem = FirstCell->Data;
        S->Next = FirstCell->Next;
        free(FirstCell);
        return TopElem;
    }
}
```

