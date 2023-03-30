# 文

## 変数のスコープ (scope)
- JS のグローバル変数は１つだけ（window）
- ブロックスコープで，レキシカルスコープ変数（定義した段階で決定するスコープ）
- **`let`や`const`をつけ忘れると，グローバル変数として扱われてしまう（JS の残念な仕様だが，TS ではこれを指摘してくれる）**

## if-else文
- if-else の複数のブロックなどで変数を扱う場合には，let が便利

## 三項演算子 (ternary operator)
- 条件をネストすることで else if を書くことができる（可読性の観点から微妙そう）

## for-of文 - 拡張for文
- for of でインデックスを得るには，**配列の`entries`メソッド**を用いる
```javascript
const words = ["I", "love", "TypeScript"];
for (const [index, word] of words.entries()){
   console.log(index, word); 
}
```

## switch文
- （`switch (条件) { case 値A: 処理, ... })`）
- switch の条件を比較する`case`は厳密等価演算（`null`と`undefined`は等価だが厳密等価ではない）

## switchのフォールスルー問題
- `case`では条件を分岐させる機能がないので，break が必要であり，これがバグの原因になり得る
- `noFallthroughCasesInSwitch`を`true`にすると，フォールスルーを警告してくれる

## switchと変数スコープ
- **switch ごとにスコープが作成される**ので，それぞれの case 内に同じ変数を宣言するとエラーになる
- どうしても case ごとにスコープを作成したければ，`case 1: {}`のように中括弧で`case`を囲む
  
## 例外処理 (exception)
- try-catch(-finally) 文によりエラーを追うことができる
- 通常は`try`の中に`throw new Error()`とかく
- `useUnknownInCatchVariables`を有効にすると，`catch`の変数の型が`unknown`型になる（JS のデフォルトでは any 型になるので，どんな型がなげられるかを正確に表現するために有効化するとよい）
- JS の catch は１つだけなので，エラーハンドリングを分岐したい場合は catch ブロックの中で分岐を書く必要がある
```typescript
try {

} catch (e) {
  if(e instanceof TypeError){

  }else if(e instanceof RangeError){

  }
}
```
- try-catch はブロックスコープなので，try-catch の外で変数を参照したい場合は`let`にしておく

## never型
- どのような型も入れられない（anyであっても）
- 逆に never から他の型への代入は可能
- `void`は値をもてるが，`never`は値をもてない
```typescript
const ok: void = undefined;  // ok
const ng: never = undefined;  // ng
```
- **`void`と`never`は戻り値が無いという点は同じだが，関数が終了するかが異なる**（never は中断される（例外が必ず発生する関数など） or 永遠に実行される（無限ループ））
- **`never`型を使った網羅性チェックが有用な使い方の１つ（ユニオン型で条件分岐を行うときにロジックがパターンを網羅しているかどうかを default 部分でチェックできる）（チェックの方法はいくつかあるが，例外を投げる方式にすると後々のためになる）**
  1. `noUnusedLocals`コンパイラオプションを使うことで，使われていない変数について警告をだすかを設定可能
  2. 実行時の JS を見ても意味がわかるコードになる

## 制御フロー分析と型ガードによる型の絞り込み
- ユニオン型で型注釈を書いたときに，片方の型でのみ定義されているメソッドやプロパティにアクセスすると型エラーが起こる
- 型ガード内で特定の型処理を終えた後に return を書くことで，残りのロジックは残っている型と判断されるような書き方も可能
- オブジェクトの判定は`null`を選択肢から排除するように注意すること
- `typeof`ではオブジェクトであることまでしか判断できないが，`instanceof`を使用することで**特定のクラスの**インスタンスであるところまで判定可能
- **特定のプロパティを持つかを判定する型ガードを`in`を用いることで書ける**
- 型ガードを関数化した型ガード関数，型ガードを変数に使用するパターン（`const isDate = date instanceof Date; // TypeScript4.4以降で可能`）

## unknown型
- 型がわからないときに使い，どんな値も代入可能（型安全な any と言われる）
- **any はどの型に対しても代入可能な一方，unknown は代入不可能**
- **プロパティへのアクセス，メソッドの呼び出しも許されない**
- 型の絞り込みには`typeof`, `instanceof`含んだ if 文を用いる（型ガード）
- これを関数化した型ガード関数を用いることも
- **`unknown`を配列型に絞り込むときは，`Array.isArray()`を用いる（この中でさらに配列要素までチェックするとより安全になる）**
- `unknown`をオブジェクト型に絞り込むときは，`typeof`がオブジェクトかつ`null`チェックをするが，**その他にプロパティまでチェックするのが安全**
- 逐一実装するのは面倒なので，バリデーションライブラリの導入が楽（`zod`, `superstruct`など）
- **unknown の用途として，「any 型で返す関数を unknown にすることでより安全に使用する使い方（`const data: unknown = JSON.parse("...")`）」と「型アサーションの制約回避がある（`str as unknown as number`など）」**
- TS4.4以降では`tsconfig`の設定によって標準で投げられる例外を`any`or`unknown`のいずれかで選べるようになっている（`useUnknownInCatchVariables`）

## anyとunknownの違い
- いずれもどの型の値も入れられる一方，`unknown`から特定の型は不可能となっており，`unknown`ではプロパティ，メソッドへのアクセスも許されない（その分安全にチェックできるということ）

## おまけ
- `toFixed`, `toUpperCase`, `startPad`, `配列.every()`