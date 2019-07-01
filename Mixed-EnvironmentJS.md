# 混合環境的 Javascript

`JS 幾乎都處於一個 hosting environment(宿主環境)中，會因為上下文的內容而造成不可預期的影響。`
舉例來說:當你程式碼與其他來源的程式碼一起執行時，或是當你的程式碼在不同類型的 JS 引擎(不僅限瀏覽器)中執行時，有些行為可能會變得不同。

### 官方 ECMAScript

[ECMAScript 官網](http://www.ecma-international.org/)  
[歷史演進](https://zh.wikipedia.org/wiki/ECMAScript)

ECMAScript 是由網景的布蘭登·艾克開發的一種手稿語言的標準化規範；最初命名為 Mocha，後來改名為 LiveScript，最後重新命名為 JavaScript[1]。1995 年 12 月，昇陽與網景聯合發表了 JavaScript[2]。1996 年 11 月，網景公司將 JavaScript 提交給歐洲電腦製造商協會進行標準化。ECMA-262 的第一個版本於 1997 年 6 月被 Ecma 組織採納。`ECMAScript是由ECMA-262標準化的手稿語言的名稱。`

**附加條款 B** -為了瀏覽器中 JS 的相容性而偏離官方規格的特定情況。
若都在瀏覽器中執行則不會發現任何問題，如非如此(例如 node.js、Rhino)或在你不確定的環境中，則需要小心。

- 八進位數字字面量是允許的，比如在非 strict mode 下的 0123（小數 83）。
- `已廢棄` window.escape(..)和 window.unescape(..)允許你使用%分割的十六進制轉義序列來轉義或非轉義字符串。例如：window.escape( "?foo=97%&bar=3%" )產生"%3Ffoo%3D97%25%26bar%3D3%25"
- String.prototype.substr 與 String.prototype.substring 十分相似，除了第二個參數是 length（要包含的字符數），而非結束（不含）的索引。`String.prototype.substr 已不建議使用`

---

- hh
- kkk

```js
```
