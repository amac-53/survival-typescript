# tsconfig

## tsconfig.jsonを設定する
- `npx tsc --init` or `tsc --init`（TS をグローバルインストールしている場合）
- `"compilerOptions"`にある代表的な項目に触れる
### target
- どのバージョンの JS に出力するかを指定
- 例えば，`es5`を指定した場合は，アロー関数を関数宣言にコンパイルしてくれる
### フロントエンドとバックエンド
- モジュールの読み込み方法が異なる
- `module`部分を変更すべきで，バックエンドは`commonjs`, フロントは`es2015`, `es2020`, `esnext`を指定する
- 現状の target には使用したい機能がない場合は lib オプションを指定する（現時点では提案中の機能を使用できるようにするものをポリフィルという）
- lib を指定する場合，明示的にどの target の lib を使用するかも明記する（`"lib": ["es2018, "esnext.AsyncIterable"] // es2018を省くとまずい`）
- 基本的に最新の LTS を target を指定するとよいが，Babel等に任せたい場合は，`target: "esnext"`と指定することで任せることになる

## strict
- strict系オプションを一括で有効化するもので，デフォルトでは`false`（8つ存在する）
- 個々のオプションをそれぞれ有効化しているとそちらが優先される
- TS 公式は有効化を推奨している
### noImplicitAny
- 暗黙の any を禁止する（引数やプロパティを明示的に示す必要があるが，戻り値は推論可能なので警告は出ない）
- `noImplicitReturns`で戻り値の型も必要になる
### strictNullChecks
- これが false の場合`null`, `undefined`の代入がチェックされていないので，実行時エラーを許容することも
- **このオプション次第で戻り値の型に影響する場合がある**（false の場合は，`Array.find`は戻り値として`undefined`を考慮せず，Array の要素の型になるが，true の場合は`undefined`とのユニオン型になり，`getElementById()`は`HTMLElement | null`のユニオンになる）
- 型ガード関数にも影響があり，戻り値の型として，`string | undefined`としていても，`string`になる
- **期待しない null, undefined の代入は危険なので true にしておくべき**
### strictFunctionTypes
- 引数の変性をチェックしてくれる
- 広い範囲のものを渡せる**反変性**（number <- number | null），狭い範囲のものを渡せる**共変性**（number | null <- number）がある
- **TS の関数は双変性をもつ（代入する変数の型の範囲よりも広い or 狭いどちらの型のものも代入できるという性質）**
- 双変性だと狭い範囲の関数を渡した場合，元々想定していた型のものが代入できなくなるという事故が起こるため，反変性のみ許されるのがいい
- strictFunctionTypes で反変性にできるが，メソッド型には働かない（`interface Obj {method(n: number | null): any;}`はチェックされないが関数型に置き換える（`method: (n: number | null) => any;`）とチェックされる）という点に注意
### strictBindCallApply
- 関数オブジェクトのメソッドである bind, call, apply の型チェックが行われる
- デフォルトでは，`bind`, `call`, `apply`で呼び出す関数の戻り値型注釈は無視され，any になってしまう
### bind, call, apply の基礎
- そもそも this は実行時に決まる値でふらふらするので結構面倒やし，指定できたら便利なはず
- いずれも第一引数で this の参照先を指定，第二引数以降で関数自体の引数を指定可能
- call, apply は**使用時点で関数を実行**する点で bind と異なる（bind は新たな this の参照先を持った関数を返す）
- call は**第二引数以降は文字列**，apply は**第二引数は配列を指定**する必要がある
### strictPropertyInitialization
- クラスプロパティの初期化を必須にする
- `strictNullChecks`も`true`にして使う必要がある
- 初期化方法は「コンストラクタで初期化」，「初期化子で初期化」，「undefinedとのユニオン型で型注釈」の３つ
### noImplicitThis
- this の型注釈を必須にする
- 名前付き関数，匿名関数は実行時に this が決定されるので，書いている時点では`any`型と同じ扱い
- このためタイポなどで実行時エラーになることがあり，this の型を明示するように指定しない限り実行できない制限を課すことが可能に
### useUnknownInCatchVariables
- `catch(e)`の`e`を`nuknown`型として扱う
- そもそも throw はどんな型でも投げれたので any 扱いだったが，型安全を保障するために unknown にすることが可能になった
- もしこのオプションを効かせたまま any を使いたければ，明示すればよい（`e: any`）
### alwaysStrict
- JS の strict を有効にする

## noUnusedLocals

## noUnusedParameters

## exactOptionalPropertyTypes

## noImplicitReturns

## noFallthroughCasesInSwitch

## noUncheckedIndexedAccess

## noImplicitOverride

## noPropertyAccessFromIndexSignature