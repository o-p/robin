---
title: Add in Range
layout: post
date: 2016-10-14 12:00:00 +0800
categories: tricks
tags: ["operators"]
---


利用各種運算子的題目


**題目:** [challenge "addInRange" - CodeFights](https://codefights.com/challenge/JZs5SSyPfQg55QMYZ)

題目就是從 `參數1` 累加到 `參數2` 的基本題

Range: 從 **-2 * 10<sup>9</sup>** ~ **2 * 10<sup>9</sup>**

Time limit: 4000ms

---

一開始先檢驗最基本的迴圈

```js
// 70 chars
addInRange = (lower, upper) => {
    var n = 0
    while (lower <= upper) n += lower++
    return n
}
```

正確性沒有問題, 但會超過時間限制, 另外目前最短解是31 chars, 所以這題型就直接不考慮用迴圈或遞迴解

那當然剩下的就是直接用公式解了, 這應該也不用查了, 就梯形面積公式調整下而已: `(A + B)(B - A + 1) /2`

```js
// 33 chars
addInRange = (L, U) => (L + U) * (U - L + 1) / 2
```

於是乎, 就忽然陷入瓶頸了...

這麼簡單的公式可是人家最短的解答硬生生少了兩個字

因為這種整數累加必定不會有浮點數, 所以看來 `/ 2` 可以改成 `>> 1`, 但問題是這樣實際上還多了一個字

所以只好先嘗試把這式子乘開合併: 

```js
// 33 chars
addInRange = (L, U) => (U * U - L * L + U + L ) / 2
```

這時發現攤開後的式子反而可以直接套上 `>>1`:

```js
// 32 chars
addInRange = (L, U) => U * U - L * L + U + L >> 1
```
為了讓這式子人比較容易看, 所以整理下, 現在前半段是 `U(U + 1) - L(L - 1)`, 那有沒有辦法把那個`1`給偷掉呢?

用`--`, `++` 不會省到字數, 但如果改用`~` 就剛好會跟正數差1, `e.g. ~1 === -2`

所以這時候把需要多出 +1 的 `U項` 改成 `-U(~U)`, 然後把 `L項` 往前移, 整理成:

```js
// 31 chars
addInRange = (L, U) => L - L * L - U * ~U >> 1
```

達成 31 chars.

完全是在訓練整理數學式跟利用各種運算子還有運算子的優先級...
