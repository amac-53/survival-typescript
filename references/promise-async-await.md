# Promise / async / await

- Promise は`asynchronous.md`で触れたので省略

## async / await
- Promise をより簡単に書ける構文
- 関数内で Promise が返されていなくても，戻り値の型を`Promise`で包んで返す（そのまま`Promise`を返すことも可能で２重にラップされることはない）
```typescript
async function requestAsync(): Promise<number> {
    return 1;
};

// Promise のまま書くと．．．
function requestPromise(): Promise<number> {
    return new Promise((resolve) => {
        return resolve(1);
    });
}
```
- await は Promise が解決されるまで実行を待機して解決された値を返す

```typescript
function request1(): Promise<number> {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(1);
        }, 1000);
    });
}

function request2(result1: number): Promise<number> {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(result1 + 1);
        }, 1000);
    })
}

// async, await
async function main(){
    const result1 = await request1();
    const result2 = await request2();
    console.log(result2);
}

main();
```