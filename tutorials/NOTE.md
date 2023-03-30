# 作って学ぶTypeScript

## 開発環境の準備
- Node.js は V8 (Chromeと同じ)エンジンで JS を動かす
- Node.js は TypeScript コンパイラ（tsc）を動かすのに必要
- VSCode の他に **JetBrains IDE シリーズ** というものがあり，言語ごとに最適化された個別の IDE が提供されている有償のもの

## 簡単な関数を作ってみよう
- `node increment.js`とすると`console.log()`が見れる
- `tsc increment.ts`とすることで，実行用の`increment.js`が作成される


## Reactでいいねボタンを作ろう
- React の３大特徴（仮想 DOM, 宣言的UI, コンポーネントベース） 
- `npm list -g`でグローバルインストールのパッケージを参照できる
- `< Component />`みたいなやつを self-closing element っていうらしい
- `className`は初期の React が DOM に直接値をセットしていた名残らしい（DOM では HTML の class が className プロパティになっている）．現状は直接割り当てていないので技術的には改変可能だが，開発陣は現状のコードが動かんくなったら困るから慎重ではあるらしい

## Next.jsで猫画像ジェネレーターを作ろう
- pages ディレクトリ直下のものは page component と呼ぶらしい
- 関数宣言で書いてもいいが，NextPage 型で注釈を付けられるのがいい
- async 関数の戻り値の型を指定することで，存在しないプロパティへのアクセスをしないようにコードを改変できる
- **サーバーサイドからデータが返ってこない可能性などをちゃんと考慮するなら，レスポンスをチェックするコードを書くのが望ましい**
- **useEffect に非同期関数を直接渡せない**
- JSX のマスタッシュでは文が使えないので，論理演算子や参考演算子のような式しか入れられない
- SSR の実装のために getServerSideProps, getStaticProps, getInitailProp (Next の v9 まで)　を使用

## Vercelにデプロイしてみよう
- vercel の利点（プル陸単位で自動プレビュー環境を利用可，JS, CSS ファイルを自動圧縮して CDN 配信）

## Jestでテストを書こう
- 必要なのは，`jest`（本体）， `ts-jest`（TSに対応するためのもの），`@types/jest`（Jest の API の型定義ファイルで，テストコードの型チェックができるようになる）
- `yarn ts-jest config:init`で`jest.config.js`を生成
- `.test.ts`, `.spec.ts`がテストできるファイルの拡張子
- `yarn jest`でテスト実行
- テストをしたい関数は export して，テストを書くファイルで import して使う
- Jest では expect 関数と matcher（expect 関数に生えているメソッド） を使用し，結果が期待している値になっているかを記述する
- `toBe`マッチャーは`===`と同じ

## Reactコンポーネントのテストを書こう
- 例によって create-react-app でプロジェクト作成
- `testing-library`を使うことで React コンポーネントをテストできる
- 「コンポーネントのテストは描画した後に，コンポーネントに操作を行い，状態を確かめる」ことでテスト可能
- 具体的には，`render`関数で DOM を取得し，そのテキストが OFF という文字列に等しいかを確認する
- `jest-dom`が持つ`toHaveTextContent()`でテキストを抽出できる
- クエリ，アサーション，ユーザイベントなどの用語は testing-library の定義を確かめて
- スナップショットテスト（コンポーネントの一部ではなく，全体を確かめるテストで，コンポーネントを一旦作り終えるまでは機能しないテストなのでテストファースト開発には不向き）
- 具体的な手順は１．検証したい状態にコンポーネントを持っていき，２．スナップショットに照合する
- 実際に，`yarn test`と実行すると，`__snapshot__`ディレクトリ直下に`SimpleButton.test.tsx.snap`というファイルができ，React起動時に自動生成されるDOMなどの情報が入る
- 二回目以降はこれと比較して，おかしなものが出来ていないかを検知してくれるらしい

## Prettierでコード整形を自動化しよう
- prettier [ファイル or ディレクトリ] で対象をすべて自動整形する（`--write`をつけることで書き換えまで行ってくれる）
- オプションでルールをいじることも可能（`--no-semi --tab-width 4`など）
- ふつうは，設定ファイルを書く（`.prettierrc`, `prettier.config.js`, `.prettierrc.yaml`, `.prettierrc.toml`など）
- `prettier-ignore`コメントを残すとその直下は prettier が無視される
 
## ESLintでTypeScriptのコーディング規約チェックを自動化しよう
- 公開されている規約で有名なものは３つ（Google, JS, Airbnb）
- ESLint は自動修正機能も持つ
- 心理的安全性も確保できる
- コンパイラも型や構文エラーの指摘を行うが，本質はコードの変換にあり，リンターはパフォーマンスやセキュリティ，コーディングスタイルの指摘も行う相互補完的な役割

## `.eslintrc.js`という設定ファイルについて
- eslint コマンドを実行したディレクトリを起点に遡って設定ファイルを探していく仕様（複数の設定ファイルを見つけると内容をマージしてしまう）
- これを避けるため，`root: true`としておくとこのファイル以上の階層を探しに行かなくなる
- JS / TSがどの実行環境で使用されるかを`env`で定義する（`browser: true`でwindow, alert が認識され，`es2021`で2021 までに導入されたグローバル変数が認識可能）
- `parseOptions`ではどの構文を使っているかを伝える（`ecmaVersion`はどのバージョンのES構文までokかを伝える（デフォルトでは5なので必ず設定すること）．もし`env`で設定済みの場合は，省略も可能）（`sourceType`はモジュール(import/export)を使うかどうかを指定でき，ふつうは使うので"module"と書く）
- 他にも`rules`を追加できる（`no-console`（console.log は書かない）, `camelcase`（変数名は camel case を使用すること）, `semi`（セミコロンは省略しない）など）
- ルールには重大度を割り当てることができ，off(0) -> warn(1) -> error(2) の３つを設定可能
- ものによってはキャメルケースを割り当てないなども可能にしたい場合は，細かく設定も可能
- `--fix`オプションをつけることで，自動修正が可能（`npx eslint src --fix`）
- ルールが多すぎて調べて導入するには高コストなので，`sharerable config`がおすすめ
- 今回は，`eslint-config-airbnb-base`を適用
- 一部分のルールを無効化するには，`eslint-disable-next-line [ルール名]`でコメントアウトするとよい

- typeScript で行うには追加の設定が必要
- `parser`では構文解析用の parser を設定する（これがないと TS の解釈が不可能．この parser で JS もまとめてパース可能）
- `plugins`では第三者のルールを追加できる（ここでは TS　のルールを追加したいので導入している）
- `parserOptions`はここでは，TS ESLint のオプションを書いている（`tsconfigRootDir`にプロジェクトルートの絶対パス，`project`にはコンパイラの設定ファイル（tsconfig.eslint.json）を`tsconfigRootDir`からの相対パスで示す）
- `ignorePatterns`は ESLint のチェック対象外のファイル．ディレクトリを指定する（TS のプロジェクトでコンパイル後に生成される JS は凛としないので dist は対象外にしている）
- `extends`では shareable config を使う（まず JS のルール -> TS のルール -> TS ESlint が推奨するルールセットの順に追加）
- `rules`では，shareable config の上書きに使っている
- VSCode に拡張機能を入れるとコマンドなしでリアルタイムに指摘してくれる

## huskyでコミット前チェックを自動化しよう
- 執筆中らしい．．．