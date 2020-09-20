---
title: UVALive-8291 Pavers
categories: Solutions
tags:
  - dp
date: 2018-11-15 23:03:18
comments: false
---

### 出處
[UVALive-8291](https://icpcarchive.ecs.baylor.edu/external/82/8291.pdf)

### 題意

給 n 問 2 \* n 的矩形能被以下幾種形狀組合而成的方法數
以及所有方法用到的 1 \* 1、1 \* 2、L 型方塊分別加總
![](https://i.imgur.com/VfgCw5S.png)

#### Sample Input
```
2
1 1
2 2
```

#### Sample Output
```
1 2 2 1 0
2 11 16 8 4
```

### 思路

#### The Rod-cutting Problem

這題我是用 [The Rod-cutting Problem](https://www.roading.org/algorithm/introductiontoalgorithm/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E7%AC%94%E8%AE%B0%EF%BC%881%EF%BC%89rod-cutting.html) 來想的，剛好最近演算法上課有教到(((o(*ﾟ▽ﾟ*)o)))
將 2 \* n 用 2 \* i 去切，其中 i = 1, 2, ..., n，而且 2 \* i 是不能垂直分割的
看 2 \* n 有幾種切的方式，再針對每種切法將每個切下來的 2 * i 的組合數乘起來
所以就會有以下的 dp 式子（可以在完整的 code 裡面找到影子，但是算組合還要加一點東西）
``` cpp
for (int i = 1; i < n; i ++)
    dp(n) += k * dp(n - i); // k is a constant
```

#### 2 * i 不可垂直分割的組合數

當 i = 1 的時候很明顯只有以下兩種
![](https://i.imgur.com/m6JZ6kU.png)

i = 2 的時候有以下用螢光筆標起來的 7 種
沒標的因為可以從中間切下去變成 2 \* 1 的所以不算
![](https://i.imgur.com/LGfEIyl.png)

i = 3 的時候是 8 種
![](https://i.imgur.com/qphyZom.png)

i = 4 的時候也是 8 種
![](https://i.imgur.com/RF3O2DW.png)

一般化之後會發現只有 i = 1, 2 的時候是特例
當 i = k, k > 2 的時候組合數都是 8
只有調皮的橫向 2 \* 1 放中間才會形成不能垂直切的情況
然後考慮1 \* 1, L 型在四個角落的排列組合就好
接著觀察一下會發現 1 \* 1 跟 L 型都各有 8 個，橫向 2 \* 1 的會有 8 \* (k - 2) 個

#### 實作

``` cpp
struct obj {
    LL x = 0, a = 0, b = 0, c = 0;
};
vector<obj> dp(MAX);
```
obj 裡面的 x 是指組合數，a 是 1 \* 1 的方塊數，b 是 1 \* 2，c 是 L 型
dp[k] 就是 2 \* k 的答案

``` cpp
tmp = solve(n - i);
dp[n].x += 8 * tmp.x;
```
拿 i > 2 的狀況舉例
dp[n] 的組合數就是 2 \* i 的組合數 8 乘上 dp[n - i] 的組合數

``` cpp
dp[n].a += 8 * tmp.x + 8 * tmp.a;
dp[n].b += ((i - 2) * 8) * tmp.x + 8 * tmp.b;
dp[n].c += 8 * tmp.x + 8 * tmp.c;
```
a 方塊的數量就是 (dp[i] 中 a 數量) \* (dp[n - i] 的組合數) + (dp[n - i] 中 a 數量) \* (dp[i] 的組合數)
b 和 c 以此類推

### Code

``` cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
#define MAX 10000

struct obj {
    LL x = 0, a = 0, b = 0, c = 0;
};
vector<obj> dp(MAX);

obj solve(int n) {

    if (dp[n].x) return dp[n];
    
    // i = 1
    obj tmp = solve(n - 1);
    dp[n].x += 2 * tmp.x;
    dp[n].a += 2 * tmp.x + 2 * tmp.a;
    dp[n].b += 1 * tmp.x + 2 * tmp.b;
    dp[n].c += 0 * tmp.x + 2 * tmp.c;

    // i = 2
    tmp = solve(n - 2);
    dp[n].x += 7 * tmp.x;
    dp[n].a += 8 * tmp.x + 7 * tmp.a;
    dp[n].b += 4 * tmp.x + 7 * tmp.b;
    dp[n].c += 4 * tmp.x + 7 * tmp.c;
    
    // i > 2
    for (int i = 3; i < n; i ++) {
        tmp = solve(n - i);
        dp[n].x += 8 * tmp.x;
        dp[n].a += 8 * tmp.x + 8 * tmp.a;
        dp[n].b += ((i - 2) * 8) * tmp.x + 8 * tmp.b;
        dp[n].c += 8 * tmp.x + 8 * tmp.c;
    }
    
    // i = n
    dp[n].x += 8;
    dp[n].a += 8;
    dp[n].b += (n - 2) * 8;
    dp[n].c += 8;
    
    return dp[n];
}

int main() {
    ios_base::sync_with_stdio(false); cin.tie(0);
    
    dp[0].x = 1;  dp[0].a = 0;  dp[0].b = 0; dp[0].c = 0;
    dp[1].x = 2;  dp[1].a = 2;  dp[1].b = 1; dp[1].c = 0;
    dp[2].x = 11; dp[2].a = 16; dp[2].b = 8; dp[2].c = 4;
    
    int T, num, n; cin >> T; for(int t = 1; t <= T; t ++) {
        cin >> num >> n;
        obj ans = solve(n);
        cout << t << ' ' << ans.x << ' ' << ans.a << ' ' << ans.b << ' ' << ans.c  << '\n';
    }
}
```

### 碎碎唸

因為最近被提醒了某堂課需要寫題解跟出題
所以就順便架了一直滿想架架看的部落格，覺得很好玩ww
打算先試著寫寫看題解再來挑戰出題的部分
就從平常社團練習時間難得解出來的題目開始
不過因為隊友實作能力真的很棒所以通常都輪不到我ˊˇˋ
所以應該會少少的ˊˇˋ

寫題解果然也需要練習，總覺得有很多詞不達意的地方

這題後來有偷偷去跟電神問了他的做法
是模擬不同形狀從以下三種狀態拔出的方法數 dp 累加
![](https://i.imgur.com/t3PVtyg.png)
我也有試著推過可是在排容的時候整個卡住了ˊˋ
而且看起來要很強的實作能力
但我覺得他的方法很直覺
我的好曲折想到的時候別人都寫好幾題了（當然也有可能是我想太慢XD

不管怎麼樣，寫出來很開心
部落格架好也很開心
然後演算法跟 SA 作業延期也很開心
今天真是開心的一天ww
