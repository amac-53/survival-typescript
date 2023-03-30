# 値・型・変数 (3/24)

## 変数宣言: letとconst
- `let`は初期値がない場合，`undefined`
- `const`は再代入不可能な変数名を宣言するだけで，ミュータブルなオブジェクトを不変にする保護効果はない
- **完全に不変なオブジェクトにするには，プロパティを readonly にする必要がある**
- 配列もオブジェクトの一種なので，変数自体への再代入はできないが，`push`, `pop`は可能（ミュータブル）

## 変数宣言の型注釈 (type annotation)
- 型アノテーションともいい，変数名の右側に書く

## varはもう使わない
- var は再宣言まで可能なので意図しない結果を生む可能性が大いにあり，使うべきではない
- **グローバル変数として定義したときに，window オブジェクトのプロパティとして定義されるため，既存のプロパティを上書きすることも大いにあり，危険**
```typescript
var innerWidth = 10;
console.log(window.innerWidth); // 10
```
- JS ではスコープの先頭で変数が生成される（変数の巻き上げという）
- **var は巻き上げの段階で undefined で初期化されるため，定義する前に参照できるが，let, const は変数の評価のタイミングまで初期化されないため，Reference Error が起こる**
- **var は関数スコープ，let, const はブロックスコープなので注意**

## 変数宣言の型推論
- **型アノテーションを付けない場合，コンパイラによる自動推論が行われる（TS では`let x = 1; x = "hello"`はコンパイルエラーとなる）**

## 型推論と動的型付けの違い
- 動的型付けは実行時，静的型付けはコンパイル時に型が決定される
- 型推論もコンパイルのタイミングで型が決定されるため，扱い的には静的型付け
- `let x = 1; x = "hello";`は TS のコンパイラは通らないが，JS は動的型付けを行うので，上記のコードもそのまま通る

======================================
## プリミティブ型 (primitive types)
- JS はプリミティブとオブジェクトの２つに分類
- **プリミティブは「イミュータブル」かつ「プロパティをもたない」が基本特性（string, number がプロパティを持つのは JS の特徴で，プリミティブからオブジェクトに変換する機能である自動ボックス化（autoboxing）が備わっているため）**
- **プリミティブは７つ（boolean, number, string, undefined, null, symbol, bigint）**
- boolean の他に Boolean，number の他に Number もあり

### 論理型 (boolean type)
- `const isOk: boolean = true;`

### 数値型 (number type)
- **小数点で始めたり，終わらせたりも可能（`0.1 === .1, 5.0 === 5.`）**
- 区切り文字としてアンダースコアも使用可能 （`100_000_000`みたいな）（python でも見かける）
- プロパティとして`toString()`を使用する場合，（小数点としてのドットとアクセッサーとしてのドットを区別できないため，書き方を工夫する必要がある）（`(5).toString() or 5..toString()`）
- 64bit のうち52bit が整数値，11bit が小数値, 1bit が符号（-(2^53 - 1) ~ 2^53 - 1まで表現可能）（10^16あたりまで表現可能）
- 特別な値として，`NaN`, `Infinity`がある
- `NaN == NaN`（or `NaN === NaN`）は常に false を返すので注意
- **`NaN`であるか否かは`Number.isNaN(値)`で確認する**
```typescript
const price = parseInt("百円");
if(Number.isNaN(price)){
  console.log("数値化できません");
}
```
- **小数計算の誤差も当然起こり得るので，できるだけ整数値で計算するようにしたり，包括的な解決として`decimal.js`を用いる工夫をする**（`0.1 + 0.2 == 0.3`は false）

### 文字列型 (string type)
- **テンプレートリテラルのみ文字列途中で改行できる**
```typescript
console.log(`改行
してみる`)
```
- **best practice として "" > '' > ``（文字列展開が必要な場合）を推奨**
- 文字列結合演算子（+）で結合する

### null型
- **null リテラルは null（`const x: null = null;`）（null の注釈には`null`を使用する）**
- `console.log(typeof null); // "object"`

### undefined型
- 値のない変数，戻り値なし関数，存在しないプロパティにアクセスしたときに返される値（`const x: undefined = undefined;`）
- **リテラルを持たず，`undefined`は変数（具体的な値はない）**
- 型注釈は undefined でいいが，戻り値なし関数は void を使用する
```typescript
typeof undefined; // undefined
```

### undefinedとnullの違い
- **undefined は変数，null はリテラル（数値などと同じ具体的な値といったイメージ）なので，null という変数は作成できないが，undefined という変数を作成することは可能**
- 意味的な違いで言うと，undefined は「代入されていないため値がない」，null は「代入すべき値が存在しないため値がない」という意味
- JSON化すると，undefined は削除される一方で null は残る（`console.log(JSON.stringify({ foo: null }) // {"foo": null}`）
- **こだわりがなければ，undefined のみを使うというルールに統一すべき**
  - null は自然発生しないので，こっちに合わせるには変換コードが大量発生して面倒
  - null を返す API を使うなら，変換してしまう or （変換コードが多くなるなら）その部分だけ null を許容するという案でもいい

### シンボル型 (symbol type)
- number or string が初期化した場所まで同じでないと区別されるような型（`const s: symbol = Symbol(5);`）
- JS の新たな API 開発時に下位互換性を保つために導入されたらしいので，使用場面はあまりない

### bigint型
- **number よりも大きい整数を扱える型．初期化方法は大きく２通りで，「リテラルを用いる」（`const x: bigint = 100n`）or「Bigint関数を用いる」 (`const x = BigInt("100")　// or 100`)**
- TS で用いるには**target es2020以上にする必要がある**
- 計算には型を合わせる必要がある（`const i = 2n + BigInt(3) // 2n + 3 はできない`）

## 型強制 (type coercion)
- 暗黙の型変換の元，計算が行われることを指す（`"1" - 1 // => 0` or `"1" + 1 // => "11"`）

## ボックス化 (boxing)
- メソッドやフィールドを扱うために，プリミティブをオブジェクトに変換する操作
- null と undefined 以外は，**ラッパーオブジェクトを通すことで扱える**が，暗黙的にボックス化してくれているので明示的に変換する必要はない（**自動ボックス化**）
```typescript
const str = "abc";
const strObj = new String(str);  // ここは暗黙的に行われる
strObj.length;  // フィールド
strObj.toUpperCase();  // メソッド
```
- 型はプリミティブの先頭を大文字にした形で定義されているが，プリミティブ => ラッパーオブジェクトは可能でも逆は基本出来ない
- ラッパーオブジェクト型は演算子も使えない（`const num: Number = 1; num * 2; // これは不可能`）
- **ラッパーオブジェクトで型注釈を行うメリットはないのでプリミティブで書くようにすること**

## リテラル型 (literal type)
- 特定の値だけを代入可能にしたもの（`const isTrue: true = true; // false は 使用不可`）
- **マジックナンバー（ハードコーディングされた値）やステート（状態遷移を扱うとき）に使用され，ユニオンと組み合わせることが多い（`let num: 1 | 2 | 3 = 1;`）**

## any型
- どんな型でも ok になるやつ（これを宣言されるとコンパイラが型チェックをしなくなるので基本は避けるべき）（`const str: any = 123; str.toLowerCase() //これもコンパイルエラーが出なくなり，実行時エラーになる`）
- コンテキストから予測できないとき（関数の引数など）は，暗黙的に any となる
- **`tsconfig.json`で`noImplicitAny: true`とすることで any 型を許容しないことも可能**（必ずしも any が悪とは限らないのでチームごとに決定）
- コンパイルエラーに詰まったり，既存の JS プロジェクトに TS を導入するときには，まずは動くものを最優先で作るために any を許容するなどの「**がんばらない TypeScript**」というアプローチを採用することも


==================================
## オブジェクト
### プリミティブ以外はすべてオブジェクト
- **プリミティブは値が同じだと同一だが，オブジェクトはプロパティが同じであってもインスタンスが異なると同一でない**

### オブジェクトリテラル (object literal)
- オブジェクトリテラル（`{}`）と`new`による生成があるが，前者を用いるのが普通
- JSON はそのままオブジェクトリテラルとして解釈可能

### オブジェクトのプロパティ
- プロパティ（値とキーの対）の集合体で，値にはプリミティブ，関数，オブジェクトも代入可能
- メソッドはキーと関数名に分けずに，関数と同様に定義可能になっている
- **JS ではメソッドとフィールドの区別が曖昧なため，メソッドに null を代入することでフィールドに変えてしまうことも可能**
```typescript
const calculator = {
  sum(a, b) {
    return a + b;
  }
}

calculator.sum(1, 1);  // 2
calculator.sum = null;
calculator.sum(1, 1) // フィールドに変換されたので呼び出せなくなる
```

### オブジェクトの型注釈 (type annotation)
- **型エイリアスとインライン型注釈がある**（プロパティの区切りは「,」でもいいが，prettier が「;」に変更するのでこっちを最初から採用しておくとよい）
```typescript
// インライン注釈
let box: {width: number; height: number;};
box = {width: 1080, height: 720};

// 型エイリアス
type Box = {width: number; height: number;};
let box: Box = {width: 1080, height: 720}; 

// メソッド
let calculator: {
  sum(x: number, y: number): number;
};
// or 関数構文の書き方
let calculator: {
  sum(x: number, y:number) => number;
};

// Record
let foo: Record<string, number>;
foo = { a: 1, b: 2 };
``` 
- 型推論も使えるが，**メソッドの場合は引数の型注釈は必要**
- **`strictFunctionTypes`を有効にした場合，引数のチェックが双変から共変へと厳格になる点に注意**
- 連想配列のようなオブジェクトを定義するときには，`Record`を使用可能
- `object`型を型注釈に用いることも可能だが，プロパティの情報がなくコンパイルエラーになる上，期待しないオブジェクトも代入できてしまうのでおすすめしない

### オブジェクト型のreadonlyプロパティ (readonly property)
- プロパティを読み取り専用にしたい場合は，readonly を先頭につける
- 再帰的ではないので，入れ子内部も読み取り専用にしたいなら readonly をつける必要がある
- **readonly は TS の概念なので実行時には`readonly`が消された JS コードが生成されるため，エラーにならない（つまりエラーを無視してコンパイルしてしまえばそのコードは通ることになってしまうが，きちんと修正しておけば問題はない）**
- プロパティが多くなった時にはユーティリティ型の`Readonly<T>`を使用する（`let obj = Readonly<{ a: number; b: number: ,...}>`）

### readonlyとconstの違い
- **`const`はオブジェクトのプロパティへの再代入は許可されているが，`readonly`はプロパティへの代入を禁止することができる（つまり const と readonly を使うことで完全に不変不能なオブジェクトを作成可能）**

### オブジェクト型のオプションプロパティ (optional property)
- オプショナルを型付けするには，プロパティ名の後ろに`?`を書く（`let size: {width? : number};`）
- **もちろんプロパティを持たなくてもいいので，空オブジェクト（size = {}）も`undefined`（size = {width: undefined}）も可能だが，`null`（size = {width: null}）は許されない**

### 余剰プロパティチェック (excess property checking)
- 必須プロパティの他に勝手なプロパティを追加して代入すると，余計なプロパティがあるかという TS のチェック機能が働くことがある
- ただし，**この機能が効くのはオブジェクトリテラルの代入に対してのみ**で，このようにしているのは TS が利便性を優先しているから
```typescript
let onlyX: {x: number};
onlyX = {x: 1, y: 2};  // これは余計なオブジェクトリテラルを直接代入しているので許されない
let xy: {x: number; y: number} = {x: 1, y: 2};
onlyX = xy; // これは変数代入なので通る（利便性を重視した結果）
```

### インデックス型 (index signature)
- 型注釈の時にフィールド名を決めずに指定したい場合，インデックス型を使える（`let obj: {[K: string]: number;}; // これはプロパティがすべて number 型であることを示す`）
- フィールド名は`[K: string]`で`K`部分は任意の型変数名にできるが，`K` or `key`にするのが一般的で，**フィールド名の型は`string`, `number`, `symbol`のみ指定可能（そもそもオブジェクトのフィールドのキーはこの3つ以外許されていない）**
- **フィールド名が定義されていないプロパティにも代入可能**
```typescript
let obj: {[K: stirng]: number};
obj = {a: 1, b: 2};
obj.c = 3;  // ok
obj["d"] = 4;  // ok
```
- `noUncheckedIndexedAccess`を有効にした場合，インデックス型では自動的にプロパティに指定した型と`undefined`のユニオン型になるという点に注意
- **`Record<K, T>`ユーティリティを用いるのと等価であるという点に注意**
```typescript
// 以下は等価
let obj1: {[K: string]: number;};
let obj2: Record<string, number>;
```

### プロトタイプベース
- **オブジェクト指向プログラミング(OOP: Object-Oriented Programing)では，オブジェクトを生成するが，その生成方法には「クラスベース」（Java, PHP, Ruby, Pythonなど）と「プロトタイプベース」（JavaScript）がある**
- プロトタイプ（原型）ベースでは，「元となるオブジェクトから新たなオブジェクトを生成する」（`Object.create(元となるオブジェクト)`）という方法を取る
- クラスベースは，new で新たなインスタンスを作成し，extends で継承した新たなクラスの中で新たな関数を定義したりするが，プロトタイプベースは
```typescript
const counter = {
  count: 0,
  countUp() {
    return this.count++;
  },
};

const resettableCounter = Object.create(counter);
resettableCounter.reset = function (){ this.count = 0; };
```
のように元のオブジェクトから作った新しいオブジェクトのプロパティに追加していく形で継承を行っている
- **ES2015で`class`, `extends`が導入されたので書きやすくはなっている（これも見た目上クラスベースの書き方ができるだけで，中身はプロトタイプベースであることに注意）（当時流行っていた Self の考え方に共感して，シンプルな構造にすることを狙いとしたのと開発期間が１０日と短かったこともあり，プロトタイプベースに行きついたらしい）**

### object、Object、{}の違い
- オブジェクトの型注釈をするときはプロパティの型まで指定するのが一般的
- ざっくり型注釈する方法もあり，`object`, `Object`, `{}`
- `object`はプリミティブを代入できない（`a = /a-z/ //正規表現はオブジェクトなので ok`）
- `Object`はインターフェースであり，プロパティを持つ値ならなんでも代入できる（`null`, `undefined`を以外を除く全てを代入できる）（というのも，プリミティブも auto-boxing によりプロパティを持てるため）
- **`Object`はプリミティブも代入できてしまうので TS では使うべきでないとされている**
- `{}`は`Object`と同じ

### オブジェクトの分割代入 (destructuring assignment)
- プロパティを2度書くのを避ける書き方で，`const { プロパティ名, ..., } = オブジェクト名`
- 別名を指定したいときは，`const { プロパティ名: 別名 } = オブジェクト名`
- **入れ子構造の場合は，その階層分まで入れ子構造を書いて分割代入する**
```typescript
const continent = {
  name: "北アメリカ",
  us: {
    name: "アメリカ合衆国",
    capitalCity: "ワシントンD.C.",
  },
};

const {
  us: {
    name,
    capitalCity,
  },
} = continent;

console.log(name, capitalCity); // アメリカ合衆国 ワシントンD.C.
```
- **デフォルト値を指定する場合，`const { プロパティ名 = デフォルト値 } = オブジェクト名`で書き，そのプロパティが`undefined`のときに代入される（`null`のときはデフォルト値は適用されない）**
```typescript
const color = { r: undefined, g: 122, b: null };
const { r = 0, g: green = 0, b = 0 } = color;
console.log(red, green, b); // 0, 122, null
```

### Shorthand property names
- オブジェクトのキーと変数名が同じときに限って，プロパティ名を省略してオブジェクトに値を代入できる

### オプショナルチェーン (optional chaining)
- JS では`null` or `undefined`のプロパティを参照するとエラーを吐くので，「`null`でない」かつ「`undefined`でない」をチェックする必要がある
- **ネストするとさらに記述量が増えるのでこのシンタックスシュガーとしてオプショナルチェーンがある（`null` or `undefined`を参照したときは`undefined`が返る）**
- **関数，メソッド，配列の呼び出しも引数括弧の前に`?.`を付けて使うことができる**
```typescript
const book = { author: { email: "alice@example.com" } };
// 普通なら
const authorEmail = 
  book === null || book === undefined
    ? undefined
    : book.author === null || book.author === undefined
    ? undefined
    : book.author.email;
// オプショナルチェーンだと
const authorEmail = book?.author?.email;

// 関数の場合
const increment = (n) => n + 1;
const res = increment?.(1); // 2;
// 配列の場合
const book = ["サバイバルTypeScript"];
const title = book?.[0]; // "サバイバルTypeScript"（.いる点に注意）
```
- **TS で使用した場合，得られる型は最後のプロパティと`undefined`のユニオン型になる**
```typescript
let book: undefined | { title: string };
const title = book?.title;
```
- TS のコンパイル結果は`target: es2020`以上の時は，そのままオプショナルチェーンにコンパイルされ，2019以前では参考演算子にコンパイルされる
- **Null合体演算子（`??`）を用いるとオプショナルチェーンが`undefined`を返したときのデフォルト値を代入可能（`const title = book?.title ?? "デフォルトタイトル"`）**

### オブジェクトをループする方法
- for-in 文 or for-of 文（少し工夫が必要）
- **JS のオブジェクトには元になるプロトタイプがあり，オブジェクトリテラルであれば`Object.prototype`になる**
- これを変更するとこのプロトタイプを持つすべてのオブジェクトに影響する
- **for-in 文はプロトタイプのプロパティも含めてループする仕様なので意図しないところでループ回数が変わる可能性があるため，`Object.prototype.hasOwnProperty.call()`でプロトタイプのプロパティでないかを確認したスコープ内で処理を書くのが安全**
- for-of では`for (const [key, value] of Object.entries()){}`で回すことができ，`hasOwnProperty`チェックが不要で，key, value のみが必要ならそれだけを取得すればよい
```typescript
const foo = { a: 1, b: 2, c: 3 };
for (const key of Object.keys(foo)){
  console.log(key); // a, b, c
}

for (const value of Object.values(foo)){
  console.log(value); // 1, 2, 3
}
```
==================================


## 構造的部分型 (structural subtyping)
- クラスを扱うことができる言語では，基本型とその継承関係にある派生型が存在する
- 派生型の派閥として，公称型（派生型同士は互いに置換不可能: C++, Java）と構造的部分型（見た目が同じなら置換可能: Go, TypeScript）がある
- **リスコフの置換原則**があるので，スーパークラスの型でサブクラスを受け取ることは上述したどちらも可能だが，継承したサブクラス同士の型が可換なのが後者のみ
- **TS ではスーパークラス同士でもこれがおこる（interface がおなじであるため）という点に注意**


==================================
## 配列
- **型注釈は`number[]` or `Array<number>`のような２つの書き方がありどちらも同じなので，プロジェクトごとに統一すると良い**
- 配列の代入は参照渡しなので，浅いコピー（一段目までのコピー）を実現するにはスプレッド構文を用いる
- 存在しないインデックスにアクセスしたときにはエラーにならず，`undefined`を得る（これは TS でも指摘できない点に注意）
- 配列へのアクセスを安全にするために，`noUncheckedIndexedAccess`を有効にするとよい
- 読み取り専用の配列を定義するには，`readonly number[]` or `ReadonlyArray<number>`にするとよい
- 読み取り専用にすることで，push, pop はコンパイルエラーがでることになるが，無視して実行ももちろん可能
- readonly array を array に代入はできないが（型アサーションを用いれば別），逆は可能
- 分割代入（基本は先頭から，カンマを使うと指定部分だけ，レスとパラメータで残りをまとめられる等）
- 非破壊的メソッドとして`concat`があり，これは新しい配列を返す
- 注意すべき破壊的メソッドとして`reserve`がある（戻り値があるのに破壊的なので面倒）
  - **元の配列を書き換えず逆順の配列を作りたいなら，`newArray = [...prevArray].reserve`とする**
- ループ処理は基本３つで，`for(let i=0; i<arr.length; i++)`, `for(const value of arr)`, or  `arr.forEach((value, i) => {})`メソッド
- メソッドだけは`continue`や`break`が書けないため若干不便なこともあるかも
- **`for in**は順番が保たれる保証がなく，途中で追加されたプロパティも回してしまうという欠点があるらしい
==================================


## タプル (tuple)
- **TS の関数は値を１つしか返せないので，複数返したい場合はリストに入れて返却する**
- **配列の要素が複数の型で構成される者に対して，`any` or `unknown`で作成するよりもタプルを使用したほうがよい**
- **タプル型は`[]`の中に型を書くだけ**
```typescript
function tuple(): [number, string, boolean] {
  return [1, "ok", true];
}

const list: [number, string, boolean] = tuple()
list[0].toExponential();
list[2].valueOf();  // プリミティブな値を返す

// 分割代入
const [num, , bool]: [number, string, boolean] = tuple();
```
- タプルの使い道の一つとして，時間のかかる処理を並列に行うときに`Promise.all()`を使用する場面があり，そこで使うらしい（`const tuple: [number, string] = await Promise.all([takes5Seconds(), takes3seconds(),]);` // 順番に注意）

## 列挙型 (enum)
- 以下のように宣言し，それぞれがプロパティになる TS 独自の型
```typescript
enum Position {
  Top,
  Right,
  Bottom,
  Left,
}

// オブジェクトのプロパティになり，連番になる
console.log(Position.Top); // 0
console.log(Position.Bottom); // 2
```
### 数値列挙型 (numeric enum)
- もし値を代入した場合，残りのメンバーはそこからの連番になる
### 文字列列挙型 (string enum)
- 文字列を代入することで文字列列挙型として使用可能
```typescript
enum Direction {
  Up = "Up",
  Down = "Down",
}
```
### 列挙型(enum)の問題点と代替手段
- 列挙型にはいくつかの問題（特に型安全上の問題など）があり，代替のものを使用したほうが良いという案もある
- **１つは「ユニオン型を使う」，もう１つは「オブジェクトリテラルを使う（`as const`）」という案がある**

## ユニオン型 (union type)
- いずれかの型を示すやつ（`type List = (number | string)[]; \\ number | string[]としてしまうと配列はstringだけになる`）
- `string | null`が`string`のときのみ何かしたいのなら，`if(typeof maybeUserId === "string")`とするとよい

## 判別可能なユニオン型 (discriminated union)
- 通常のユニオン型は自由度が高く，好きな型を組み合わせられる
- もしファイルの状態管理に`type UploadStatus = InProgress | Success | Failure`のユニオン型を使用していたとすると，`InProgress`は進捗率％というプロパティを持っており，他成功か失敗のフラグを持っているなどの状況が考えられるため，絞り込みに`プロパティ in  オブジェクト`のロジックを逐一書く必要が出てくる
- 上記を可読性の高い状態で実現するのが識別可能なユニオン型で，リテラル型（+ undefined + null）のディスクリミネータと呼ばれるプロパティを持っているものを指し，これを満たすことでオブジェクトごとに固有のプロパティを保持できるというもの（`type InProgress = {type: "InProgress"; progress: number;}`では"InProgress"がリテラルで type ディスクリミネータを指す）
- これは`status.type`で分岐することができる（if 文 or switch 文で）
- ディスクリミネータは分割代入で変数にすることも可能（`const { type, color } = shape;`）

## インターセクション型 (intersection type)
‐ ユニオン型と相反しており，どの型もを示す（`type ThreeDimensionalPoint = TwoDimensionalPoint & Z;`）
- プリミティブ同士で作ると，`never`という型が出来上がり何も代入できなくなる（意外なところに使い道があるらしい）
- システムの巨大化によりパラメータが巨大化したとすると，必須パラメータとそうでないものに分離した後に，`type Mandatory = Required<T>`と`type Optional = Partial<T>`とに分けて，`type Parameter = Mandatory & Optional`とすることでわかりやすいものになる

## 型エイリアス (type alias)
- プリミティブ，リテラル型からなんでも別名を付けられる機能（`type OK = 200;`）
- 何度も同じ型を使いまわすときに保守性が向上する

## 型アサーション「as」(type assertion)
- 型推論の上書き機能
- 書き方は２通りで，`as`構文 or アングルブラケット構文（JSX と似ているためあまり使われない）
```typescript
const value: number | string = "this is a string";
// as 
const strLength = (value as string).length;
// angle-bracket
const strLength = (<string>value).length;
```
- なんでも`as`で変換できるわけではないが，`unknown`を経由することでエラーを回避できたりもする（`const num = 123; const str: string = num as unknown as string`）
- キャストは型変換だが，型アサーションは変換しない（コンパイラに伝えるだけなのでJS の実行時に影響しない）
- **型アサーションは強力なので，先に型ガードなどで解決を図ってから使うのが良い**
- 型注釈は型ヒントを与えてあげるのに対し，型アサーションは今コンパイラが考えている型を訂正する役割がある

## constアサーション「as const」 (const assertion)
- **オブジェクトリテラルの末尾に`as const`をつけるだけですべてのプロパティが `readonly`かつリテラル型を指定したことと同じ扱いになり，完全固定される**
- プロパティの先頭に`readonly`をつけるのと似ているが，`readonly`は必要なプロパティを選択してつけられる上，constアサーションは再帰的に`readonly`になるという点も異なる

## 明確な割り当てアサーション(definite assignment assertion)
- **`strictNullChecks`が`true`のとき，初期化されていない変数参照を行うとエラーになり，別の関数内で初期化されていたとしてもエラーになる**
- **上記オプションに加えて，`strictPropertyInitialization`が`true`のときにも constructor 以外でクラスのプロパティが初期化されている場合は追いかけずに警告を出す仕様**
- これをコンパイラに伝える機能として，definite assignment assertion がある（プロパティ名の後に`!`をつける）
- 変数を参照する部分で変数の後に`!`をつける非Nullアサーション(non-null assertion)もある
```typescript
let num!: number; // definite assinment assertion
initNum();
console.log(num * 2);
// console.log(num! * 2); // non-null assertion

function initNum() {
  num = 2;
}
```
- **コンパイラから型チェックの権限を人間に移すものなのでできるだけ使わない方が安全なので先に型ガードなどの別の案を検討することを推奨**

## typeof 演算子
- `typeof true`(boolean)みたいなことが可能
- **`typeof null`は"object"という点に注意**
    - null ではない object を判定したければ`value !== null && value === "object`"と書く必要がある
- **配列も object と判定されるので，配列かどうかを判定したい場合は`Array.isArray(n)`を使用する**

## truthy, falsy
- false ではないが false　のように扱われる値を falsy という
- NaN(number型), 0n(bigint)等
- これは例えば null を取り除きたい場面などで意図しない挙動を生む可能性があるので，**きちんと true, false で返すようにする**


# 気づいたこと，ライブラリの使い方など
- substring について
```typescript 
x: string = "hello";
console.log(x.substring(1, 3)); //[1, 3)の半開区間なので，"el"
```
- `NaN`が出てくる時と判定方法
- JSONオブジェクトのそれぞれの使い方(`JSON.parse()`, `JSON.stringify()`)
- `Bigint(100)`や`new String(str)`で new がつくか否かは，前者が関数で後者がオブジェクトであるという違いだけ