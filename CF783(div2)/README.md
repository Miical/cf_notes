比赛链接：https://codeforces.com/contest/1668

官方题解：https://codeforces.com/blog/entry/102013

## A. Direction Change

https://codeforces.com/contest/1668/problem/A

```C++
#include <iostream>
#include <cstdio>
#include <cstring>
using namespace std;
int main() {
  int T; scanf("%d", &T);
  while (T--) {
    int n, m; scanf("%d%d", &n, &m);
    if (n > m) swap(n, m);
    if (n == 1) {
      if (m <= 2) 
        printf("%d\n", m - 1);
      else
        printf("-1\n");
      continue;
    }

    int ans = 2 * (n - 1);
    if (m > n) ans++, m--;
    if (m > n) ans += (m - n) / 2 * 4 + (m - n) % 2 * 3;
    printf("%d\n", ans);
  }
  return 0;
}
```



## B. Social Distance

https://codeforces.com/contest/1668/problem/B

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
using namespace std;
typedef long long LL;
const LL INF = 1e18;
int main() {
  int T; scanf("%d", &T);
  while(T--) {
    int n, m; scanf("%d%d", &n, &m);
    LL Min = INF, Max = -INF, sum = 0; 
    for (int i = 1; i <= n; i++) {
      LL x; scanf("%lld", &x);
      Min = min(Min, x);
      Max = max(Max, x);
      sum = sum + x;
    }
    printf(sum + Max - Min + n <= m? "YES\n": "NO\n");
  }
  return 0; 
} 
```



## C. Make it Increasing

https://codeforces.com/contest/1668/problem/C

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
using namespace std;
typedef long long LL;
const LL INF = 1e18; 
const int MAXN = 5005;
int a[MAXN];

int main() {
  int n; scanf("%d", &n);
  for (int i = 1; i <= n; i++) 
    scanf("%d", &a[i]);

  LL ans = INF; 
  for (int pos = 1; pos <= n; pos++) {
    LL num = 0, sum = 0;
    for (int i = pos + 1; i <= n; i++) {
      LL t = num / a[i] + 1LL;
      sum += t;
      num = t * a[i];
    }
    num = 0;
    for (int i = pos - 1; i >= 1; i--) {
      LL t = num / a[i] + 1LL;
      sum += t; 
      num = t * a[i];
    }
    ans = min(ans, sum);
  }
  printf("%lld\n", ans);
  return 0; 
} 
```



## D. Optimal Partition

https://codeforces.com/contest/1668/problem/D

### 题目大意

给定数组a (1<=n<=5e5)，把a划分任意个连续的子序列。每个子序列有贡献值，如果序列和为正则贡献值为序列长度，序列和为0贡献值为0，序列和为负贡献值为负的长度，求最大贡献和。

### 题解

一条结论：

对于小于等于0的元素，一定存在一种最优方案，使得或者他们一个元数作为一段，或者被添加到正数段中。不会出现几个元素连在一起成为负段或零段，否则我们一定可以把他们拆开使得结果不下降。（证明可见官方题解）

所以对于小于等于0的元素，我们只用考虑两种情况，一是单独成段，二是添加到正数段中。

回到题目，本题有十分显然的$O(n^2)$算法，设$f(n)$为前n个元素的最优解，则$f[i] = max(f[j]+val_{j+1, i}), j < i$

之后尝试使用本题的特点去优化方程

- 考虑第一种情况，当前元素单独成段，则

  $f[i] = max(f[i], f[i-1]+val_i)$

- 考虑第二种情况，寻找正数段，则

  $f[i] = max(f[i], f[j]+val_{j+1, i}), val_{j+1, i}>0, j<i$

  由于$val_{j+1, i}>0$，因此$val_{j+1, i} = i - j$，且$f[j]+val_{j, i-1} = f[j] + (i - j) = i + (f[j]-j)$，状态转移方程变为$f[i] = max(f[j]-j)+i$，因此只需要用数据结构维护前面的$f[j]-j$，并支持查询最大值就可以了。

  还有一个问题，我们如何保证$val_{j+1,j}>0$呢，只需要按前缀和进行排序，只查询$\{ j|pre[j] <pre[i] \} $，就可以了。且由于我们顺序插入，也保证了不会查询到后面的$j$。(妙！)

有用的技巧: 

- 用pair去排序
- 排序之后记录pos，然后可以索引到比他小的元素
- 推状态转移方程时$f[j] + val_{j, i-1}=i+(f[j]-j)$，是一个常用技巧和很好的切入点。

### 代码

注意，由于本题的特殊性才可以将树状数组这样用。平时要用线段树

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;
typedef long long LL;
const int INF = 0x7fffffff;
const int MAXN = 500005;

int c[MAXN], n; 
int lowbit(int x) {return x & -x;}
int get_max(int x) {
  int Max = -INF;
  for (; x > 0; x -= lowbit(x)) 
    Max = max(Max, c[x]);
  return Max;
}
void add(int x, int val) {
  for (; x <= n; x += lowbit(x)) 
    c[x] = max(c[x], val);
}

int a[MAXN], pos[MAXN], f[MAXN];
LL pre[MAXN];
vector<pair<LL, int> > v;
int main() {
  int T; scanf("%d", &T);
  while (T--) {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
      scanf("%d", &a[i]);
      pre[i] = pre[i - 1] + a[i];
      v.push_back(make_pair(pre[i], -i));
    }
    
    sort(v.begin(), v.end());    
    for (int i = 1; i <= n; i++)
      pos[-v[i-1].second] = i;

    for (int i = 1; i <= n; i++)
      c[i] = -INF;
    for (int i = 1; i <= n; i++) {
      f[i] = f[i-1] + (a[i]>0? 1: a[i]<0? -1: 0);

      f[i] = max(f[i], get_max(pos[i])+i);
      if (pre[i] > 0) f[i] = i;

      add(pos[i], f[i]-i);
    }
    printf("%d\n", f[n]);

    for (int i = 0; i <= n; i++)
      a[i] = f[i] = pos[i] = pre[i] = c[i] = 0;
    v.clear();
  } 
}
```



