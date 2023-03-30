# Advanced Topics

## NPMパッケージ開発者のためのtsconfig
- 一旦不要なので飛ばす

## デュアルパッケージ開発者のためのtsconfig (Dual Package)
- 一旦不要なので飛ばす

## ジェネレーター (generator)
- アスタリスクを付けて表記する（`function*`, `public *method_name`）
```typescript
function* generatorFunction1(){

}

const generatorFunction2 = function* (){

}

class GeneratorSample {
    public *generatorFunction3(){

    }
}
```
- Generator は`IterableIterator<T>`を実装したクラスのオブジェクトのことで，Generator関数内で呼ばれたときに（呼び出し元へ値を返却し，次に呼ばれたときはその続きから処理を開始する）`yield`キーワードを使える
- `Promise`が一般化する前は代わりに非同期処理を担当していたが，**現在では大量のデータを小出しにしたいときに使い道がある**らしい


## セッターとゲッター (set, get)
- プロパティへのインターセプター（参照・代入・監視）として存在する
- メソッドと異なり，呼び出すときに`()`は不要
```typescript
class Human {
    private _name: string;

    public constructor(name: string){
        this._name = name;
    }

    // Getter 戻り値の型は必ず必要
    get name(): string {
        return this._name;
    }

    // Setter 引数は１つは必ず必要
    set name(name: string){
        this._name = name;
    }
}

const human = Human("");
// setter
human.name = "太郎";
// getter
console.log(human.name);
```