# 今こそNext.jsの出番が<span>来たぞという話</span>

## Next.jsとは？
- React製のSPA総合フレームワーク
- と言いつつ、Nuxt.jsよりは機能・規約が少なめ。基本的にはReactでSSR・静的HTML生成がしたいときに使うという考え方でよい

## なぜいまNext.js?
- CLでいま導入されていることが多いのは圧倒的にNuxt.js。Nuxt.jsの方が後発で多機能。
- **TypeScriptとの相性が圧倒的によい** という話に尽きる。型の恩恵を受けたければVueよりReactだし、NuxtよりNext。
- 先日の[Next.js 9のリリース](https://nextjs.org/blog/next-9)で、Nuxt.jsで評判が良かった機能の導入や、TypeScript導入のサポートが進み、実戦投入しやすくなった。

## 試したいときは？
- [getting started](https://nextjs.org/learn/basics/getting-started)
- 必要なものをだいたいそろえたボイラープレートを作ったのでこちらも使ってみてね
    - [https://github.com/fnobi/hinagata-next](https://github.com/fnobi/hinagata-next)

## いまどきのReactの書き方
- 基本ぜんぶ関数(Functional Component)で書くのが正義
    - 後述のgetInitialPropsを使うときはクラスで書くほうが個人的に好き
- React Hooksの機能を使うと、関数の形で書いてもキレイにstateを持たせることができる
- 適当に書くと再描画の負荷がめっちゃ高いコンポーネントができてしまうので注意
    - `useMemo` `useEffect` を積極的に使っていこう！
        - Vueで言えばそれぞれcomputed / mounted,watchにあたると思えばだいたいあってる
    - このあたりは喋りすぎると長くなるのでほどほどに

## 静的HTML Exportについて
- `next build && next export`
    - ビルドして.nextファイルを吐き出してからexportする必要がある（Nuxtと違ってコマンド一発じゃない）ところに注意
- デフォルトでは、 `pages` ディレクトリ以下に置いたコンポーネントが、ディレクトリ構成を保ったままhtmlとして吐き出される
    - コンポーネントの初期状態が、ちゃんとHTMLとして現れるので、SEO的にもばっちり
- `next.config.js`で`exportTrailingSlash`というオプションをオンにすると、ファイルが２つ分つくられる（about.html, about/index.html)
- SSR = APIから値を取ってきた結果を出力するHTMLに組み込みたい場合、コンポーネントに`getInitialProps`というstaticメソッドを追加する（Next.js専用のReact Component拡張仕様）
    - ここにAPIからデータを取ってくる感じの非同期処理を書いて、returnするとrender処理の際propsとして渡される
- ディレクトリ構成とは違う形で出力したいとき・`/post/:id` みたいな動的ルーティングを組み込みたいときは、`exportPathMap`という処理を`next.config.js`に書く。
    - 動的変わるパス名に応じてコンテンツを出し分けたいというときは、これまたコンポーネント側に`getInitialProps`を書く必要あり。要はexport時点でのrenderに必要な情報はぜんぶ`getInitialProps`で取ってくる必要がある。
- もろもろのサンプル
    - [https://github.com/fnobi/hinagata-next/pull/13/files](https://github.com/fnobi/hinagata-next/pull/13/files)

## SPA開発環境によく突っ込むものをそろえていく
- TypeScript
    - pages以下に置くコンポーネントの拡張子をtsxにする。以上。
- css
    - emotionというフレームワークがおすすめ（いえもんが書いてるの見て知った）
        - cssをオブジェクト形式で書いて、React DOMに紐付けていく
        - cssについても型補完が効くので、typoが減るぞ！
        - [https://github.com/emotion-js/emotion](https://github.com/emotion-js/emotion)
- lint
    - TypeScriptでもeslintでOK
    - 個人的には、webpack設定を拡張してsave時にlint & fixしてしまうのがおすすめ
- メタ系設定
    - `Head`というよしなコンポーネントをNext側で用意してくれている。どこで書いてもheadタグに入れたいもの突っ込んでくれる。
        - [https://nextjs-docs-ja.netlify.com/docs/#head-への挿入](https://nextjs-docs-ja.netlify.com/docs/#head-%E3%81%B8%E3%81%AE%E6%8C%BF%E5%85%A5)
### store

- まあ順当にreduxを入れるのが正攻法
    - Next側で、繋ぎ込み用のヘルパーnpmは用意してくれているものの、導入時はそれなりにコード書く必要がある

- ただ自分は正攻法のredux入れるのに無駄な工程が多すぎてキレてしまったのでstoreを独自実装するという暴挙に出た
    - 実際reduxはもともとjsで動かす前提で設計されたものなので、TypeScriptで組む場合はここまで細かい規約いらなくない…？そうおもわない…？
    - lib: [https://github.com/fnobi/hinagata-next/blob/master/src/lib/TypeRegi.ts](https://github.com/fnobi/hinagata-next/blob/master/src/lib/TypeRegi.ts)
    - store定義: [https://github.com/fnobi/hinagata-next/blob/master/src/store/sample.ts](https://github.com/fnobi/hinagata-next/blob/master/src/store/sample.ts)
    - 使用例: [https://github.com/fnobi/hinagata-next/blob/master/pages/index.tsx#L28](https://github.com/fnobi/hinagata-next/blob/master/pages/index.tsx#L28)

## おわり