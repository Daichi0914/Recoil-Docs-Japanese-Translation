# インストール

## NPM
Recoilパッケージはnpmとyarnにあります。
最新の安定版をインストールするには、次のコマンドを実行します。

npmを使用している場合:

```terminal
npm install recoil
```

yarnを使用している場合:

```terminal
yarn add recoil
```

### Bundler
NPMを介してインストールされたRecoilは、WebpackやRollupなどのmodule bundlersとうまく組み合わせられます。

### ES 5サポート
RecoilビルドはES 5にトランスパイルされず、ES 5でのRecoilの使用はサポートされていません。
ES 6を基本として提供していないブラウザをサポートする必要がある場合は、Babelでコードをコンパイルし、`preset@babel/preset-env`を使用してください。
ただし、これはサポートされていないため、問題が発生する可能性があります。

特に、Reactと同様に、Recoilは、ES 6の`Map`と`Set`のタイプやその他の機能に依存します。polyfillsを使用してこれらの機能をエミュレートすると、パフォーマンスが大幅に低下する可能性があります。


## CDN
バージョン0.0 .11以降、Recoilは`<script>`タグで直接使用できるUMDビルドを提供し、シンボルRecoilをグローバルネームスペースに公開しています。新しいバージョンからの予期しない破損を避けるために、特定のバージョン番号とビルドにリンクすることをお勧めします。

```html
<script src="https://cdn.jsdelivr.net/npm/recoil@0.0.11/umd/recoil.production.js"></script>
```

CDN上のすべてのRecoilファイルは、jsdelivrで参照できます。

## ESLint
プロジェクトで`eslin-plugin-react-hooks`を使用している場合。たとえば、次のようなeslint設定では、

```json
// previous .eslint config
{
  "plugins": [
    "react-hooks"
  ],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

`additionalHooks`のリストに'`useRecoilCallback`'を追加することをお勧めします。
この変更により、ESLintは`useRecoilCallback()`に渡された依存関係が誤って指定された場合に警告を出し、修正を提案します。
`additionalHooks`のフォーマットは正規表現の文字列です。

```json
// modified .eslint config
{
  "plugins": [
    "react-hooks"
  ],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": [
      "warn", {
        "additionalHooks": "useRecoilCallback"
      }
    ]
  }
}
```