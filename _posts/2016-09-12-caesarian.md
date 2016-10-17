---
layout: post
date:   2016-09-12 17:50:00 +0800
categories: tricks
tags: ["string to number", "number to string", "buffer"]
title: 字元與數字互換的奇淫技巧
---

**題目:** [Challenge "caesarian" - CodeFights](https://codefights.com/challenge/Nx88Ei5vnfib8SKD6)

題目很簡單, 就是做個字元位移

> Caesarian shift (Caesar cipher) is a method used in cryptography where a string message is encrypted by shifting the letters n times.
> For more information, see this wiki page.
> You are given an integer n, which can be positive, negative or zero.
> Positive values indicate right shifts, and negative values indicate left shifts.
> Given a message and n, return message encrypted by the shift n.
> 
> ---
> 
> ### Example
> 
> For `message = "abc"` and `n = 3`, the output should be
> `caesarian(message, n) = "def"`.
> 
> "a", shifted to the right 3 times, becomes "d", "b" becomes "e" and "c" becomes "f".
> 
> For `message = "egg"` and `n = -1`, the output should be
> `caesarian(message, n) = "dff"`.
> 
> ---
> 
> ### Input/Output
> 
> **[time limit]** 4000ms (js)
>
> -- 
>
> **[input]** {string} `message`
> The message to be encrypted.
>
> **Constraints:**
> `0 ≤ message.length ≤ 500.`
>
> --
> 
> **[input]** {integer} `n`
> The shift value.
> 
> **Constraints:**
> -2 x 1e9 ≤ n ≤ 2 x 1e9.
> 
> --
> 
> **[output]** {string}
> Encrypted message.

很簡單寫了個 **parseInt** <=> **toString** 的版本

```js
// 158 chars
function caesarian(message, n) {
    // 先將 shift 的字數調整到加減26之間
    n %= 26
    
    const code = [...message]
        // 將文字轉為數字 a-z => 0 - 26
        .map(w => parseInt(w, 36) - 10)
        // shift, 並確保數字往正向發展, 方便處理
        .map(i => (i + n + 26) % 26)
        // 調回 10 - 36
        .map(i => i + 10)
        // 轉回 a - z
        .map(i => i.toString(36)) 

    return code.join('')
}

```

壓縮後, 維持 **parseInt** & **toString** 方法

```js
// 88 chars
caesarian = (M, n) =>
    [...M].reduce((s, m) =>
        s + (
            (parseInt(m, 36) + 16 + n % 26) % 26 + 10
        ).toString(36), '')
```

改用 **String.fromCharCode** & **charCodeAt**

```js
// 85 chars
String.fromCharCode(...[...M].map(v => (v.charCodeAt() + 7 + n % 26) % 26 + 97 ))
```

理論上到這邊一般來說差不多就是最短的解答了,

但是這時候距離最短的解還差了20 chars

而且還有差不多10人都不超過70 chars 

這種差距通常是方法整個用錯...

在比賽結束後特別看了下

原來這類轉換的問題, 在nodejs 上有更奇淫的技巧存在....Buffer

```js
var bufferFromString = Buffer('abc') // <Buffer 61 62 63>

Array.from(bufferFromString) // [ 97, 98, 99 ] => Array<integer>

var bufferFromArray = Buffer([ 100, 101, 102 ]) // <Buffer 64 65 66>

bufferFromArray.toString() // 'def'
```

OK, 所以看來透過Buffer 可以更簡短的 String <=> Array<integer> 互換

那就先來寫個人看的版本:

```js
function caesarian(message, n) {
  var buffer = Buffer(message)
  var shift = n % 26 + 26 // -25 => +1
  var charcodeA = 97 // 'a'.charCodeAt()
  
  var shiftArr = Array.from(buffer)
                      .map(int => // 'a' => 97
                           (
                              int - charcodeA // 97 => 0
                              + shift
                           )
                           % 26 // 27 => 1
                           + charcodeA
                      )
  
  var shiftBuffer = Buffer(shiftArr)
  return shiftBuffer.toString()
}
```

確認所有testers 都可以pass, 開始縮短

在他老母都認不出來他之前, 先記錄下:

```js
// 90 chars
B = Buffer
caesarian = (M, n) => {
  bm = B(M)
  
  S = [...bm].map(i => (i - 97 + n % 26 + 26)
                           % 26
                           + 97
                      )
  
  sb = B(S)
  return '' + sb
}
```

final:

```js
// 65 chars
B = Buffer
caesarian = (M, n) =>
  '' + 
  B(
      [...B(M)].map(
          i => (i + 7 + n % 26) % 26 + 97
      )
  )
```

以上, 就是nodejs only 的string <=> int 奇淫技巧
