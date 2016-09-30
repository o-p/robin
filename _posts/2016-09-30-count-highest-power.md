---
title: Count Highest Power
layout: post
date: 2016-09-30 11:30:00 +0800
categories: tricks
tags: ["Hamming weight", "population count"]
---


難以想標題的一篇, 技巧很瑣碎


**題目:** [challenge "countHighestPower" - CodeFights](https://codefights.com/challenge/HzPehmvMe6ySL5Zy9)

題目要求的是從1 到 N 之間所有數字, 每個數字各自最大的2<sup>n</sup>因數, n的加總.

>
> Let `H(N)` be the **highest power of 2** that divides `N`.
> For example, `H(1) = 0`, since the highest power of 2 that divides `1` is **2<sup>0</sup>**,
> and `H(6) = 1`, since **2<sup>1</sup>** divides `6`, but **2<sup>2</sup>** does not.
> 
> For the given `N`, find the sum of `H(K)` for all `K` in range `[1, N]`.
>
> ---
>
> ### Example
>
> For `N = "1"`, the output should be
> `countHighestPower(N) = "0"`.
> 
> The sum consists of a single element `H(1) = 0`.
> 
> For `N = "10"`, the output should be
> `countHighestPower(N) = "8"`.
> 
> The answer equals `H(1) + H(2) + H(3) + ... + H(10) = 0 + 1 + 0 + 2 + 0 + 1 + 0 + 3 + 0 + 1 = 8`.
>
> ---
>
> ### Input/Output
>
> - **[time limit]** 4000ms (js)
>
> - **[input]** string N
> 
>     The number given as a string.
> 
>     Constraints:
>
>     1 ≤ N ≤ 10<sup>12</sup>.
> 
> - **[output]** string
>

---

因為所有單數必定是 2<sup>0</sup> (直接不計入sum), 所以基本上只需要處理雙數, 而且只要找到最高的一個, 剩下的就不用理他.

依照到這邊的想法, 這時候開始實作的話, 可能步驟會是, 先找出 N 最接近的2<sup>n</sup> => 透過log2 或 toString(2), 然後每個數字依次往下測試.

但這種從 1 ~ N 之間有多少個符合xxx規則的題目, 通常都有更有效率的找法, 就是反過來直接套規則找數字:

假設N是1026:

  - 2<sup>10</sup> - 最接近的2<sup>n</sup>, 只可能出現一次 - sum + 10
  - 2<sup>9</sup> - 看來也只能出現一次, 因為 2 * 2<sup>9</sup> 就變成 2<sup>10</sup> - sum + 9
  - 2<sup>8</sup> - 1x2<sup>8</sup>, <strike>2x2<sup>8</sup></strike>, 3x2<sup>8</sup>, <strike>4x2<sup>8</sup></strike> - sum + 16
  
到這邊差不多可以看出規則, n 從 1 ~ K, 每個都只需要檢查單數項, 因為雙數項勢必存在更高一階的n
  
到這邊開始實作的話, 基本上就是從n = 1 開始往上找, 直到 2<sup>n</sup> > N

```js
// 113 chars
function countHighestPower(N) {
    var sum = 0
        n = 1,
        c = 2,
        a = N / c;

    for (; a >= 1; ++n, c *= 2, a = N/c) {
        sum += n * Math.round(a / 2)
    }
    
    return sum + ''
}
```
依照上面的code, 估計還可以縮個10來字, 不過跟第一名 42 chars落差太大, 決定找現成的論文

---

用寫好的function 帶入 1 ~ 10的結果 `[0, 1, 1, 3, 3, 4, 4, 7, 7, 8]`, 找到這篇: [a(n) = n minus (number of 1's in binary expansion of n). Also highest power of 2 dividing n!.](https://oeis.org/A011371)

裡面提到幾個主要方法:

1. ```a(n) = a([n/2]) + [n/2] = [n/2] + [n/4] + [n/8] + [n/16] + ...```, 測試ok
2. ```A(x) = 1/(1 - x)*Sum(k = 1, infinity, x^(2^k)/(1 - x^(2^k)))``` - 看來實作困難, 還有無限大, 直接放棄
3. ```a(n) = n - A000120(n)``` => A000120: binary(n) 中 1 的總量, 確認可用
4. ```a(n) = A005187(n) - n, n >= 0.``` => A005187: ```a(n) = a(floor(n/2)) + n``` - 看來就是**方法1** 的進化版

**方法1** 的測試:

```js
// 71 chars
F = N => N ? Math.floor(N / 2) + F(Math.floor(N/2)) : 0
countHighestPower = N => F(N) + ''
```

**方法3** 的測試:

```js
// 62 chars
countHighestPower = N => N - ((+N).toString(2).split(1).length - 1) + ''
```

**方法4** 的測試:

```js
// 60 chars
F = N => N ? F(Math.floor(N / 2)) + N : 0
countHighestPower = N => F(+N) - N + ''
```

---

照查出來的方法, 應該就是**方法3** 跟**方法4** 拿來調整.

**方法3** 關鍵點應該在於能用多短的方法解出N的[Hamming weight](https://oeis.org/A000120), 也就是二進位表示法中1的總量.

而且目前看來似乎沒有一個非常簡短的方程式可以直接算出, 但針對這方法, 在最後submit的答案中看到一些很有趣的作法:

```js
// 63 chars
countHighestPower = n => 
    n-eval([...(+n).toString(2)].join("+"))+""
```
這作法很有趣的組裝`eval`, 雖然最後沒有縮短字數, 不過對於從一開始學js就避免使用`eval`的我來說覺得充滿想像空間~~~~

```js
// 54 chars
countHighestPower = n => {
    for (m = n; m; m /= 2)
        n -= m & 1
    return n + ""   
}
```

到這看來就是**方法3** 的極限了, 目前幾乎所有X進位計算Y字數這類的題目,
最短解幾乎都不是透過將字串轉成array後使用split, join, reduce等方法來實現,
這也顯示出JavaScript String 方法的貧乏.

---

這題最後是透過**方法4** 整合成單一的遞迴來實現最短解, 要整合成單一的遞迴, 主要要解決的問題有:

- for n = 0 => return 任何作加起來會成為0的值
- for n > 0 => return string solution
- 不能用 `A | 0`, `~~A` 之類的方法來無條件捨去, 因為會超過32bit, 要套用`Math.floor`或自己寫

先把**方法4** 的兩個方程式合併:

```js
// 52 chars
f = countHighestPower = N => 
 // N && +f(Math.floor(N / 2)) + Math.floor(N / 2)
    N && +f(N = Math.floor(N / 2)) + N + ''
```

到這邊答案正確也超越**方法3** 了, 但為了轉型態跟無條件捨去, 花了很多字數.

另外可以發現, 因為方程式前後都是將參數floor後計算, 也就是說其實參數是浮點數是沒差的, 改成 `f(N /= 2)`, 讓後面+N的部分來處理floor.

前面有提到, 這邊無法用`(N | 0)`來捨去, 所以這邊改用: `N - N % 1`

變成:

```js
// 43 chars
f = countHighestPower = N => 
    N && +f(N /= 2) + N - N % 1 + ''
```

寫在一起後發現跑出來減號, 非常好, 因為除了加法外, 都會轉換成數值來處理, 所以連型態轉換都省了, 整理成:

```js
// 42 chars
f = countHighestPower = N => 
    N && f(N /= 2) - N % 1 + N + ''
```

