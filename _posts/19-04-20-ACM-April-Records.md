---
layout: post
title: ACM April Records
comments: True
---

## [[DP/生成函数] Loj 515「LibreOJ β Round #2」贪心只能过样例](https://loj.ac/problem/515)

1. 想出暴力：维护一大堆 S，依次计算每个数给 S 集合带来的变化。
2. （Fork）看上去可以生成函数。但是先考虑了下面的步骤。
3. 用一个标记数组来维护 S 集合。
4. 优化一：后继状态只由一个前导状态得出，滚动求值。
5. 优化二：数组中保存的信息只关心存在性（0/1），并且观察到每次推导都做大量重复工作（所有 1 位变化相同距离），故 Bitset 优化，移位相或维护 S 集合。复杂度 $O(N^2)$。

```CPP
#include <bits/stdc++.h>
using namespace std;
int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    bitset<1000010> bs0, bs1;
    int n, a, b;
    cin >> n;
    bs0.reset();
    bs1.reset();
    bs0[0] = 1;
    for (auto i = 0; i < n; i++) {
        cin >> a >> b;
        for (auto j = a; j <= b; j++) {
            bs1 |= bs0 << (j * j);
        }
        bs0 = bs1;
        bs1.reset();
    }
    cout << bs0.count() << endl;
    return 0;
}
```

5. 接下来整一个生成函数。$S =\prod_{i=1}^n (x^{{a_i}^2} + x^{{(a_i + 1)}^2} + … + x^{{b_i}^2})$。预处理上标后全部 FFT，计算 S 的点值，然后 DFT。瓶颈是 FFT，复杂度 $O(N^2\log N)$。由于 DP 常数巨大，生成函数会更快。

## [[线段树/树套树] Luogu P1903 [国家集训队]数颜色 / 维护队列](https://www.luogu.org/problemnew/show/P1903)

1. 暴力：Q(L, R)，遍历区间，O(N)；R(P, Col)，单点修改，O(1)。
2. 分析：一维空间，一个信息（Bitset：颜色存在性），满足可加性（相或合并），单点修改，区间查询。线段树冲冲冲。复杂度 $O(N\log N)$，最坏复杂度 $O(N^2\log N)$。

```CPP
#include <bits/stdc++.h>
using namespace std;

int n, m;
bitset<1000010> d[50010];
int a[50010];

int getsum(int l, int r, int s, int t, int p) {
    if (l <= s && t <= r)
        return d[p].count();
    int m = (s + t) / 2, sum = 0;
    if (l <= m)
        sum += getsum(l, r, s, m, p * 2);
    if (r > m)
        sum += getsum(l, r, m + 1, t, p * 2 + 1);
    return sum;
}

void build(int s, int t, int p) {
    if (s == t) {
        d[p].set(a[s]);
        return;
    }
    int m = (s + t) / 2;
    build(s, m, p * 2), build(m + 1, t, p * 2 + 1);
    d[p] = d[p * 2] | d[(p * 2) + 1];
}

void change(int s, int t, int p, int pos, int col) {
    if (s == t) {
        d[p].reset(a[s]);
        a[s] = col;
        d[p].set(a[s]);
        return;
    }
    int m = (s + t) / 2;
    if (pos <= m) {
        change(s, m, p * 2, pos, col);
        d[p] = d[p * 2] | d[(p * 2) + 1];
    } else {
        change(m + 1, t, p * 2 + 1, pos, col);
        d[p] = d[p * 2] | d[(p * 2) + 1];
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    char flag;
    int p, q;
    cin >> n >> m;
    for (auto i = 1; i <= n; i++)
        cin >> a[i];
    build(1, n, 1);
    for (auto i = 1; i <= m; i++) {
        cin >> flag >> p >> q;
        if (flag == 'Q')
            cout << getsum(p, q, 1, n, 1) << endl;
        else
            change(1, n, 1, p, q);
    }
    return 0;
}
```

3. 最坏复杂度看上去比较傻屌，所以想别的模型。一个经典的去重：询问 [L, R] 区间中不同颜色的个数，等价于询问区间中的颜色，有多少种最近一次出现的位置在 L 之前，即 $\sum_{i=L}^R \{last[i] \lt L\}$。于是得到了一个区间 K 大问题的变式，版本为 $L$，维护信息 $last[i]$。额外要求需要快速查找修改位置相关颜色的前驱后继，选择 `set` 维护每一种颜色存在的位置。注意到二维化后使用了红黑树而非主席树，本题数据规模下区间权值线段树内存不够。

```CPP
#include <bits/stdc++.h>
#include <bits/extc++.h>

#define MAXN 50000
#define MAXC 1000000
#define MAXV 100000

using namespace std;
using namespace __gnu_pbds;

typedef pair<int, int> pii;
typedef tree<pii, null_type, less<pii>, rb_tree_tag, tree_order_statistics_node_update> rbtree;

set<int> col_set[MAXC + 1];
int loc_to_col[MAXN + 1];
rbtree ver[MAXV + 1];
unordered_map<int, int> cnt[MAXV + 1];

inline int lowbit(int x) {
    return x & (-x);
}

void update(int tree_num, int num, int flag) {
#define TREE ver[tree_num]
    if (flag) {
        if (!cnt[tree_num].count(num))
            cnt[tree_num][num] = 1;
        else
            cnt[tree_num][num]++;
        TREE.insert(make_pair(num, cnt[tree_num][num]));
    } else {
        TREE.erase(make_pair(num, cnt[tree_num][num]--));
        if (!cnt[tree_num][num])
            cnt[tree_num].erase(num);
    }
#undef TREE
}

int query(int l, int r, int v) {
    int ans = 0;
    pii sch = make_pair(v, 0);
    for (int i = r; i > 0; i -= lowbit(i))
        ans += ver[i].order_of_key(sch);
    if (l != 1) {
        for (int i = l - 1; i > 0; i -= lowbit(i))
            ans -= ver[i].order_of_key(sch);
    }
    return ans;
}

void modify(int tree_num, int prv, int nxt) {
    for (int i = tree_num; i <= MAXV; i += lowbit(i)) {
        update(i, prv, 0);
        update(i, nxt, 1);
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        int last = 0;
        cin >> loc_to_col[i];
        auto iter = col_set[loc_to_col[i]].insert(i).first;
        if (iter != col_set[loc_to_col[i]].begin())
            last = *(--iter);
        for (int j = i; j <= MAXV; j += lowbit(j))
            update(j, last, 1);
    }
    for (int i = 1; i <= m; i++) {
        char op;
        cin >> op;
        if (op == 'Q') {
            int l, r;
            cin >> l >> r;
            cout << query(l, r, l) << endl;
        } else {
#define PRE col_set[loc_to_col[j]]
#define NEW col_set[k]
            int j, k, pre_last, pre_next, new_last, new_next;
            bool pre_has_next, new_has_next;
            cin >> j >> k;
            auto i_ltc_j_j = PRE.find(j);
            auto i_k_j = NEW.upper_bound(j);
            if (i_ltc_j_j == PRE.begin())
                pre_last = 0;
            else {
                pre_last = *(--i_ltc_j_j);
                ++i_ltc_j_j;
            }
            if ((++i_ltc_j_j) == PRE.end())
                pre_has_next = 0;
            else {
                pre_next = *i_ltc_j_j;
                pre_has_next = 1;
            }
            if (pre_has_next)
                modify(pre_next, j, pre_last);
            if (i_k_j == NEW.end())
                new_has_next = 0;
            else {
                new_next = *i_k_j;
                new_has_next = 1;
            }
            if (i_k_j == NEW.begin())
                new_last = 0;
            else
                new_last = *(--i_k_j);
            if (new_last != pre_last)
                modify(j, pre_last, new_last);
            if (new_has_next)
                modify(new_next, new_last, j);
            PRE.erase(j);
            NEW.insert(j);
            loc_to_col[j] = k;
#undef PRE
#undef NEW
        }
    }
    return 0;
}
```

