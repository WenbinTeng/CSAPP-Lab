# CS:APP Cache Lab

### Preface

```
.logic_lru_cache_line
+ - - - + - - - + - - - - - - + - - - - + - - - - - - - - - - - +
| valid |  lru  |     tag     |  index  |         data          |
+ - - - + - - - + - - - - - - + - - - - + - - - - - - - - - - - +
```

### Part A

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <limits.h>
#include <getopt.h>
#include "cachelab.h"

typedef struct {
    int vld;
    int lru;
    int tag;
} CacheLine;

int main(int argc, char** argv)
{
    int opt;
    int s;  // Number of set index bits
    int E;  // Number of lines per set
    int b;  // Number of block bits
    char *filename = "";
    long long time = 0;
    int hitCnt = 0;
    int misCnt = 0;
    int eviCnt = 0;

    while(-1 != (opt = getopt(argc, argv, "s:E:b:t:")))
    {
        switch (opt)
        {
        case 's':
            s = atoi(optarg);
            break;
        case 'E':
            E = atoi(optarg);
            break;
        case 'b':
            b = atoi(optarg);
            break;
        case 't':
            filename = (char*)optarg;
        default:
            break;
        }
    }

    CacheLine **cache = (CacheLine**)malloc((1 << s) * sizeof(CacheLine*));

    for (int i = 0; i < (1 << s); ++i)
    {
        cache[i] = (CacheLine*)malloc(E * sizeof(CacheLine));
        memset(cache[i], 0, E * sizeof(CacheLine));
    }

    char id;
    unsigned long long addr;
    unsigned int size;
    FILE *fp = fopen(filename, "r");
    while (fscanf(fp, " %c %llx,%d", &id, &addr, &size) > 0)
    {
        if (id == 'I') continue;

        int _tag = addr >> (s + b);
        int _idx = (addr >> b) & ~(0xffffffff << s);

        int minVal = INT_MAX;
        int minIdx = 0;
        
        int isHit = 0;
        int isEvi = 0;

        for (int i = 0; i < E; ++i)
        {
            CacheLine *cl = &cache[_idx][i];

            if (!cl->vld)
            {
                minVal = INT_MIN;
                minIdx = i;
                isEvi = 0;
                continue;
            }
            if (cl->tag != _tag)
            {
                if (minVal > cl->lru)
                {
                    minVal = cl->lru;
                    minIdx = i;
                    isEvi = 1;
                }
                continue;
            }

            minIdx = i;
            isHit = 1;
            break;
        }
        cache[_idx][minIdx].lru = time;
        ++time;
        if (id == 'M') { ++hitCnt; }
        if (isHit) { ++hitCnt; continue; }
        if (isEvi) { ++eviCnt; }
        ++misCnt;
        cache[_idx][minIdx].vld = 1;
        cache[_idx][minIdx].tag = _tag;
    }

    printSummary(hitCnt, misCnt, eviCnt);

    return 0;
}

```

```
                        Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
     3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
     3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
     3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
     3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
     3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
     3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
     6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
    27

TEST_CSIM_RESULTS=27
```

### Part B

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N])
{
    for (int i = 0; i < M; i += 8)
    {
        for (int j = 0; j < N; j += 8)
        {
            for (int k = i; k < i + 8; k += 1)
            {
                int v1 = A[k][j + 0];
                int v2 = A[k][j + 1];
                int v3 = A[k][j + 2];
                int v4 = A[k][j + 3];
                int v5 = A[k][j + 4];
                int v6 = A[k][j + 5];
                int v7 = A[k][j + 6];
                int v8 = A[k][j + 7];
                B[j + 0][k] = v1;
                B[j + 1][k] = v2;
                B[j + 2][k] = v3;
                B[j + 3][k] = v4;
                B[j + 4][k] = v5;
                B[j + 5][k] = v6;
                B[j + 6][k] = v7;
                B[j + 7][k] = v8;
            }
        }
    }
}
```

```
# 32*32

Function 0 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:1766, misses:287, evictions:255

Function 1 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 1 (Simple row-wise scan transpose): hits:870, misses:1183, evictions:1151

Summary for official submission (func 0): correctness=1 misses=287

TEST_TRANS_RESULTS=1:287
```

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N])
{
    for (int i = 0; i < N; i += 8)
    {
        for (int j = 0; j < M; j += 8)
        {
            for (int k = 0; k < 4; k += 1)
            {
                int v1 = A[k + i][j + 0];
                int v2 = A[k + i][j + 1];
                int v3 = A[k + i][j + 2];
                int v4 = A[k + i][j + 3];
                int v5 = A[k + i][j + 4];
                int v6 = A[k + i][j + 5];
                int v7 = A[k + i][j + 6];
                int v8 = A[k + i][j + 7];
                B[j + 0][k + i] = v1;
                B[j + 1][k + i] = v2;
                B[j + 2][k + i] = v3;
                B[j + 3][k + i] = v4;
                B[j + 0][k + 4 + i] = v5;
                B[j + 1][k + 4 + i] = v6;
                B[j + 2][k + 4 + i] = v7;
                B[j + 3][k + 4 + i] = v8;
            }
            for (int k = 0; k < 4; k += 1)
            {
                
                int v1 = A[i + 4][j + k], v5 = A[i + 4][j + k + 4];
                int v2 = A[i + 5][j + k], v6 = A[i + 5][j + k + 4];
                int v3 = A[i + 6][j + k], v7 = A[i + 6][j + k + 4];
                int v4 = A[i + 7][j + k], v8 = A[i + 7][j + k + 4];
                int vt;
                vt = B[j + k][i + 4], B[j + k][i + 4] = v1, v1 = vt;
                vt = B[j + k][i + 5], B[j + k][i + 5] = v2, v2 = vt;
                vt = B[j + k][i + 6], B[j + k][i + 6] = v3, v3 = vt;
                vt = B[j + k][i + 7], B[j + k][i + 7] = v4, v4 = vt;
                B[j + k + 4][i + 0] = v1, B[j + k + 4][i + 4 + 0] = v5;
                B[j + k + 4][i + 1] = v2, B[j + k + 4][i + 4 + 1] = v6;
                B[j + k + 4][i + 2] = v3, B[j + k + 4][i + 4 + 2] = v7;
                B[j + k + 4][i + 3] = v4, B[j + k + 4][i + 4 + 3] = v8;
            }
        }
    }
}
```

```
# 64*64

Function 0 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:9138, misses:1107, evictions:1075

Function 1 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 1 (Simple row-wise scan transpose): hits:3474, misses:4723, evictions:4691

Summary for official submission (func 0): correctness=1 misses=1107

TEST_TRANS_RESULTS=1:1107
```

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N])
{
    for (int i = 0; i < N; i += 8)
    {
        for (int j = 0; j < M; j += 23)
        {
            if (i + 8 <= N && j + 23 <= M)
            {
                for (int k = j; k < j + 23; k += 1)
                {
                    int v1 = A[i][k];
                    int v2 = A[i + 1][k];
                    int v3 = A[i + 2][k];
                    int v4 = A[i + 3][k];
                    int v5 = A[i + 4][k];
                    int v6 = A[i + 5][k];
                    int v7 = A[i + 6][k];
                    int v8 = A[i + 7][k];
                    B[k][i + 0] = v1;
                    B[k][i + 1] = v2;
                    B[k][i + 2] = v3;
                    B[k][i + 3] = v4;
                    B[k][i + 4] = v5;
                    B[k][i + 5] = v6;
                    B[k][i + 6] = v7;
                    B[k][i + 7] = v8;
                }
            }
            else
            {
                for (int s = i; s < min(i + 8, N); ++s)
                {
                    for (int t = j; t < min(j + 23, M); ++t)
                    {
                        B[t][s] = A[s][t];
                    }
                }
            }
        }
    }
}
```

```
# 61*67

Function 0 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:6316, misses:1863, evictions:1831

Function 1 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 1 (Simple row-wise scan transpose): hits:3756, misses:4423, evictions:4391

Summary for official submission (func 0): correctness=1 misses=1863

TEST_TRANS_RESULTS=1:1863
```

