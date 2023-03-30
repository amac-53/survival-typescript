# 組み込みAPI
- 特殊なオブジェクトと考えておく

## Map<K, V>
- いわゆる辞書や連想配列で，オブジェクトの一種
- `const map = new Map<string, number>()`により，オブジェクトを作成する
- 型を省略した場合は型推論してくれる
- コンストラクタ引数も型も省略した場合は，`Map<any, any>`になる
- Map の型アノテーションは，`map: Map<string, number>`のように行う
- key は厳密等価比較される（`null !== undefined`となるが，**NaN は厳密等価ではないが例外的に同じとみなされる**）
```typescript
const map = new Map<string, number>();
map.set("a", 1);
console.log(map.get("a")); // 1

// コンストラクタ引数 ver.
const map = new Map<string, number>([
    ["a", 1],
    ["b", 2],
    ["c", 3],
])
console.log(map); // {"a" => 1, "b" => 2, "c" => 3}

// 比較 ver.
const map = new Map<number, number>(); // 空 Map
map.set(NaN, 1);
map.set(NaN, 2);
console.log(map); // {NaN => 2}

const map = new Map<object, number>();
map.set({}, 1);
map.set({}, 2);
console.log(map); // {{} => 1, {} => 2}
```
### Mapの操作
- `map.set("a", 1);`で値を set でき，上書きもそのままできる
- `map.get("a");`で値を get できるが，ないものは undefined で返される
```typescript
// null合体演算子との合わせ技
const map = new Map([["a", 1]]);
console.log(map.get("a") ?? 2); // 2
```
- `map.delete("a")`で値を削除する（**key が存在しない場合は false を返し，存在する場合は true を返す**）
- `map.has("a");`で key が存在すれば true, そうでなければ false を返す
- **TS では`has`で key を存在確認したのち，`get`する方法はうまくいかないため，はじめから`get`したのち，`undefined`でないかを確認するとよい**
- `map.size;`で要素数を調べる（**これだけフィールド**）
- `map.clear();`で全要素を削除
- `map.keys()`で key の反復可能オブジェクトを返す（`const keys = [...map.keys()]; // これで配列にできる`）
- `map.values()`で value の反復可能オブジェクトを返す（`const values = [...map.values()]; // これで配列にできる`）
- `map.entries()`で key, value の反復可能オブジェクトを返す（`const keyValues = [...map.entries()];`）
- for で回す（`for (const [key, value] of map){}`）
- **浅いコピーを行うには`map`を次のコンストラクタに渡せばいい（Tips でも紹介されている方法）**
```typescript
const map1 = new Map<string, number>([["a", 1]]);
const map2 = new Map(map1);  // 浅いコピー
console.log(map2);
```
- 配列 <= Map（逆は実用的でなさそう）, オブジェクト <=> Map が可能
- **直接 JSON 化できないので，一度オブジェクトに戻す必要がある**
```typescript
const map = new Map([
    ["a", 1],
    ["b", 2],
]);

// Map => 配列
const keyValues = [...map];

const obj = Object.fromEntries(map);  // Map => object
console.log(JSON.stringify(obj)); // object => JSON

const obj = { "a": 1, "b": 2 };
const map = new Map(Object.entries(obj)); // 一旦
```
- **オブジェクトとの違いは，キーの上書きが起こるか，キーの型（Map は任意の型が ok），反復順（Map は挿入順なので単純），JSON化などの点で異なる**


## Set<T>
- 重複を許さない値の集合
- **型を決めずに空 set を初期化すると`unknown`型になり，後から値を set できない**ので，先に型を決めておくこと
```typescript
const fruits = new Set(["apple", "apple", "orange", "banana"]);
console.log(set); // { 'apple', 'orange', 'banana' }

// 型を先に決める
const fruits = new Set<string>();
```
### Setの操作
- `fruits.add(1).add(2)`のようにすると値が追加した値の順番通りに追加できる
- `fruits.delete(1)`で値を削除する
- `fruits.has(1)`で値があるかを確認
- **`fruits.size`で要素数を確認**
- `fruits.clear()`で全削除
- `for (const fruit of fruits){}`で回す
### 便利な使い方
- Set => 配列
- **JSON 化するには，一度配列に戻す必要がある
```typescript
const fruits = new Set(["apple", "orange", "banana"]);
const array = [...fruits];
```
- **配列から重複要素を取り除きたいとき**（一度 Set にしてから配列に戻す）
```typescript
const array1 = [0, 0, 1, 1, 2, 2];
const array2 = [...new Set(array1)];
console.log(array2); // [0, 1, 2]
```


## Date
- Java の Date クラスを基にして，10日でエンジンに組み込まれたらしく API がひどいらしい
- この理由からそのまま使わず，パッケージの導入を検討するといいらしい
### よく見る日付操作のパッケージ
1. `date-fns`（関数として操作する
2. `Day.js`（`Moment.js`と互換性があり `Moment.js`からの乗り換え先として検討される）
3. `Moment.js`（絶大な知名度を誇るが新規開発をなされていないのでこれから取り入れるべきかは疑問）
### Dateの操作
- **`Date.getFullYear()`で年**
- `Date.getMonth()`で月（0-11を返すので注意）
- `Date.getDate()`で日付，`Date.getDay()`で曜日
- `Date.getHours()`で時間，`Date.getMinutes()`で分，`Date.getSeconds()`で秒，`Date.getMilliseconds()`でミリ秒
- **ISO8601形式（`YYYY-MM-DDThh:mm:ss.sssZ`）に変換するのに，`Date.toJSON()`を使用**


## RegExp
- 正規表現のための組み込みクラスで，リテラル表記とコンストラクタを使用した表記の２通り存在
- **リテラルは，`/`で文字を囲み最後にフラグを書く**
- コンストラクタは第一引数に検索したい文字列，第二引数にフラグを書く（`\\d`と表記している点に注意）
- ふつうはリテラルを用いる
```typescript
const regexp1 = /0(8|9)0-\d{4}/g;
const regexp2 = new RegExp("0(8|9)0-\\d{4}", "g");
```
### RegExpの操作
- `regex.test("検索文字列")`で一致するものがあれば true, そうでなければ false を返す
- `regex.exec("検索文字列")`でマッチした文字列を返し，何もなければ null を返す
- **上記はいずれも，RegExp のメソッドだが，string のメソッドにも同じものが存在し，`str.match(正規表現)`で`regex.exec()`と同じことが可能**（`g`フラグがある場合は完全一致した文字列の配列を返す点に注意）
- キャプチャグループを名前で指定することができる機能は TS とは相性が良くないため，オプショナルチェーンを使うとよい（後で）


## Error
- 例外を扱うためのオブジェクト
- `new Error()`でオブジェクト生成を行い，`throw`で投げる
- どんな値でも`throw`可能（`throw "id is not string!";`）
- 組み込みとして`Error`のサブクラスには，`EvalError`, `InternalError`など７つのクラスがある（自分でエラーのサブクラスを定義することももちろん可能）
```typescript
class CustomError extends Error {
    construnctor(message?: string){
        super(message);
    }
}

const err: CustomError = new CustomError("FAILED!");
console.log(err.name);    // Error
console.log(err.message); // FAILED!
```
- `throw`された例外は`catch`で補足できるが，どのような値でも`throw`できるため，型は定まらず`tsconfig.json`の`useUnknownInCatchVariables`の設定で，`any` or `unknown`のいすれかの型として解釈される
- `catch`した型やインスタンスを絞りたければ，型ガードを使う
```typescript
try {

} catch (e) {
    if(e instanceof Error){
        // 処理
    }
}
```