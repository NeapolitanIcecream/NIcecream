---
layout: post
title: "ACM May Records"
description: "Monthly reports."
date: 2019-05-08
tags: [ACM, Algorithm, Network Flow]
comments: true
---

## [[费用流] Luogu P4452 [国家集训队]航班安排](https://www.luogu.org/problemnew/show/P4452)

一架飞机的一次包机由它经过非负整数次包机后，空载经过非负整数个机场转移得到，要求这次转移后时间不在这次包机的起始时刻之后，这次包机的终止时刻允许这架飞机在全局终止时刻之前回到全局起点机场。注意到空载时间和费用都满足弱的三角形不等式，空载转移可以直接在前一个状态所在机场和这次包机的起点机场之间进行，也就是不需要考虑空载飞机一通走位后到达起点机场。

建图如下：

+ 将包机按终止时刻排序，每次包机抽象为一个节点，记录那次包机的终止时刻和终点机场。枚举前面的所有节点，如果前面节点满足 $T_{end}(i) + T_{cost}(i, j) \le T_{start}(j)$ 和 $T_{end}(j) + T_{cost}(j, S) \le T_{end}$，在前面节点与这个节点之间连一条边，容量为 $1$，费用为 $C(i, j) - E(j)$。这描述了飞机从某个机场完成包机之后，空载转移到目标机场或就地接受包机的行为。
+ 建造源点和汇点，在源点与中间各点间建边如上，约定 $T_{end}(S) = 0$；在中间各点与汇点间建边如上，约定 $j = S$，这意味着第二个条件恒为真。特别地，在源点和汇点间建边，容量为 $\infty$，费用为 $0$。这描述了未接受过包机的飞机从全局起点机场空载飞往目标机场接受包机，以及飞机完成包机返回全局起点机场或始终在全局起点机场等待的行为。
+ 建造超级源点，在超级源点和源点之间连一条容量为 $k$，费用为 $0$ 的边。这描述了同时只能有 $k$ 架飞机活动的条件。

然后在超级源点和汇点之间跑最小费用流。

调试发现当中间节点有多条出边，多条入边时，多于一条入边可能被利用。每次包机只有一架飞机参与，因此希望每次只有一条入边被利用。

拆点如下：

+ 每次包机拆为两个节点。一个节点继承入边，一个节点继承出边；两节点间建边，容量为 $1$，费用为 $0$。

```cpp
#include <bits/extc++.h>
#include <bits/stdc++.h>

namespace PrimalDual {
  ...
}

const int N = 1000;
const int M = 50000;
const int INF = 0x7f7f7f7f;
int Tstart[M], Tend[M], Tendcity[M], Tcost[N][N], Cost[N][N], S, T, Ssuper, _N, _M, _K, _T;
struct event {
    int a;
    int b;
    int s;
    int t;
    int c;
} eve[M];

bool cmp(const event &a, const event &b) {
    return a.t < b.t;
}

using namespace std;

int main() {
    scanf("%d%d%d%d", &_N, &_M, &_K, &_T);
    S = 0;
    Ssuper = 2 * _M + 1;
    T = 2 * _M + 2;
    for (int i = 0; i < _N; i++)
        for (int j = 0; j < _N; j++)
            scanf("%d", &Tcost[i][j]);
    for (int i = 0; i < _N; i++)
        for (int j = 0; j < _N; j++)
            scanf("%d", &Cost[i][j]);
    for (int i = 1; i <= _M; i++)
        scanf("%d%d%d%d%d", &eve[i].a, &eve[i].b, &eve[i].s, &eve[i].t, &eve[i].c);
    sort(eve + 1, eve + _M + 1, cmp);
    for (int i = 1; i <= _M; i++) {
        Tendcity[i] = eve[i].b;
        Tend[i] = eve[i].t;
        PrimalDual::addEdge(2 * i - 1, 2 * i, 1, 0);
        if (eve[i].t + Tcost[eve[i].b][0] <= _T)
            for (int j = 0; j < i; j++)
                if (Tend[j] + Tcost[Tendcity[j]][eve[i].a] <= eve[i].s)
                    PrimalDual::addEdge(2 * j, 2 * i - 1, 1, Cost[Tendcity[j]][eve[i].a] - eve[i].c);
    }
    for (int i = 1; i <= _M; i++)
        if (eve[i].t + Tcost[eve[i].b][0] <= _T)
            PrimalDual::addEdge(2 * i, T, 1, Cost[Tendcity[i]][0]);
    PrimalDual::addEdge(S, T, INF, 0);
    PrimalDual::addEdge(Ssuper, S, _K, 0);
    pair ans = PrimalDual::minCostFlow(Ssuper, T, 2 * _M + 3, INF);
    printf("%d\n", -ans.second);
    return 0;
}
```

