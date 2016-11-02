---
title: Alternating Digit Sum
layout: post
date: 2016-11-02 12:00:00 +0800
categories: tricks
tags: ["operators"]
---


又是個利用運算子優先順序與型態轉換的奇淫技巧


**題目:** [challenge "AlternatingDigitSum" - CodeFights](https://codefights.com/challenge/uT2ywJR5QtJzJGqMb)

input: n

output: 假設 n! = "abc...xyz", 求 z - x + y ... a modulo 11 的結果

Range: 0 ≤ n ≤ 10<sup>9</sup>

Time limit: 4000ms

---

這題目因為輸入會到10<sup>9</sup>, 其實完全不用考慮在不提前處理的情況下運算n!

不過為了理解整個解題流程, 還是先寫一個方便檢驗用的function

```js
factorial = n => n ? n * factorial(n - 1) : 1

function AlternatingDigitSum(n) {
    var sum = factorial(n)
    var digits = sum.toString(10)
    
    return [...digits].reverse()
                      .reduce((r, val, i) => {
                        // 這邊 +11 因為modulo 11 後需為 0 ~ 10 之間
                        return (r + 11 + (i % 2 ? -1 : 1) * val) % 11
                      }, 0)
}
```

小數值的測試都有符合題目本身提供的答案, 但當然不用submit了, 反正後面保證爆炸, 偏偏看來又不能直接把n先modulo過.

所以寫個快速測試下所有小數值答案看是否有特殊規則:

```js
[...Array(20)].map((v, i) => AlternatingDigitSum(i))
// [1, 1, 2, 6, 2, 10, 5, 2, 5, 1, 10, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

OK, 於是乎發現了這題目的技巧, 就是n超過11之後完全不用理他, 這是因為任何11的倍數, 所有奇數項的總和必定等於偶數項的總和
(很想給它個帥氣的名字, 但我真的查不到, 只知道好像是國中小驗證倍數的技巧之一)

所以當然看來這題最短解有兩個主要方向了, 一個是從原本這function 加上n範圍的限制, 另一個就是直接寫答案.

先嘗試把寫好的function繼續調整, 依照現有答案先去OEIS查詢下,
發現這篇([Final nonzero digit of n! in base 11.](https://oeis.org/A136697))雖然題目不同, 但前面結果一致.

也就是說其實我只需要處理 `n! % 11` 就可以了

原本function 整理成:

```js
// 54 chars
F = n => n ? n * F(n-1) % 11 : 1
AlternatingDigitSum = n => n > 10 ? 0 : F(n)
```

最後把兩個function 整理成一個:

```js
// 47 chars
F = AlternatingDigitSum = n => n % 11 ?
    n * F(n-1) % 11 :
    1 > n | 0
```

暫時就想不到怎麼繼續了.

---

改用直接寫答案來測試下:

```js
// 53 chars
AlternatingDigitSum = n =>  [1, 1, 2, 6, 2, 10, 5, 2, 5, 1, 10][n] | 0
```

即使減少兩個兩位數字, 總字數也還是不變

```js
// 53 chars
AlternatingDigitSum = n =>  [0, 0, 1, 5, 1, 9, 4, 1, 4, 0, 9][n] + 1 | 0
```

可是在這邊所有的數字的變成一位數了，似乎可以改成用字串:

(待續)
