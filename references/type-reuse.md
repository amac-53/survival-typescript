# 型の再利用
- 既存の型を使用して新しい型を生み出すことを指す
- 例えば JS であるオブジェクトを定義し，そのオブジェクトのキーが欲しいときは，別途ハードコーディングするよりも`Object.keys()`で既存のものを再利用する方が保守性が高い
- 型もこれと同様のことが言えるということ


## typeof型演算子
- 変数から型を抽出する演算子で，型定義にも用いることが可能 
- ここでの typeof は TS のもので JS のもの（値の型を調べるものでオブジェクトの中身を見て細かく判定はしてくれない）とは異なるという点に注意
```typescript
// typescript の typeof
const point = { x: 100, y: 200 };
type Point = typeof point; // { x: number, y: number }
```


## keyof型演算子
- **オブジェクト型**からプロパティ名を型として返す演算子
- 複数プロパティを持つオブジェクトの場合，ユニオン型で返される
- **インデックス型に使用すると，インデックスのキーの型が返るが，{[K: string]: number};みたいな場合は，`string | number`が返る**（obj[0]とobj["0"]は同じだから）
- Mapped Types に使用すると，キーの型が返る
- プロパティを持たないオブジェクトに使用すると，`never`が返る（`keyof {}`）
- any に対して使用すると，`string | number | symbol`が返る（`keyof any`）
- 利点はプロパティの記入漏れや追加の際の型の再定義がいらないので，保守性が上がるという点
- 基本的に`Mapped Types`と一緒に使われる（後述）
```typescript
type Person = {
    name: string;
}
type PersonKey = keyof Person; // type PersonKey = "name"

// インデックス型
type MapLike = { [K: string]: any };
type MapKeys = keyof MapLike; // type Mapkeys = string | number

// Mapped Types
type MapLike = { [K in "x" | "y" | "z"]: any };
type MapKeys = keyof MapLike; // type Mapkeys = "x" | "y" | "z"
```


## ユーティリティ型 (utility type)
- 型から別の型を導き出してくれる型で，型の世界の関数みたいなイメージ
### Required<T>
- 全てのプロパティからオプショナル（`?`）を取り除いた型
```typescript
type Person = {
    surname: string;
    middleName?: string;
    givenName: string;
};

type RequiredPerson = Required<Person>;
// RequiredPerson = {
//     surname: string;
//     middleName: string;
//     givenName: string;
// }
```
### Readonly<T>
- 全プロパティを読み取り専用にする型
- **注意点として，再帰的でないので直下の階層のプロパティしか readonly にならない**
### Partial<T>
- 全プロパティをオプショナルプロパティ（`?`をつける）にする型
- Options Object パターンとの組み合わせが有効
- **FindUserArgs だけでは呼び出し時に引数が必要ない場合でも，findUsers({})とする必要があり，これをなくすためにデフォルト引数に`{}`を与える**
```javascript
// 先に User といういろんなプロパティがある型を定義している
// この宣言で全てのフィールドがオプショナルになる
type FindUserArgs = Partial<User>;

function findUsers({
    surname = "Doe",
    middlename = "John",
    ...
}: FindUserArgs = {}){
    // 処理
}
```
### Record<Keys, Type>
- インデックス型を表すので，**`Keys`は string, number, symbolとリテラル以外は不可能**
- 以下はリテラルがキーで，値が文字列になるオブジェクトの例
```javascript
type Person = Record<"firstName" | "middleName" | "lastName", string>;
const person: Person = {
    firstName: "Robert",
    middleName: "Cecil",
    lastName: "Martin",
}
```
### Pick<T, Keys>
- T に入ったオブジェクト型から Keys で指定したオブジェクト型を返す
- 元の型の部分型として作成できるため，元の型の大枠を変える場合に`Pick`を使っていない場合は逐一部分型の設定を探しに行って変更する必要があり，非常に面倒
```typescript
type User = {
  surname: string;
  middleName?: string;
  givenName: string;
  age: number;
  address?: string;
  nationality: string; 
}

type Person = Pick<User, "surname" | "middleName" | "givenName">;
// type Person = {
//   surname: string;
//   middleName?: string;
//   givenName: string;
// };
```
### Omit<T, Keys>
- T のオブジェクト型から Keys で指定したプロパティを除いたオブジェクトを返す
- **注意点として，Pick は存在しないプロパティを指定した場合，TS コンパイラが指摘する一方で，Omit では指摘してこないことが挙げられる（したがって，タイポがある場合は省いたつもりでいることもある）**
### Exclude<T, U>
- T で指定した**ユニオン型**から U を除いたユニオン型を返す
- **Omit と同様に部分集合である必要がないので，タイポしていてもコンパイラが指摘することはない**
- **元のユニオン型に何らかの追加や変更を加えた場合は，Exclued した型の第二引数も変更しにいかないと意図した挙動にならないので注意**
### Extract<T, U>
- Exclude の逆でユニオン型 T から U だけを抽出したユニオン型を返す 


## Mapped Types
- インデックス型ではどのようなキーも設定できてしまうため，アクセス時に`undefined`かどうかのチェックが必要になる
- **入力の形式が決まっているのであれば Mapped Types を使うとよい**
- 上述した`Readonly<T>`もこの機能で実現されている
- **インデックス型との違いとして，追加のプロパティが定義できない点が挙げられる**
- もしプロパティを追加したければ，追加部分を別オブジェクトとして定義し，Mapped Types とのインターセクション型で実現する
```javascript
type KeyValues = {
    [K in string]: string;
};
type Name = {
    name: string;
}
type KeyValuesAndName = KeyValues & Name;
```

## インデックスアクセス型 (indexed access types)
- プロパティの型や配列要素の型を参照する方法で，`オブジェクト型["プロパティ名"]` or `配列型[number]`でアクセスする
- ユニオン型でのアクセスも可能
```typescript
type Person = { name: string; age: number };
type T = Person[keyof Person]; // string | number
```
- 配列へのアクセスは`[number]`をつける（tips でも述べているが，`number`でアクセスするのは配列であることを明示するため）
```typescript
type StringArray = string[];
type T = StringArray[number]; // string
```
- typeof との組み合わせでタプルから各要素の型を導くことも可能
```typescript
const stateList = ["open", "closed"] as const;
type State = typeof stateList[number];  // type State = "open" | "closed"
```
- 存在しないプロパティ名を指定すると警告される
- タプルの要素参照は数値リテラルを用いる
```typescript
type Tuple = [string, number];
type T = Tuple[0]; // string
```