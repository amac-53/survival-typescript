# import、export、require

## スクリプトとモジュール
- モジュールでない（import, export がない）JS ファイルはスクリプトと呼ぶ

## 値の公開と非公開
- export しないと公開されないので，**基本的な方針は値や関数の非公開**（Java などの他の言語ではメンバがデフォルトで公開されることも）

## モジュールは常にstrict mode
- **モジュールは常に`strict mode`である**という点に注意
- `strict mode`では未定義の変数への代入がエラーになるなどの少し厳格な JS

## モジュールはimport時に一度だけ評価される
- 複数回 import した場合は最初の import しか評価されない（キャッシュされる, or シングルトン的なもの）（**シングルトンは実行時に１つのインスタンスしか生成できないようなパターン**）
```javascript: module.js
console.log("モジュールを評価しています")
export const value = 1;
```

```javascript: main.js
import "./module.js"; // モジュールを評価しています
import "./module.js";
import "./module.js";
```

## モジュールの歴史的経緯
- かつてもモジュールという考え方は存在していたが，JQuery などでは html ファイル上で，`script`タグで依存関係（読み込む順番）に注意して読み込んで使う程度のものだった
- Node.js の登場後，使いたいパッケージをそのまま持ってくる形が主流になった

## CommonJS
- `module.exports`で export でき，読み込むときは名前を付け替えることが可能
- `module.exports`はいくらでも書けるが，適用されるのは最後のやつだけ
- `module.exports`は読み込み側でいくらでも名前を書き換えることが可能なので，名前を変更されたくない場合は，`exports.increment = (i) => i + 1`のようにする（分割代入を使用することで名前の書き換えは実は可能）

```javascript
// node_modules パッケージ
const package1 = require("package1");
// 自作モジュール
const myPackage = require("./MyPackage");
```
```javascript
// export ファイル (dayOfWeek.js)
module.exports = "Monday";
module.exports = "Tuesday";
module.exports = "Wednesday";  // これしか export されない

// require ファイル（名前変更可能）
const day = require("./dayOfWeek");


// export ファイル（util.js）
exports.increment = (i) => i + 1;

// require ファイル
const util = require("./util");
console.log(util.increment(3));  // 4

// or 分割代入
const { increment } = require("./util");
console.log(increment(3));  // 4
```

## ES Module
- **`require`はファイルのどこに書いてもいいが，`import`は一番上に書く必要がある**
- named import のときは，`import { increment as inc } from ファイルの相対パス`のように書く
```typescript
const increment = (i) => i + 1;
export { increament };
```
- import は先頭に読み込むファイルを書くので，動的に切り替えられない（if 文などで必要なファイルを分けることができないということ）
- これを解決する手段として，`import()`がある（require と異なる点として非同期にモジュールを読み込む点が挙げられる）
```javascript
import("./util").then(({ increment }) => {
    console.log(increment(3)); // 4
})
```

## Node.jsでES Moduleを使う
- CommonJS で動作するのが前提なので準備が必要
- １つは，`.mjs`という拡張子に変更する（呼び出し側も呼び出される側も）（**`import`文では`mjs`という拡張子を省略しないこと**）
- もう１つの選択肢として，`package.json`に **`type: "module"`を追加することで，拡張子を変更せずとも ES モジュールを使える**（type を省略してしまうと，`type: "commonjs"`と指定したとみなされる）
- 上記の設定で CommonJS を使いたくなった場合，`.cjs`をつける & 呼び出す側は`createRequire`を用いるなどの工夫が必要となる
- **現時点では，多くのパッケージが ESモジュールに対応していないので，`type: "module"`という設定を適用したとしても，`.js`ファイルを読み込めない（内部がCommonJSで書かれているため）上，`.cjs`に拡張子を変更したとしても`createRequire`とかいう文法に書き換えないといけないので結局動作しないという使いづらい環境になっている**

## TypeScriptでは
- TS では ESモジュールで書くが，CommonJS を使いたいときにはコンパイル時に設定することでどっちにも対応できるようになっている
- 拡張子に関しては，`.cjs`, `.mjs`を出力する設定はないのでブラウザ，サーバどちらでも使用可能な JS を生成するには若干工夫が必要
- サーバーなら CommonJS, ブラウザなら ESモジュールが無難な選択肢にはなる
- アップデートなどでロジックを変更したことを明示したいのであれば，named export を使うことが望ましいが，全体に関しては好みの範疇