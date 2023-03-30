# 非同期処理
- JS はシングルプロセス，シングルスレッド
- API リクエストの処理待ち時間を有効利用するため，ノンブロッキングを扱う
- ES2015から追加された`Promise`により，コールバック地獄から解き放たれる
- `async/await`が ES2017で追加されたことでより書きやすくなった

## Promise<T>
- Promise がない時代に，2つの API のリクエスト結果を順に受け取り，それを次のリクエストに使用する場合の実装を行うと，以下のようなコールバック地獄を生む
```javascript
function request1(callback){
  setTimeout(()=>{
    callback(1);
  }, 1000)
}

function request2(result1, callback){
  setTimeout(()=>{
    callback(result1 + 1);
  }, 1000)
}

// 処理
request1((result1) => {
  request2(result1, (result2) => {
    console.log(result2);
  })
})
```
- TS で Promise を使う場合は，`Promise<T>`ジェネリクスを用いる（`T`は`fulfilled`のときに返される型を指定する）
- ジェネリクスは省略不可能で，型が合わない場合もコンパイルエラーになる
- **Promise の基礎として，コンストラクタのコールバック関数の引数に`resolve`, `reject`を渡すことが可能で，`resolve`が実行されると then が実行され，`resolve`に渡した引数は`then`で受け取ることができる**
- **また，`reject`が実行されると，catch が実行され，catch の引数に`reject`の引数を渡すことが可能**
```typescript
type User = {
  name: string;
  age: string;
};

function getUser(): Promise<User> {
  return new Promise((resolve) => {
    const user: User = {
      name: "太郎",
      age: 10,
    }
    resolve(user);
  })
}

// 処理
getUser().then((user: User) => {
  console.log(user);
})
```
- 覚えるべき３メソッド：
1. `Promise.prototype.then()`は`fulfilled`のとき呼び出され，**コールバック関数の引数は`Promise<T>`の`T`, 戻り値として，`S` or `Promise<S>`を返すと`Promise<S>`を返してくれる**
  - コールバック内で例外を投げると`rejected`される
```typescript
Promise.resolve(1)
  .then(() => {
    throw new Error();
  })
  .then(() => {
    console.log("fulfilled");
  })
  .catch(() => {
    console.log("rejected");
  });
```
1. `Promise.prototype.catch()`は`Promise<T>`が `rejected`されたときに呼び出され，コールバック関数内の第一引数は any 型になる．これも`S` or `Promise<S>`を返すと`Promise<S>`を返してくれる
    - **catch のあとに then をつなげると catch が実行されたときとそうでないときの型を then に伝える必要があり，面倒なのでふつうは逆順で書く**
2. `Promise.prototype.finally()`は`Promise<S>`が`settled`のときに呼び出され，コールバック関数に引数はなく戻り値を設定することも不可能（ES2018追加）

- 静的メソッドととして，以下がある
1. `Promise.all()`は第一引数に要素が`Promise`の配列を取り，実行結果を順番通りに返す（**１つでも拒否されると拒否される`**）
2. `Promise.resolve()`は履行(fulfilled)された`Promise`を返す
3. `Promise.reject()`は拒否(rejected)された`Promise`を返す
4. `Promise.allSettled()`は ES2020 以降のもので，`Promise.all()`同様に第一引数に Promise の配列を受け取るが，全ての結果が settled になるまで実行する
5. `Promise.race()`は`Promise.all()`と同様に第一引数に Promise の配列を受け取るが，最も早く決定された Promise の結果を（fulfilled, reject に関係なく）返す
- Promise には`pending`, `fulfilled`, `rejected`の３状態が存在し，`fulfilled`と`rejected`を合わせて`settled`ということもある

## 🚧async
- 

## 🚧await
- 

# 参考
- [コールバック地獄からの脱出](https://qiita.com/umeko2015/items/2fdb2785eac8f4117f23)
- [Promiseに関して](https://zenn.dev/oreo2990/articles/bd1bf724d718a4)