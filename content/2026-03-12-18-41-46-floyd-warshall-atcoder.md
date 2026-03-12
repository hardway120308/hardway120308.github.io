---
title: 背包問題的變體：使用動態規劃求解一個AtCoder問題
description: 本題本質是動態規劃中的0-1背包問題的變體，透過修改狀態轉移方程以及循環的操作方式可以解答
author: 仿生鵜鶘
tags: 動態規劃,演算法,AtCoder
---

# 背包問題的變體：使用動態規劃求解一個AtCoder問題

[問題連結：D - Package Delivery](<[321](https://atcoder.jp/contests/awc0023/tasks/awc0023_d)>)

總共有N個包裹運送請求，給定它們的重量以及利潤，還有載重限制S和最低利潤限制T，求滿足限制的包裹組合中，所接受的請求數最小值。

## 思路

本題是背包問題的翻版。

- 狀態：dp[k][w] 表示選擇k個包裹時，總重量（準確的）為w時的最大利潤。
- 轉移方程：針對每一個物品循環（當前物品利潤val,重量weight），則轉移方程為$ dp[k+1][w] = max(dp[k+1][w],dp[k][w-weight]+val) $
- 初始化：dp[0][0] = 0，其他dp[0][w] = -∞。
- 答案：從1到k遍歷每個選擇以及所有重量，找到第一個滿足總利潤>=T的k值為答案

## 循環的具體實作

偽代碼：

```
for 項目(val,w) in 包裹s:
	for k in N到0:
		for curr_w in S到S-w:
			if (dp[k][curr_w - w] 可到達（透過選擇物品）)
				dp[k+1][curr_w] = max(dp[k+1][curr_w], dp[k][curr_w-w]+val)
```

為什麼要倒序遍歷k：因為如果正序遍歷，會出現先選擇了第一個物品（k=1且W=W_1），則第二次（k=2）時又選擇到同一個物品（W=2W_1）的狀況。倒序遍歷避免了出現這種狀況（如果重量正序遍歷可能沒有影響？答案：是）

## 程式碼

```c++
#include <bits/stdc++.h>
#include <utility>

using namespace std;

int main() {
    const int NEG_INF = -1;
    int n, s, t;
    cin >> n >> s >> t;

    int ans = -1;
    // dp表代表選擇了k個項目，總重量**精確**為w的最大利潤
    int dp[n + 1][s + 1];
    vector<pair<int, int>> items;

    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= s; j++) {
            dp[i][j] = NEG_INF;
        }
    }
    dp[0][0] = 0;

    for (int i = 0; i < n; i++) {
        int p, c, w;
        cin >> p >> c >> w;
        int profit = p - c;
        if (profit > 0 and w <= s) {
            items.push_back(make_pair(w, profit));
        }
    }

    for (auto item : items) {
        int weight = item.first;
        int val = item.second;
        // 倒序避免重複選取
        for (int i = n; i >= 1; i--) {
            // 要運送此請求至少要w的重量，但最多不會超過S
            for (int curr_weight = weight; curr_weight <= s; curr_weight++) {
                // 如果可以透過選擇項目到達
                if (dp[i - 1][curr_weight - weight] != NEG_INF) {
                    // dp表更新為前一個項目中選取本項目或者保留目前的值不變
                    // 要取和現在值max的原因是如果有同重量的物品，則第i=0迴圈進來之後會壓到同樣dp[1][W_1]的值
                    // 此時需要判斷誰的利潤更大才是局部最優解
                    dp[i][curr_weight] = max(dp[i][curr_weight], dp[i - 1][curr_weight - weight] + val);
                }
            }
        }
    }

    for (int k = 0; k <= n; k++) {
        for (int w = 0; w <= s; w++) {
            if (dp[k][w] >= t) {
                cout << k << endl;
                return 0;
            } else if (k == n and w == s) {
                cout << -1 << endl;
                return 0;
            }
        }
    }

    return 0;
}

```
