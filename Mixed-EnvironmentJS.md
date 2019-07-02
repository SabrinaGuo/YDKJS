# 混合環境的 Javascript

`JS 幾乎都處於一個 hosting environment(宿主環境)中，會因為上下文的內容而造成不可預期的影響。`
舉例來說:當你程式碼與其他來源的程式碼一起執行時，或是當你的程式碼在不同類型的 JS 引擎(不僅限瀏覽器)中執行時，有些行為可能會變得不同。

## 官方 ECMAScript

[ECMAScript 官網](http://www.ecma-international.org/)  
[歷史演進](https://zh.wikipedia.org/wiki/ECMAScript)

ECMAScript 是由網景的布蘭登·艾克開發的一種手稿語言的標準化規範；最初命名為 Mocha，後來改名為 LiveScript，最後重新命名為 JavaScript[1]。1995 年 12 月，昇陽與網景聯合發表了 JavaScript[2]。1996 年 11 月，網景公司將 JavaScript 提交給歐洲電腦製造商協會進行標準化。ECMA-262 的第一個版本於 1997 年 6 月被 Ecma 組織採納。`ECMAScript是由ECMA-262標準化的手稿語言的名稱。`

**附加條款 B** -為了瀏覽器中 JS 的相容性而偏離官方規格的特定情況。
若都在瀏覽器中執行則不會發現任何問題，如非如此(例如 node.js、Rhino)或在你不確定的環境中，則需要小心。

- 八進位數字字面量是允許的，比如在非 strict mode 下的 0123（十進位 83）。
- `已廢棄` window.escape(..)和 window.unescape(..)允許你使用%分割的十六進制轉義序列來轉義或非轉義字符串。例如：window.escape( "?foo=97%&bar=3%" )產生"%3Ffoo%3D97%25%26bar%3D3%25"
- String.prototype.substr 與 String.prototype.substring 十分相似，除了第二個參數是 length（要包含的字符數），而非結束（不含）的索引。`String.prototype.substr 已不建議使用`

**Web ECMAScript**

涵蓋官方的 ECMAScript 規格與目前瀏覽器中的 Javascript 實作的差異。
換句話說，這些項目是瀏覽器「要求」但並未列入官方規格的 Annex B。

- \<!--和-->是合法的單行註釋分割符。
- String.prototype 擁有返回 HTML 格式化字符串的附加方法：anchor(..)、big(..)、blink(..)、bold(..)、fixed(..)、fontcolor(..)、fontsize(..)、italics(..)、link(..)、small(..)、strike(..)、和 sub(..)。

`注意：它們在實際應用中非常罕見，而且一般來說不鼓勵使用，而是用其他內建 DOM API 或用戶定義的工具取代。`

- RegExp 擴展：RegExp.$1 ..  RegExp.$9（匹配組）和 RegExp.lastMatch/ RegExp["$&"]（最近的匹配）。
- Function.prototype 附加功能：Function.prototype.arguments（內部 arguments 對象的別名）和 Function.caller（內部 arguments.caller 的別名）。

`注意： arguments 和 arguments.caller 都被廢棄了，所以你應當盡可能避免使用它們。這些別名更是這樣——不要使用它們！`

[Annex B](https://es5.javascript.ru/B.html)  
[Web ECMAScript](https://github.com/tc39/ecma262/labels/web%20reality)

## Host 物件

對於那些自動定義或由 host 環境(例如瀏覽器)創建並提供給 JS 的變數而言，規範 JS 中變數行為的那些定義完備的規則可能有例外，這些就是所謂的「host 物件」
(host objects 其中包含了內建的內建的 object 與 function)

```js
var a = document.createElement("div");

typeof a; // "object" -- 意料之中的
Object.prototype.toString.call(a); // "[object HTMLDivElement]"

a.tagName; // "DIV"
```

a 不僅是一個 object，而且是一個特殊的 host 物件，因為它是一個 DOM 元素。它擁有一個不同的內部[[Class]]值（"HTMLDivElement"），而且帶有預定義的（而且通常是不可更改的）屬性。

另一個已經在前幾章的“Falsy 對象”一節中探討過的同樣的怪異之處是：有些物件被強制轉換為 boolean 時，它們將（令人困惑地）被轉換為 false 而不是預期的 true。

另一些需要注意的 host 物件特殊行為：

- 無法取用一般 object 的內建功能，例如 toString()
- 無法被覆寫
- 擁有特定的預先定義的唯獨特性
- 具有的方法之 this 無法被覆寫為其他物件
- 其他……

對其行為的假設也需要小心，因為他們常與正規的 JS object 有所不同。
較常互動的 host 物件是 console 物件與其的各個函式(log(..)、error(..)等)。這個 console 物件是由 hosting 環境所特別提供，程式碼才能與之互動，進行各種與開發工作有關的輸出作業。

## 全域的 DOM 變數

https://www.kancloud.cn/kancloud/you-dont-know-js-types-grammar/516742

https://yakimhsu.com/javascript/JS_Daquan-04.html

---

- hh
- kkk

```js
```
