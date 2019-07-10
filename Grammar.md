# Grammar 文法

你或許認為你知道如何撰寫 JS 程式碼，但該語言文法的各個部分都有非常多需要留意的微妙之處，可能會導致混淆或誤解，所以我們要探討並把事情釐清。

**文法是描述其語法(運算子、關鍵字等)如何結合在一起，形成格式正確的有效程式的一種結構化方式**

## 述句與運算式

開發人員常會假設「述句(statement)」「運算式(expression)」這兩個詞大略相同。在 JS 當中它們之間有一些非常重要的差異。

> 用英語的文法來解釋:一個句子是表達一個想法用的字詞完整結構。它由一或更多的「片語」所構成，其中每一個都能以標點符號或連接詞(and、or)來結合。一個片語本身可由更小的片語組成，有些片語是不完整的，無法單獨使用，其他片語則本身就具有意義。

JS 的文法也是如此，述句=>句子、運算式=>片語、運算子=>連接詞或標點符號。
JS 中的每個運算式都能被估算為單一個特定的值作為結果

```js
var a = 3 * 6;
var b = a;
b;
```

`3 * 6`是一個運算式（求值得值 18）。而第二行的`a`也是一個運算式，第三行的`b`也一樣。a 和 b 運算式兩者都會估算為當下儲存在那些變數中的值，剛好也就是 18。

這三行每一行都含有運算式的一個述句。var a = 3* 6 與 var b = a 被稱作是「宣告述句」，因為它們各自都宣告了一個變數(並選擇性地指定了一個值給它)。a = 3* 6 與 b = a 指定 (扣掉那些 var)都被稱作是指定運算式。

第三行只含有一個運算式 b，不過它本身也自成一個述句，因此這通常被稱為一個「運算式述句」。

### 述句完成值

「述句都有完成值」即便那個值只是 undefined。
如何可以看到一個述句的完成值呢?

> 在你瀏覽器 console 的位置打上該述句，當你執行時就會回報最近一次執行的述句完成值。

```js
var a = 3 * 6;
var b = a;
b;

var b = a; //18? undefined?
```

在 ES5 規格中，12.2 的「Variable Statement」中，VariableDeclaration 演算法實際上返回了一個值（是含有所宣告的變數名稱的 string），但是這個值基本上被 VariableStatement 演算法吞掉了（除了在 for..in 循環中使用），而這強制產生一個空的（也就是 undefined）完成值。

簡單的說，var 述句在語言規格裡定義就是會產生一個空的完成值。

事實上你在 console 做過很多程式碼實驗，大概會看到許多不同的述句執行之後都會回報 undefined，或許不清楚那是什麼或為何如此，回報的其實就是述句的完成值。

那要如何捕捉這種完成值?

在討論 HOW 之前先探討一下 WHY 吧!

我們需要考慮其他類型的完成值。舉例來說任何正規的{ .. }區塊都有一個完成值，其值就是它所包含最後一個述句或運算式的值。

```js
var b;

if (true) {
  b = 4 + 38;
}

//42  if區塊的完成值 取自最後一個運算式述句 b = 4 + 38
```

一個區塊的完成值就像是該區塊中最後一個述句隱含的回傳值。
但有一個明顯的問題存在，使下方程式碼無法運作:

```js
var a, b;

a = if (true) {
	b = 4 + 38;
};
```

我們無法用簡單的方式來捕捉一個述句完成值並把它指定給另一個變數。

下方將使用 eval()來捕捉，但這不是一個好方法，僅用來證明演示「述句完成值是真的東西，不僅可以在 console 主控台中看到也可以被捕捉」

```js
var a, b;

a = eval("if (true) { b = 4 + 38; }");

a; // 42
```

ES7 有提出「do expression(do 運算式)」

```js
var a, b;

a = do {
  if (true) {
    b = 4 + 38;
  }
};

a; // 42
```

do{...}運算式執行一個區塊，而該區塊內最後一個述句完成值變成 do 運算式的完成值，而那可被指定給 a。其想法是把述句視為運算式，讓它們能夠出現在其他述句內，不用包裹在一個行內函式運算式，並可進行明確的 return。

### 運算式副作用

大多數的運算式不會有副作用

```js
var a = 2;
var b = a + 3;
```

運算式 a + 3 本身並沒有副作用，例如它不會變更 a。
可能帶有副作用的運算是最常見的例子是函式呼叫運算式(function call expression)

```js
function foo() {
  a = a + 1;
}

var a = 1;
foo(); // 结果：`undefined`，副作用：改变 `a`
```

其他帶有副作用的運算式(++ -- delete =)

```js
var a = 42;
var b = a++;
```

a++運算式帶有兩個獨立的行為:  
1.會回傳 a 目前的值將 42 指定給 b  
2.變更 a 本身的值將之遞增

```js
var a = 42;
var b = a++;

a; // 43
b; // 42
```

許多人會誤以為 b 跟 a 一樣都是 43 這個值，這種混淆在於沒有充分考慮到 **++運算子的副作用在何時產生** 。
++遞增運算子與--遞減運算子都是單運算元運算子，可以前綴(++a)也可以後置(a++)

```js
var a = 42;

a++; // 42
a; // 43

++a; // 44
a; // 44
```

前綴:++a 其副作用會在值從該運算是回傳 **之前** 發生。
後置:a++ 值從該運算是回傳 **之後** 發生。

```js
++a++ ?

//ReferenceError
//對++a++來說會優先執行a++產生42 ++42卻無法執行下去了
//少了變數參考而++無法直接在42這樣的值身上產生作用
```

若是用()來封住 a++，是否可以讓 a++執行後在指定給 b 呢?

```js
var a = 42;
var b = \(a++);

a; // 43
b; // 42

//()本身無法定義一個新的包裹運算式，依然是先回傳42，除非可以在++副作用後重新估算a，否則不會得到43這個值。
```

不過你還有一個選擇!可以使用,述句序列逗號運算子，它可以讓你將多個獨立的運算式述句串成單一個述句:

```js
var a = 42,
  b;
b = (a++, a);

a; // 43
b; // 43

//第二個a代表述句運算式會在a++運算式副作用之後被估算，回傳43給做為要指定給b的值
```

delete 也帶有副作用，其用來移除一個 object 的一個特性或是一個 array 的插槽。通常只被當作一個獨立的述句來呼叫:

```js
var obj = {
  a: 42
};

obj.a; // 42
delete obj.a; // true
obj.a; // undefined
```

如果請求的作業是有效或是被允許的，delete 運算子的結果值就會是就會是 true，否則就是 false。而這個運算子的副作用就是移除指定的特性(或陣列的插槽)。

最後一個帶有副作用的運算子就是 = 指定運算子，其副作用明顯又不明顯。

```js
var a;

a = 42; // 42
a; // 42 將同樣的值指定給a基本上就是一種副作用
```

> 有關於副作用的相同思考方式也是用於複合的指定運算子，像是+=、-=等。舉例來說 a= b+=2 的處理方式是先估算 b += 2 再用 = 指定式的的結果就會被指定給 a。
> 「一個指定運算式(或述句)的結果會成為被指定的值」的這個行為主要用於鏈串的指定式，例如:

```js
var a, b, c;

a = b = c = 42;
```

在此 c = 42 被估算為 42(副作用是指定了 42 給 c)，然後 b = 42 被估算為 42(副作用是指定了 42 給 b )而最後估算 a = 42 (副作用是指定了 42 給 a)

> 鏈串指定式最常犯的錯誤式 var a = b = 42。如果在執行時沒有 var b 來正式宣告 b ，那邊 var a = b = 42 部會直接宣告 b 。若有 strict 模式可能會擲出一個錯誤或是直接宣告一個全域變數。

```js
function vowels(str) {
  var matches;

  if (str) {
    // 取出所有的母音字母
    matches = str.match(/[aeiou]/g);

    if (matches) {
      return matches;
    }
  }
}

vowels("Hello World"); // ["e","o","o"]
```

It's work!!
我們也可以運用指定式的副作用，將兩個 if 述句簡化合併成一個。

```js
function vowels(str) {
  var matches;

  // 取出所有的母音字母
  if (str && (matches = str.match(/[aeiou]/g))) {
    return matches;
  }
}

vowels("Hello World"); // ["e","o","o"]

//(str && (matches = str.match(/[aeiou]/g))) 括號是必要的，因運算子優先序的關係，後續會再提到。
```

## 取決於上下文的規則

在 Javascript 文法規則中「相同的語法在不同地方使用，或是以不同方式使用，會產生不同的意義」這種取決於情境的情況還不少，若單獨考慮就可能導致頗多混淆。
以下僅列幾個常見例子!

### 大括號

程式碼中，{...}大括號會出現的地方主要有兩個:

**物件字面值(object literals)**
首先式做為一個 object 字面值

```js
// 假定有一个函数`bar()`的定义

var a = {
  foo: bar()
};
```

如何知道這是一個 object 呢?因為{...}是被指定給 a 的一個值。

**標籤(labels)**
如果我們從上面程式碼片段中移除 var a = 這個部分，會發生什麼事?

```js
// 假定有一个函数`bar()`的定义

{
  foo: bar();
}
```

許多人可能會認為這{...}只是一個不會被指定到任何地方的獨立 object 字面值，事實上並不是如此，只是一個普通的程式碼區塊!

像這樣單獨存在的{...}區塊在 JS 中並不是很常見，但這是完全有效的 JS 文法，配合著 let 區塊範疇宣告合併使用，會顯得特別有用。

這裡的{...}程式碼區塊在功能上非常類似於皆附在其他述句(例如 for/while 迴圈或是 if 條件式等)上的程式碼區塊。

那 foo: bar() 語法是什麼?為什麼合法?
JS 有一個鮮為人知也不鼓勵使用的方法，「帶了標籤的述句」foo 是述句 bar()的一個標籤，而 JS 支援了標籤跳耀。continue 與 break 述句都能選擇性地接受一個特定標籤而那會使程式流程以跳躍方式進行。

```js
// 用`foo`標籤的迴圈
foo: for (var i = 0; i < 4; i++) {
  for (var j = 0; j < 4; j++) {
    // 這些迴圈相遇時，接續(continue)外層迴圈
    if (j == i) {
      // 跳到帶有`foo`標籤的迴圈的下一代迭代(next iteration)
      continue foo;
    }

    // 跳過奇數乘積
    if ((j * i) % 2 == 1) {
      // 正常(不帶標籤的)接續內層迴圈
      continue;
    }

    console.log(i, j);
  }
}
// 1 0
// 2 0
// 2 1
// 3 0
// 3 2
```

跳過了奇數乘積的 3 1 迭代，帶有標籤的迴圈跳躍也跳過了迭代的 1 1 及 2 2 。標籤迴圈的跳躍對於 break\_搭配，從內層迴圈你想要中斷的地方跳離外層迴圈會比較有用一些。如果沒有帶標籤的 break，同樣羅及有時寫出來可能會有點怪。

```js
// 用`foo`標籤的迴圈
foo: for (var i = 0; i < 4; i++) {
  for (var j = 0; j < 4; j++) {
    if (i * j >= 3) {
      console.log("stopping!", i, j);
      // 跳出被`foo`標記的迴圈
      break foo;
    }

    console.log(i, j);
  }
}
// 0 0
// 0 1
// 0 2
// 0 3
// 1 0
// 1 1
// 1 2
// stopping! 1 3
```

> continue ： 接續執行帶有標籤的迴圈的下次迭代  
> break : 跳離帶有標籤的迴圈或區塊，並接續在其後執行

若上面的 break 不使用標籤，要達到同樣的目的需涉及一個或多個函式、共用的範疇變數存取等等，可能會更加混淆，因此這裡使用帶有標籤的 break 或許是比較好的選擇。

一個標籤可以套用到一個非迴圈的區塊，但只有 break 可以參照這個非迴圈標籤，continue 則無法接續一個非迴圈的標籤也不能不以 break 跳出一個區塊!

```js
function foo() {
  // 用`bar`標籤的區塊
  bar: {
    console.log("Hello");
    break bar; //中斷
    console.log("never runs");
  }
  console.log("World");
}

foo();
// Hello
// World
```

> 但帶有標籤的迴圈以及區塊極其少見，若真要使用帶標籤的跳要記得寫上註解喔!

### 區塊

JS 陷阱 XD

```js
[] + {}; // "[object Object]"
\{} + []; // 0
```

[] + {} 中，[] 會轉為空字串，而 {} 會轉為字串 "[object Object]"。

{} + [] 中，{} 被當成空區塊而無作用， +[] 被當成**強制轉型**為數字 Number([])

（由於陣列是物件，中間會先使用 toString 轉成字空串，導致變成 Number('')）而得到 0。

### 物件解構(object destructuring)

從 ES6 開始，你也會在「解構指定式」看到{...}，特別是 object 的解構(更多資訊在 ES6 與未來發展的單元)

```js
function getData() {
  // ..
  return {
    a: 42,
    b: "foo"
  };
}

var { a, b } = getData();

console.log(a, b); // 42 "foo"

//var { a, b } = getData(); 就是ES6的解構
//可解讀成
//var res = getData();
//var a = res.a;
//var b = res.b;
```

藉由一對{...}來進行的物件解構也能被用在具名的函式引數中，那也是同類型的隱含物件特性指定的語法糖衣!!

```js
function foo({ a, b, c }) {
  // 不再需要這樣做
  // var a = obj.a, b = obj.b, c = obj.c
  console.log(a, b, c);
}

foo({
  c: [1, 2, 3],
  a: 42,
  b: "foo"
}); // 42 "foo" [1, 2, 3]
```

### else if 與選擇性區塊

else if 這樣的語法並不存在!
那這是什麼??

```js
if (a) {
  // ...
} else if (b) {
  // ...
} else {
  // ...
}
```

else if 其實只是因為 if 或 else 後若只接單一述句，就可以省略大括號 {..} 的緣故。

```js
if (a) doSomething(a);

//許多JS風格會堅持你醫定要在單述句區塊周圍使用{}
if (a) {
  doSomething(a);
}
```

因此，上例程式碼其實是這樣的…

```js
if (a) {
  // ...
} else {
  if (b) {
    // ...
  } else {
    // ...
  }
}
```

## 運算子優先序

了解運算子優先序有助於我們理解程式碼什麼時候會執行（短路）、怎麼分批執行（結合性）。

JS 版 &&　以及 || 有趣之處在於它們會選擇並回傳它們其中一個運算元，而非只產生 true 或 false 作為結果。

兩個運算元和一個運算子

```js
var a = 42;
var b = "foo";

a && b; // "foo"
a || b; // 42
```

三個運算元以及兩個運算子

```js
var a = 42;
var b = "foo";
var c = [1, 2, 3];

(a && b) || c; // "foo"
a || (b && c); // 42
```

來回想前面的範例

```js
var a = 42,
  b;
b = (a++, a);

a; // 43
b; // 43
```

如果移除了()呢?

```js
var a = 42,
  b;
(b = a++), a;

a; // 43
b; // 42
```

變更了指定給 b 的值! 因為 , 運算子的優先序比 = 運算子還要低。所以 b = a++, a 會被解讀為(b = a++),a 。如果 a++ 帶有副作用，在++ 變更 a 之前被指定給 b 的值會是 42。

以下的範例:

```js
if (str && (matches = str.match(/[aeiou]/g))) {
  // ..
}
```

指定式周圍的()是必要的，因為 && 優先序比 = 還要高，若沒有 () 來迫使該繫結優先發生，則運算式救會被視為 (str && matches) = str.match..。 但(str && matches)結果不會是個變數，會是一個值(undefined) 無法在一個 = 指定式的左邊。

再來個比較複雜的:

```js
var a = 42;
var b = "foo";
var c = false;

var d = (a && b) || c ? (c || b ? a : c && b) : a;

d; // 42
```

WHY??
第一個部分 （a && b || c） 是像`(a && b) || c`那樣動作，還是像`a && (b || c)`那樣動作？

```js
(false && true) || true; // true
false && (true || true); // false
```

```js
(false && true) || true; // true
(false && true) || true; // true
```

還是左到右處理順序??

```js
true || (false && false); // true

(true || false) && false; // false -- 不
true || (false && false); // true -- winner
```

由此可見 && 運算子會先被估算，其次才是 || 運算子。

[MDN 運算子優先清單](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table)

### 短路

之前提過的 && || 類運算子的短路特質:

- 對 ||（or）來說，若結果為 true，則取第一個運算元為結果；若結果為 false，則取第二個運算元為結果。
- 對 &&（and）來說，若結果為 true，則取第二個運算元為結果；若結果為 false，則取第一個運算元為結果。

因此可應用於:

||（or） 可用來設定變數的初始值。
&&（and）可用來執行「若特定條件成立，才做某件事情」，功能近似 if 述句。
範例如下。

```js
const a = "Hello World!";
const b = 777;
const c = null;

a && c; // 測試 a 為 true，選 c，結果是 null
a && b; // 測試 a 為 true，選 b，結果是 777
undefined && b; // 測試 undefined 為 false，選 undefined，結果是 undefined
a || b; // 測試 a 為 true，選 a，結果是 "Hello World!"
c || "foo"; // 測試 c 為 false，選 'foo'，結果是 "foo"
```

## 較緊密的繫結

讓我們把注意力轉回前面的複雜語句的例子，特別是? :三元操作符的部分。這種? :優先級與&&和||比起來是高還是低？

```js
(a && b) || c ? (c || b ? a : c && b) : a;
```

第一種
`a && b || (c ? c || (b ? a : c) && b : a)`

第二種
`(a && b || c) ? (c || b) ? a : (c && b) : a`

答案是第二種，因為 && 優先序比 || 還要高，而 || 優先序比 ? : 還要高。

### 結合性

一般來說，運算子僅有左結合或右結合，就看 grouping 的動作是從左邊還是右邊。

- 左結合，意即由左至右處理，例如：&&、||。
- 右結合，意即由右至左處理，例如：三元運算子、指定運算子 = 。

> `a && b && c` 這樣的運算式中就會有隱含的規組動作發生。
> 技術上來說會是 `(a && b) && c` 因為 && 是左結合( || 也是左結合)

> `a ? b : c ? d : e;`技術上則會是 `a ? b : (c ? d : e)` 因為三元是右結合。

```js
var a, b, c;

a = b = c = 42;

// 先估算c = 42
```

回到最開始的複雜指定運算式吧!!

```js
var a = 42;
var b = "foo";
var c = false;

var d = (a && b) || c ? (c || b ? a : c && b) : a;

d; // 42
```

規組拆解如下

```js
(a && b) || c ? (c || b ? a : c && b) : a;
```

1. (a && b)是"foo".
2. "foo" || c 是"foo".
3. 對於第一個?測試，"foo"是 truthy。
4. (c || b)是"foo".
5. 對於第二個?測試, "foo"是 truthy。
6. a 是 42.

## 自動分號

自動分號插入（Automatic Semicolon Insertion，ASI）JavaScript 引擎中的剖析器（parser）會在以下情況下，自動幫程式碼補上分號，以避免剖析失敗。

- 換行，即述句結尾處與下一行之間，除了空白和註解外，沒有其他的程式碼。
- break、continue、return、yield 之後。

不需要 ASI 的情況是…

- 區塊（{...}）不需要分號做終結。

範例如下。

```js
const a = 10;

do {
  a--;
} while (a > 1);
```

說明

- a-- 後需要一個分號 ;
- while (a > 1) 後需要一個分號 ;

關於到底要不要加分號這個議題，真的有非常非常多的討論…像是

[Airbnb JavaScript Style Guide: Semicolons](https://github.com/airbnb/javascript#semicolons)

## 錯誤

JS 不僅是具有不同的錯誤子型別(TypeError、ReferenceError、SyntaxError 等等)其文法有定義了幾個編譯時期的錯誤，相對於在執行時期發生的其他所有錯誤。
長久以來存在著數個特定的情況，它們應該被捕捉並回報為「早期錯誤」。一個語法錯誤就是一種早期錯誤(例如 a = , )，其文法也定義了一些語法上有效但儘管如此還是不被允許的東西。

既然你的程式碼尚未開始執行，這些錯誤無法以 try ... catch 來捕抓，而是單純使你程式碼剖析或編譯過程失敗。

以下範例：

```js
var a = /+foo/; // Uncaught SyntaxError: Invalid regular expression: /+foo/: Nothing to repeat

//正規表達式字面值內的語法。這裡的 JS 語法沒有任何錯誤，
//不過那個無效的 regex 會擲出一個早期錯誤。
```

```js
var a;
42 = a;		// Uncaught ReferenceError: Invalid left-hand side in assignment

//指定式的目標必須是一個識別字
//或是會產生一或多個識別字的 ES6 解構運算式，
//所以像是42這樣的值放在那裏是不合法的!
```

```js
function foo(a,b,a) { }		// 沒關係

function bar(a,b,a) { "use strict"; }	// 錯誤！

//ES5 的 strict 模式定義了更多的早期錯誤。像在strict 模式中，函式參數的名稱不可以重複。
```

```js
(function() {
  "use strict";

  var a = {
    b: 42,
    b: 43
  }; // 錯誤
})();

//strict 模式物件字面值不可以帶有多個名稱相同的特性
```

### 過早使用變數

暫時死亡區域（Temporal Dead Zone，TDZ）
ES6 定義了「暫時死亡區域」（Temporal Dead Zone，TDZ），意思是程式碼中某個部份的變數的參考動作還不能執行的地方，這是因為該變數尚未被初始化的緣故。

```js
{
  a = 2; // ReferenceError!
  let a;
}

// 指定式 a = 2 在 a 變數藉由 let a 宣告初始化之前就存取(參考)了它，
//所以位於 a 的TDZ中並擲出一個錯誤
```

之前提到 typeof 對於尚未宣告的變數可有保護機制，但在這裡是無效的。

```js
{
  typeof a; // undefined
  typeof b; // ReferenceError! (TDZ)
  let b;
}
```

## 函式引數

違反 TDZ 的另一個例子預設參數值可以在 ES6 發展中提到。

### try...finally

或許 try...catch 大家都相當熟悉，但是否有想過與之配合的 finally 呢?雖然必要時兩者都可以出現，但其實 try 只需要搭配兩者其一唷!

finally 子句中的程式碼一定會執行，而他永遠都會在 try(與 catch)完成後，在任何其他程式碼執行之前，即刻執行。

那如果一個 try 子句內有一個 return 述句會發生什麼事?顯然會回傳一個值，但接受該值的出較端程式碼是在 finally 之前還是之後執行呢?

```js
function foo() {
  try {
    return 42;
  } finally {
    console.log("Hello");
  }

  console.log("never runs");
}

console.log(foo());
// Hello
// 42
```

1. return 42 會馬上執行，設定 foo()呼叫的完成值。(此時完成 try 子句)

2. finally 子句會即刻緊接著執行。(foo()函式此時才算完成)

3. 它的完成值才會傳回給最後那個 console.log(...) 述句使用。

try 子句中的 throw 也會有完全相同的行為:

```js
function foo() {
  try {
    throw 42;
  } finally {
    console.log("Hello");
  }

  console.log("never runs");
}

console.log(foo());
// Hello
// Uncaught Exception: 42
```

若 finally 子句內被擲出，它就會成為該函式主要的完成值，前面如果有 return 為該函式設定了完成值，則會被丟棄不用。

```js
function foo() {
  try {
    return 42;
  } finally {
    throw "Oops!";
  }

  console.log("never runs");
}

console.log(foo());
// Uncaught Exception: Oops!

//在 finally 中若有return則會覆寫上面的42
```

<!-- try 區塊的內容 vs finally 區塊的內容，到底是誰會先執行？誰會後執行？

先來看第一個例子。

```js
function foo() {
  try {
    return 12345;
  } finally {
    console.log("Hello World");
  }
}

console.log(foo());
//Hello World
//12345
```

再來看第二個例子。

```js
function foo() {
  try {
    console.log("Hello World");
    return 54321;
  } finally {
    return 12345;
  }
}

console.log(foo());
//Hello World
//12345
```

從執行順序來看，的確是先執行 try 區塊，再來才是 finally 區塊，但「述句完成值」會決定結果的「顯示」順序。首先，會先執行區塊的內容，像是 console.log(..)，再來才是執行函式 foo() 回傳完成值。因此，在第一個例子中，會先顯示「Hello World」，再顯示「12345」。而在第二個例子中，的確也是先執行執行區塊的內容 console.log(..)，但由於 finally 若有 return 值，則會覆寫 try 內的回傳值，而成為這個函式最後的完成值，因此得到 12345。 -->

### switch

switch 述句等同於 if-else 的縮寫，依靠 break 來決定是否要持續進行下一個 case 述句，若沒有 break 就會「落穿而過」。

範例如下，這裡有一個檢測庫存的簡易範例，假設目前庫存數量為 50，當庫存為 0 ~ 2 時提示要趕快進貨補庫存，庫存到達 50 時顯示庫存充裕，庫存到達 100 時提示貨品是不是賣不掉，其他狀況都顯示為運作正常。

```Js
const count = 50;

switch(count) {
  case 0:
  case 1:
  case 2:
    console.log('快賣完了！趕快進貨！');
  case 50:
    console.log('庫存充裕');
  case 100:
    console.log('是不是賣不掉了！？');
  default:
    console.log('運作正常');
}
//庫存充裕
//是不是賣不掉了！？
//運作正常
```

這是因為如果沒有加入 break，一旦某個符合條件了，接下來的 case 無論符合與否都會被執行，也就是剛才所提到的「落穿而過」。

加入 break 修正一下。

```Js
const count = 50;

switch(count) {
  case 0:
  case 1:
  case 2:
    console.log('快賣完了！趕快進貨！');
    break;
  case 50:
    console.log('庫存充裕');
    break;
  case 100:
    console.log('是不是賣不掉了！？');
    break;
  default:
    console.log('運作正常');
}

//庫存充裕
```

另外，switch 所做的比對是嚴格相等（===），若希望能使用寬鬆相等（==）而能有強制轉型的功能，就需要改變一下寫法，像是…

```js
var a = "12345";

switch (true) {
  case a == 10:
    console.log("10 or '10'");
    break;
  case a == 12345:
    console.log("12345 or '12345'");
    break;
  default:
  // 不會到達這裡的！
}
//12345 or '12345'
```

最後，default 不一定要放在最後，順序是什麼並不重要喔！

https://www.kancloud.cn/kancloud/you-dont-know-js-types-grammar/516741

https://cythilya.github.io/2018/10/16/grammar/
