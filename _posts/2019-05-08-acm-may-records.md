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

## [[费用流] NOI 2012 美食节](https://bzoj.nicecream.top/JudgeOnline/2879.html)

厨师行为的影响与一种类似时间的量度有关。观察单个厨师总等待时间的组成，注意到第 i 次做的菜对总等待时间贡献 `w[j] - i + 1` 次，其中 `w[j]` 表示第 j 个厨师做菜的总数。这是分层网络的特征。建图如下。

+ 为每道菜建点。建源点 S。连接 S 与每道菜，容量为 `p[i]`，费用为 0。这描述了每道菜做 `p[i]` 次这一条件。
+ 为每个厨师建点，一共建 n 层。连接菜 i 与第 j 层的所有厨师，容量为 1，费用为 `j * t[i][k]`，其中 k 表示第 k 个厨师。建汇点 T，连接所有层的所有厨师与 T，容量为 1，费用为 0。注意费用公式保证了厨师总会先完成低层数的菜。这**倒序**地描述了第 k 个厨师做的第 `w[k] - j + 1` 道菜为菜 i，尽管我们实际上还不知道 `w[k]`。

```cpp
#include <bits/extc++.h>
#include <bits/stdc++.h>

const int maxn = 40 + 5;
const int maxm = 100 + 5;
const int maxp = 800 + 5;
int n, m, t[maxn][maxm], p[maxn], S, T;
std::pair<int, int> ans;

namespace PrimalDual {
...
}

int main() {
    scanf("%d%d", &n, &m);
    S = n + n * m;
    T = n + n * m + 1;
    tot = n + n * m + 2;
    for (int i = 0; i < n; i++)
        scanf("%d", &p[i]);
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            scanf("%d", &t[i][j]);
    for (int i = 0; i < n; i++) {
        PrimalDual::addEdge(S, i, p[i], 0);
        for (int j = 0; j < n; j++)
            for (int k = 0; k < m; k++)
                PrimalDual::addEdge(i, n + j * m + k, 1, (j + 1) * t[i][k]);
    }
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            PrimalDual::addEdge(n + i * m + j, T, 1, 0);
    ans = PrimalDual::minCostFlow(S, T, tot, maxp);
    printf("%d", ans.second);
    return 0;
}
```

注意到题目有很短的和很少的有效增广路，这个数据规模虽然很大，但是时间可能不会耗费太多。如果还需要优化，可以魔改最短路算法，约束它能访问的层数；或者动态开点，仅当上一层同一点满流后才开下一层的对应点。

## [[DP] Luogu P4363 [九省联考2018]一双木棋chess](https://www.luogu.org/problemnew/show/P4363)

状态稀疏的记忆化对抗搜索，设计一种编码方案指示状态即可，可以状压也可以进制哈希。状态为「除当前局面外的最优解」，「局面」指每行存在若干个棋子唯一指示的一种情况。边界条件为 mem[tar] = 0，tar 指示了棋子填满时的情况，这时除当前局面外的最优解为 0，因为双方都不能再落子。每次由除当前局面外的最优解和当前位置的棋子 Minimax 递归转移。

```cpp
#include <bits/stdc++.h>

using namespace std;

const int INF = 0x3f3f3f3f;
const int MAXN = 10;
const int MAXM = 10;
int decimal, n, m, a[MAXN][MAXM], b[MAXN][MAXM], num[MAXN];
unordered_map<long long, int> mem;

inline long long encode() {
    long long encoded = 0;
    for (int i = 0; i < n; i++)
        encoded = decimal * encoded + num[i];
    return encoded;
}

inline void decode(long long secret) {
    for (int i = n - 1; i >= 0; i--) {
        num[i] = secret % decimal;
        secret /= decimal;
    }
}

int search(long long secret, bool first) {
    if (mem.count(secret))
        return mem[secret];
    decode(secret);
    int ans = first ? -INF : INF;
    for (int i = 0; i < n; i++) {
        if ((i == 0 || num[i] < num[i - 1]) && num[i] < m) {
            long long tar;
            num[i]++;
            tar = encode();
            ans = first ? max(ans, a[i][num[i] - 1] + search(tar, !first)) : min(ans, -b[i][num[i] - 1] +
                                                                                      search(tar, !first));
            num[i]--;
        }
    }
    mem[secret] = ans;
    return ans;
}

int main() {
    int ans;
    long long tar;
    scanf("%d%d", &n, &m);
    decimal = m + 1;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            scanf("%d", &a[i][j]);
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            scanf("%d", &b[i][j]);
    for (int i = 0; i < n; i++)
        num[i] = m;
    tar = encode();
    mem[tar] = 0;
    ans = search(0, true);
    printf("%d\n", ans);
    return 0;
}
```

