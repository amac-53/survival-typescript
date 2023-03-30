# オブジェクト指向

## クラス (class)
- **オブジェクトのひな型**を定義したもので`class`構文で定義し，`new`キーワードでオブジェクトを生成可能
- **TS ではクラス定義と同時にクラス名と同じ型が定義されるため，型注釈はクラス名で行える（`const person: Person = new Person();`）**

### コンストラクタ (constructor)
- `new`したときに実行される関数で，インスタンスのプロパティを初期化する処理を実装する
- **戻り値はインスタンスなので，型注釈不可能**
- コンストラクタに引数を渡すときは，`new`宣言時
```typescript
class Person {
    constructor(name: string) {
        // 処理  
    }
}

// コンストラクタ呼び出し
const person: Person = new Person("Alice");
```

### フィールド (field)
- インスタンスにフィールドを持たせるには，インスタンス化したオブジェクトのプロパティに値を代入する（TS では未定義のフィールドにアクセスするとエラーを吐くので注意）
- フィールド宣言時に型を省略してもコンストラクタで代入する場合は型推論される
- `strictNullChecks`と`strictPropertyInitialization`がともに有効の場合は，フィールドを`new`時に定義できていないと`undefined`になるため，コンパイルエラーとなる
- 上記のコンパイルオプションをかいくぐるには，`string | undefined`のようなユニオン型にする必要がある
```typescript
class Person {
    name: string | undefined;  //avoid error
}

const person: Person = new Person();
console.log(person.name);  // undefined
```
- フィールドの初期化はコンストラクタの引数を用いてもいいし，静的な初期化も可能（クラス内部でハードコーディング）
```typescript
class Person {
    private name;  // コンストラクタで型推論される
    construcor(personName: string) {
        this.name = personName;
    }
}

const alice: Person = new Person("Alice");
```

### メソッド (method)
- メソッド構文を用いる（後は関数宣言と同じ）
```typescript
class Greeter {
    greet(name: string): string {
        return `Hello, ${name}!`;
    }
}
```

### フィールドの初期化子 (initializer)
- 初期化子を用いることも可能（クラス宣言内で`x: number = 0`のように書き，**インスタンス化されるときに自動で実行されるため，コンストラクタで初期化するのと同じ**）
- 型が自明の場合は推論が働く

### アクセス修飾子 (access modifier)
- `private`, `public`, `protected`がある
- **何も付けなければ，`public`として扱われる**
- `private`は自分のクラスのみ，`protected`は自分のクラスとそのサブクラスからアクセス可能（つまりインスタンス化されたオブジェクトからはアクセスできない）
- `class Animal{}`を継承した`class Gorilla extends Animal{}`内でアクセスできるのが`protected`という認識で，`super.親クラスのメソッド`等の形で呼び出す
- **継承時には元々`protected`だったクラスを`public`にすることのみできる（緩める方向への変更は可能）（private はそもそも継承できないので議論の余地もない）**
```typescript
class Animal {
    public name: string;

    constructor(theName: string) {
        this.name = theName;
    }

    protected move(distanceInMeters: number): void {    
        console.log(`${this.name} moved ${distanceInMeters} m.`);
    }
}

class Gorilla extends Animal {

    // 継承パターン
    move(distanceInMeters: number) {
        super.move(distanceInMeters * 10);
    }

    // 緩く上書きパターン
    // public move(distanceInMeters: number): void{
    //     console.log(`${this.name} moved ${distanceInMeters} m.`);
    // }
}

const gorilla = new Gorilla("速いゴリラ");
gorilla.move(10);  // "速いゴリラ moved 100m."
```

### constructor shorthand
- **メソッドの引数にはアクセス修飾子を設定できないが，コンストラクタは可能**
- 元々フィールドにアクセス修飾子を付けた場合とそうでない場合とで機能は全く同じで，プロパティの宣言を省略できるというだけの機能

### クラスのreadonly修飾子
- フィールドに`readonly`修飾子をつけることで読み取り専用にできる
- **値を代入できるのは，コンストラクタかフィールド初期化子のみ**
```typescript
class Octopus {
    readonly name: string;
    readonly legs = 8;

    constructor() {
        this.name = "たこちゃん";
    }
}
```

### クラスはオブジェクト
- オブジェクト自体はプロパティの集合体なので，その性質を持つ
- JS ではプロパティを追加したり，変更したりを自由に行えるが，TS はこの拡張に制限を加えている（**`public`, `private`の制限や存在しないプロパティにアクセスできないなど**）

### 静的フィールド (static field)
- 通常のフィールドは**インスタンスのプロパティ**だが，**クラスのプロパティ**とaして静的フィールドが存在する（Java, PHP では static で宣言可能）（**インスタンスを作らずにプロパティ，メソッドを使用できるため便利な側面がある**）
- **通常の JS にはこの機能は存在しないが，クラスのフィールドを後から追加することで似たようなことをできるように一応なっている**
- 一方で，**TS は`static`キーワードでこれを実現する（型推論が働くので省略も可能）（`public static`, `static readonly`などと組み合わせることも可能）**

### 静的メソッド (static method)
- 静的フィールドと同様にクラスに属するメソッドという概念で静的メソッドというものが存在し，他の説明は静的フィールドと同様

### メソッド戻り値のthis型とメソッドチェーン
- メソッドチェーンという技を用いて可読性の高いコードを提供することが可能
- 例えば，四則演算するメソッドをそれぞれ４つもったクラスを考えると，４行の記述をしてはじめて四則演算１通りを実現できるが，これは面倒なので１行で書きたい
- このようなときに，全てのメソッドの戻り値を`this`にすると実現できる
- **型注釈にクラス名を書くが，もし継承をしたものまでチェーンさせたいなら戻り値の型注釈も`this`に変えると継承後のメソッドもチェーン可能**
```typescript
class Operator {
    protected value: number;

    public construcor(value: number) {
        this.value = value;
    }

    // 戻り値を Operator にしていると継承したときに不便
    public sum(value: number): this {
        this.value += value;
        return this;
    }

    public substract(value: number): this {
        this.value -= value;
        return this;
    }
    // 残りも同様
}

class NewOperator extends Operator {

    // ここ重要
    public constructor(value: number) {
        super(value);
    }
    public power(value: number): this {
        this.value **= value;
        return this;
    }
}

const op: NewOperator = new NewOperator(2);
op.power(3).multiply(2).power(3); // 4096
```


### クラスの継承 (inheritance)
- `class Gorilla extends Animal{}`のように`extends`を使う
- **サブクラスにコンストラクタを書く場合，親クラスのコンストラクタは必ず呼び出す必要がある（`super(// 引数があれば );`）**

### instanceof演算子
- 特定のクラスのインスタンスかどうかを調べる **JS の演算子**
- **継承関係もチェック可能で，サブクラスは親クラスのインスタンスであるという扱いになり，抽象クラスのサブクラスのインスタンスも抽象クラスのインスタンスと判定される**
- クラスのインスタンスでない場合を絞りたい場合，`if(!(a instanceof ClassA)){}`のようにすればよい

### 公称型クラス
- クラスに１つでも**非 public なプロパティが存在すると公称型になる**（**つまり１つでも private や protected なものがあると同じインターフェースであっても相互に代入できないということ**）
```typescript
class UserId {
    private readonly id: string;
    constructor(id: string) {
        this.id = id;
    }
}

class GroupId {
    private readonly id: string;
    constructor(id: string) {
        this.id = id;
    }
}

// これはインターフェース同じはずなのにエラー（公称型）
const userId: UserId = new GroupId("tmpmwfwv");
```

### 抽象クラス (abstract class)
- 直接インスタンス化しないクラスで，必ず継承して使う（`abstract class Food`）
- 抽象メソッドも宣言でき，これはサブクラスにも定義しないといけない
- **コンパイルすると，抽象クラスは`abstract`宣言が取り払われ，抽象メソッドは存在そのものを抹消される**

### finalクラス・sealedクラス
- これらのクラスは JS, TS では存在しないので注意
- final クラスは継承できないクラスで，意図せずサブクラスが生まれるのを抑制する（PHP, Java）
- sealed クラスは同じモジュール内であれば継承できるが，その枠を越えて継承できないクラス（Scala, Kotlin）

--------------------------------
## インターフェース (interface)
- **クラス**が実装すべきフィールドやメソッドを定義した TS 独自の型
- これを用いることでインターフェースが求めるメソッド名や引数の型に則っているかを確認できる
- TS のインターフェースはコンパイルチェックに活用された後，JS コード生成過程で消されるので実行時に影響することはない
- （基本は）TS は構造的部分型なので，型注釈としてどのオブジェクトにも適用できる（クラスでの使用にとどまらないということ）

### インターフェースを実装する
- 実際にクラスがインターフェースを実装するときには`implements`を用いる（複数のインターフェースを実装可能）
```typescript
interface Human {
    name: string;
    think(): void;
}

interface Programmer {
    writeCode(): void;
}

class TypeScriptProgrammer implements Human, Programmer {
    name: string = "Bob";
    think(): void {
        console.log("どういうコードにしようかな");
    }
    writeCode(): void {
        console.log("カタカタ");
    }
}
```

### インターフェースのreadonly修飾子
- インターフェースにも`readonly`修飾子をつけることが可能

### インターフェースの継承 (inheritance)
- クラスと同様に`extends`キーワードで継承する
- 元のプロパティの情報はすべて引き継がれ，新たなプロパティの追加や既存プロパティの型を**部分型に指定し直すことも可能**（全く異なる型はダメな点に注意）
```typescript
interface Person {
    age: number | undefined;
}

interface Student extends Person {
    age: number;
}
```

### オープンエンドと宣言マージ (open-ended and declaration merging)
- TS では同じ名前のインターフェースを定義してもエラーにはならない（**オープンエンド; open-ended**）
- TS では同じ名前のインターフェースを宣言するとマージされる（**宣言マージ; declaration merging**）
- 実はこの仕様は TS の開発元がバージョンアップごとにコピーコードを量産しないように工夫した結果となっている（例えば ES2019環境の型定義ファイルが欲しいなら，ES2019までの必要な型定義ファイルを読み込む必要がある）

### インターフェースとinstanceof（後で）
- PHP 等と異なり，インターフェースに instanceof を使えない点に注意（というのも，インターフェースはただの空っぽな型情報でしかないため，JS の実行時にはインターフェースはすでになくなっているから）
- 実際にインターフェースと値が互換しているか否かを判定するには，型ガード関数を用いる（最初に`null`でない and`object`を判定 -> それぞれのプロパティの型が違反していないかを判定）
- 型ガード関数ではプロパティごとに型判定を行うロジックが入っているので，大規模なオブジェクトだと面倒．そこで`zod`を用いる（オブジェクト定義時にスキーマを`zod`を用いて作成してしまう）

### interfaceとtypeの違い
- 継承を行えるのはインターフェースのみ（インターフェースは型エイリアスの継承も行えるという点に注意）
- 上書きする際に，インターフェースは部分型や追加ができる一方で，型エイリアスは交差型（`&`）で無理やり上書きができる上，矛盾する型があっても`never`型になるだけでエラーにならない
- 同名のものは，インターフェースは矛盾がない限りはマージされる一方で，型エイリアスは許されない
- 型エイリアスでは`Mapped Types`を使用可能
- **使い分けに関してだが，実のところ明確な正解はなく，プロジェクトごとに決定するのが一般的**
- **参考例として，Googleのスタイルガイドでは，プリミティブやユニオン，タプルは型エイリアス，オブジェクトの型定義はインターフェースを使うことを推奨している**
- **型の構造をアプリケーション側に割とゆだねたいライブラリ開発であれば，拡張性を考慮してインターフェースを使うのが良い**

--------------------------------
## JavaScriptとTypeScriptのオブジェクト指向機能の比較
- 抽象クラス，抽象メソッド，final /sealed クラス，final メソッドなどの機能拡張が TS では行われている