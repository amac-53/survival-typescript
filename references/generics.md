# ジェネリクス (generics)
- 同じようなロジックを持つ関数を複数作るのは面倒なので共通化したいが，そうすると`any`型などを使う選択肢がでてきて型安全が損なわれるというトレードオフがある
- これを解決する仕組みで，**型も変数のように扱う**という思想のもとに成り立っている

## ジェネリクスが使われている標準ライブラリ
- Array, Promise, Map オブジェクトが代表（`const textNumbers: Array<string> = ["1", "2", "3", "4"]`）
- Array オブジェクトの map メソッドも１つの例（`textNumbers.map<number>(function (text: string){
    return Number(text);
})`）
- 標準ライブラリの型定義ファイルは TS の公式リポジトリの lib ディレクトリで確認可能

## 型変数 (type variables)
- 型を入れられる変数（`function printAndReturn<T>(value: T): T{}`の`T`の部分）
- 関数スコープなので外で参照はできない
- 呼び出し側で代入した型を**型引数**と呼ぶ（`const value = printAndReturn<number>(123);`）
- この部分は省略も可能で，その場合はコンパイラによる推論が走る
- 型変数に使う名前はなんでもいいが`T`(templateの略)(`U`, `V`)が一般的
- 意味づけを行う場合は，`TKey`, `TValue`などのTを接頭辞にすることが多い
- 命名規則の立場として，わかりやすさのために長い型変数名にした方がよいとする立場とクラス名などと混同してしまうので抽象度の高い T にすべきという立場がある
- 別の立場として，意味がある名前にする場合は割と長いスコープで，短い場合は T とかにすべきという勢力も存在する

## 型引数の制約
- 例えば HTML 要素を書き換えたい場合，T に来る型はある程度制約があるので，TS では存在しないプロパティへのアクセス検知としてコンパイラに止められてしまう
```javascript
function changeBackgroundColor<T>(element: T){
    element.style.backgroundColor = "red";  // sytle プロパティの存在しない場合があるためコンパイルが通らない
    return element;
}
```
- `(element as any).style...`として回避することも可能だが，型チェックを外してしまうのはできる限り避けたい
- **`T extends HTMLElement`を用いることで`T`を HTMLElement のサブクラスに限定することができるため安全にアクセス可能に**
- **インターフェースを型引数に用いる場合は extends を使用可能**（実装のときには，implements キーワードなので注意）

## デフォルト型引数
- `T = Error`などとすることで，デフォルト指定が可能
- **ある型の extends であることを明記しながら省略時にはデフォルト型を指定する合わせ技もある（ただしこの場合はデフォルト型が extends する型の制約を満たしている必要がある）（`type MyErrorType<T extends Error = SyntaxError> = { error: T; type: string;};`）**
- **すでに定義された**デフォルト型引数をデフォルト型引数で指定することが可能（`<A, B = A, C = B>`みたいなこと）