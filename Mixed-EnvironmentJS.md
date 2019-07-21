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

在全域範疇中宣告，無論是否有寫 var，不只會創建一個全域變數也會在 global 物件(window)建立一個同名的特性。

建立帶有 id 屬性的 DOM 元素，也會建立同名的全域變數。

```js
<div id="foo">123</div>
```

還有

```js
if (typeof foo == "undefined") {
  foo = 42; // 永遠部會執行
}

console.log(foo); // HTML元素
```

因此你的 HTML 頁面內容也可能創建全域變數，這也是盡可能避免使用全域變數的原因之一，如果真需要使用，請用名稱獨特不太可能衝突的變數也必須確保部會與 HTML 的內容衝突。

## 原生原型

### 永遠都不要擴充原生的原型!!

無論你想到什麼不存在的方法或特性名稱，想把他加到 Array.prototype 中，只要他是一個實用、設計良好並正確命名的功能，它最後非常有可能會被新增到規格中，如此你自行擴充的功能就與規格產生衝突。

### 不要無條件地定義擴充功能

可能會意外覆寫原生的功能。

## Shims/Polyfills

Shim 指的是在一個舊的環境中模擬出一個新 API ，而且僅靠舊環境中已有的手段實現，以便所有的瀏覽器具有相同的行為。主要特徵：

- 該 API 存在於現代瀏覽器中
- 瀏覽器有各自的 API 或可通過別的 API 實現
- API 的所有方法都被重新實現
- 攔截 API 調用，並提供自己的實現
- 是一個優雅降級

[Polyfill](https://github.com/chenxiaochun/blog/issues/37)的準確意思為：用於實現瀏覽器並不支援的原生 API 的程式碼。提供了那些開發者們希望瀏覽器原生提供支持的功能。程序庫先檢查瀏覽器是否支持某個 API，如果不支持則加載對應的 polyfill。主要特徵：

- 是一個瀏覽器 API 的 Shim
- 與瀏覽器有關
- 沒有提供新的 API，只是在 API 中實現缺少的功能
- 以只需要引入 polyfill ，它會靜靜地工作

> 例如，querySelectorAll 是很多現代瀏覽器都支援的原生 Web API，但是有些古老的瀏覽器並不支援，那麼假設有人寫了庫，只要用了這個庫， 你就可以在古老的瀏覽器裡面使用 document.querySelectorAll，使用方法跟現代瀏覽器原生 API 無異。那麼這個庫就可以稱為 Polyfill 或者 Polyfiller。

shim 的概念要比 polyfill 更大一些，可以將 polyfill 理解為專門兼容瀏覽器 API 的 shim 。

## Script

瀏覽器所檢視的大多數網站或應用程式都有一個以上的檔案，其中包含了他們的程式碼，而常見的情況是其頁面中有數個\<script src=..>\</script>元素負責分別載入 那些檔案，甚至幾個帶有行內程式碼的\<script> .. \</script>元素也很常見。

它們所 共享 的一個東西是一個單獨的 global 物件（在瀏覽器中是 window），這意味著多個檔案可以將他們的程式碼附加到那個共用的命名空間(shared namespace)中藉此互動。

所以，如果一個 script 元素定義了一個全域函式 foo()，當第二個 script 運行時，它就可以取用並呼叫 foo()，就如同自己定義了那個函式一般。

但是全域變數的範疇拉升（scope hosting）的動作不會跨越這些界線發生，所以下面的程式碼並無法運作，無論它們是否是行內的\<script> .. \</script>元素還是外部加載的\<script src=..>\</script>檔案皆是如此：

```js
<script>foo();</script>

<script>
  function foo() { .. }
</script>
```

以下兩者可以運作:

```js
<script>
  foo();
  function foo() { .. }
</script>
```

```js
<script>
  function foo() { .. }
</script>

<script>foo();</script>
```

如果在一個 script 元素中發生了一個錯誤，作為一個分離的獨立的 JS 程序將會失敗並停止，但是任何後續的 script 都將會暢通無阻地運行（具有共享的 global ）。

行內程式碼區塊(inline code block)中的程式碼與在外部檔案中相同的程式碼之間的差異是，在行內程式碼中，字元</script>的序列不能一起出現，因為（無論它在哪裡出現）它將會被翻譯為代碼塊的末尾。所以，小心這樣的代碼：

```js
<script>
  var code = "<script>alert( 'Hello World' )</script>";
</script>
```

它看起來無害，但是在 string 字面量中出現的</script>將會不正常地終結 script 塊，造成一個錯誤。繞過它最常見的一個方法是：

```js
"</sc" + "ript>";
```

此外要注意的是放在一個外部檔案內的程式碼會以該檔案所提供的(或預設的)字元集(character srt, UTF-8、ISO-8859-8 等)來解讀，但同樣的程式碼在一個行內的 script 元素中會以你 HTML 頁面的(或預設的)字元集來解讀。

## 保留字

ES5 語言規範在第 7.6.1 部分中定義了一套“保留字”，它們不能被用作獨立的變量名。技術上講，有四個類別：“關鍵字(keywords)”，“未來保留字(future reserved words)”，null 字面值，以及 true/ false 字面值。

像 function 和 switch 這樣的關鍵字是顯而易見的。像 enum 之類的未來保留字，雖然它們中的許多（class、extends 等等）現在都已經實際被 ES6 使用了；但還有另外一些像 interface 之類的僅在 strict 模式下的保留字。

StackOverflow 用戶“art4theSould”創造性地將這些保留字編成了一首有趣的[小詩](https://stackoverflow.com/questions/26255/reserved-keywords-in-javascript/12114140#12114140)

> 這首詩包含 ES3 中的保留字（byte、long 等等），它們在 ES5 中不再被保留了。

在 ES5 以前，這些保留字同樣不能當作物件字面值中的特性名稱(property names)或鍵值(keys)，但這種限制已經不復存在了。

所以，這是不允許的：

```js
var import = "42";//變數名
```

但這是允許的：

```js
var obj = { import: "42" }; //特性名稱
console.log(obj.import);
```

你應當小心，有些老版本的瀏覽器（主要是舊的 IE）沒有完全地遵循這些規則，所以有些將保留字用作對象屬性名的地方任然會造成問題。小心地測試所有你支持的瀏覽器環境。

## 實作限制

JavaScript 的規格並不會在像是函式的引數數目，或是字串字面值長度這類的事情上設下限制，但儘管如此這些限制仍然存在，它們源自於不同引擎中的實作細節。

例如：

```js
function addAll() {
  var sum = 0;
  for (var i = 0; i < arguments.length; i++) {
    sum += arguments[i];
  }
  return sum;
}

var nums = [];
for (var i = 1; i < 100000; i++) {
  nums.push(i);
}

addAll(2, 4, 6); // 12
addAll.apply(null, nums); // 应该是：499950000
```

在某些 JS 引擎中，你將會得到正確答案 499950000，但在另一些引擎中（比如 Safari 6.x），你會得到一個錯誤：“RangeError: Maximum call stack size exceeded.”

已知存在的其他限制的例子：

- 在字串字面值（string literal 不僅是個字串值）中允許出現的最大字元數
- 可在一個函式呼叫的引數中送出的資料大小(單位為位元組，即堆疊大小 stack size)
- 在一個函數宣告中的參數數量
- 未經最佳化的呼叫堆疊(call stack)之最大深度(即遞迴深度 recursion):一個函式呼叫另一個函式的函式呼叫串鏈可以有多長
- JS 程式可以阻斷(block)瀏覽器持續執行多少秒
- 變量名稱允許的最大長度

  遭遇這些限制不是非常常見，但你應當知道這些限制存在並確實會發生，而且重要的是它們因引擎不同而不同。
