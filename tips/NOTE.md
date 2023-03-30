# オブジェクトを浅くコピーする
- オブジェクトはデフォルトが参照渡しなので，そのままいろんな関数に渡してたりすると意図しない書き換えが起こり得る（チーム開発であれば尚更）
- これを防ぐために浅いコピーを使用するといいよってこと
- **深いコピーでは時間的，空間的領域共に多く浪費する**ため，浅いコピーで事足りるときは使わない方が良い

- オブジェクトの浅いコピー
```typescript
const sample: object = {
    year: 1999,
    month: 7,
};

// 分割代入を使用（ES2018以降ではコンパイル後も変わらない）
const shallowCopied: object = { ...sample };

// ES2018以前は assign 命令にコンパイルされる
const shallowCopied  = Object.assign({}, sample); 
```
- Map, Set では浅いコピーを簡潔に書ける API が提供されており，配列ではスプレッド構文を用いる
```typescript
// Map
const map1 = new Map([
    [".js", "JS"],
    [".ts", "TS"],
]);
// 浅いコピー
const map2 = new Map(map1);

// Set
const set1 = new Set([1, 2, 3]);
// 浅いコピー
const set2 = new Set(set1);

// Array
const array1 = [1, 2, 3];
const array2 = [...array1];
```

# オブジェクトをマージ (結合) する
- JS, TS で行われるマージは２つのオブジェクトから新しいマージ済みのオブジェクトを生成することを指す
- スプレッド構文で引数を並べるだけでよい
```typescript
const obj1: object = {
    firstName: "Luffy",
    middleName: "D",
    LastName: "Monkey",
};

const obj2: object = {
    firstName: "Ace",
    LastName: "Portgas",
};

// ここで Ace, D, Portgas に置き換わる
const merged = { ...obj1, ...obj2 };

// ES2018以前でコンパイルすると
const merged = Object.assign(Object.assign({}, obj1), obj2);
```
- **同名のプロパティが存在する場合は最後に書かれているものに置き換えられるので注意**


# オブジェクトのサブセットを得る
- オブジェクトの中のいくつかのプロパティを持つオブジェクトを作りたい場合，「合わせ技」 or 「lodashを用いる」の２通りの方法がある
（これは現状必要なさそうなので後で．．．）

# オブジェクトで受け、オブジェクトを返す (RORO: Receive an Object, Return an Object)
- 初心者は，関数に引数が追加されたときのことを考慮せずに作ってしまう
- データを１つのオブジェクトにした引数にしてしまえば解決できる
- この書き方の利点は分割代入が可能であると同時にここにデフォルト値を付け加えられるという点
```typescript
type UserInfo = {
    name?: string;
    age?: number;
    country?: string;
    natinality?: string;
    isVip?: boolean;
    isRetired?: boolean;  // 後から追加
}

// 引退済みの人だけ検索する
function findUserByName({ name, age, isRetried = false }: UserInfo): User {
    // 処理
}
```

# コンパニオンオブジェクトパターン
- 値と型に同名を与えて区別なく使用するテクニックがある
- クラスを作るほどでもないが，それっぽい**ファクトリーメソッド**とオブジェクトがほしいときに重宝する
```typescript: rectangle.ts
export type Rectangle = {
    height: number;
    width: number;
};

export const Rectangle = {
    from(height: number, width: number): Rectangle {
        return {
            height,
            width,
        };
    },
};
```
- 以下は import する側（import が`Rectangle`だけになり，見通しもつきやすいという特徴がある）
```typescript
import { Rectangle } from "./rectangle";

const src: Rectangle = Rectangle.from(1, 3);
console.log(src.height);  // 1
console.log(src.width);  // 3
```

# オブジェクトから型を生成する
- 他の言語と異なり，TS は後から型を決定することも多々ある
- TS の`typeof`演算子をかますことでどのような型と認識されているかがわかる
- プロパティ全てを定数で取得するにはオブジェクトの最後に`as const`をつける
-　特定のプロパティのみを定数で欲しい場合は，`as`を特定のプロパティのみにつければよい
```typescript
const account = {
    accountName: "yyts",
    password: "amacdgab",
    age: 80,
    plan: "Standard" as "Free" | "Standard" | "Premium", // ここだけ定数
}

type Account = typeof account; 
// type Account = {
//     accountName: string;
//     password: string;
//     age: number;
//     plan: "Standard" | "Free" | "Premium";
// }
```

# オブジェクトからキーの型を生成する
- 特定のオブジェクトのキー全てから生成される型が欲しい場合は，`typeof`と`keyof`を使用する
- `keyof`は**オブジェクトの型から**プロパティ名をユニオン型で返す演算子なので，`type Langeage = keyof typeof オブジェクト`とするべき

# オブジェクトからプロパティの型を生成する
- 上記のプロパティ名ではなく，プロパティ自体のユニオン型がほしいときはキーの型を生成した後に，`Mapped Types`を使用すればよい
- `as cosnt`を使うことでリテラル型を取得可能
```typescript
// プロパティの型
const conf = {
    en: "Are you sure?",
    fr: "Êtes-vous sûr?",
    es: "está seguro?",
    ja: "よろしいですか？",
} 
type Language = keyof typeof conf;
type ConfirmationMessage = typeof conf[Language];

// リテラル型
const conf = {
    en: "Are you sure?",
    fr: "Êtes-vous sûr?",
    es: "está seguro?",
    ja: "よろしいですか？"
} 
type Language = keyof typeof conf;
type ConfirmationMessage = typeof conf[Language];
```

# 配列から型を生成する
- 例えば次のようなことを実現したい
```typescript
const currencies = ["CNY", "EUR", "GBP", "JPY", "KRW", "USD"];
// これが欲しい
type Currency = "CNY" | "EUR" | "GBP" | "JPY" | "KRW" | "USD";
```
```typescript
// ダメな例１
type Currency = typeof currencies; // type Currency = string[] 
// ダメな例２
const currencies = ["CNY", "EUR", "GBP", "JPY", "KRW", "USD"] as const;
type Currency = typeof currencies;  // type Currency = readonly ["CNY", "EUR", "GBP", "JPY", "KRW", "USD"]
```
- `typeof`のインデックスはリテラル型であり，配列は number 型のインデックスに要素を代入しているオブジェクトであると考えると，`number`を使うことで所望のユニオン型を取得可能
```typescript
const currencies = ["CNY", "EUR", "GBP", "JPY", "KRW", "USD"] as const;
type Currency = typeof currencies[number];  // type Currency = "CNY" | "EUR" | "GBP" | "JPY" | "KRW" | "USD"
```

# 配列から全要素の型を生成する
- 配列から型を生成する際のリテラル型の部分を以下のように書き換えることで，number 型が数値のリテラル型のワイルドカードとしてふるまうことが分かった
```typescript
type Currency = typeof currencies[0 | 1 | 2 | 3 | 4 | 5];

// 以下のように一般化が可能
type List = (string | number | boolean)[];
type Elem = List[number]; // type Elem = string | number | boolean

// 以下でも同じ結果が得られるがアンチパターン
// 理由は List が配列型でなくタプル型であると混乱させてしまう可能性があるから
type Elem = List[0];
```

# メモ
- スケープゴート：身代わり，生贄（ある集団に属する人がその集団の正当性と力を維持するために、特定の人を悪者に仕立てあげて攻撃する現象）