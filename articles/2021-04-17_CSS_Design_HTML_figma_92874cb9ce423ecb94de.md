<!--
title:   コーディングしやすいFigmaデータの作り方
tags:    CSS,Design,HTML,figma,デザイン
id:      92874cb9ce423ecb94de
private: false
-->
## これは何

- UIモックアップデータを作る際、少しでもコーディングが楽になるように作る方法を書いた記事です
    - 大抵のWebサイトやアプリ制作においては、UIのモックアップデータを作り、その後コーディングをすると思います
    - そのため、見た目が同じでもデータの作り次第で作業効率が大きく違います
- この記事で取り扱うツールはFigmaですが、本筋はSketchでもXDでも変わりません
    - ツール名称などを適宜読み替えていただければSketchやXDでも使える考え方だと思います

## 対象読者

- コーディングを依頼する人
    - 主にUIデザイナー
    - チームによってはディレクターやプロダクトオーナーも該当するかもしれません
    - 作業しやすいデータ作成を理解するために読んでもらえたら嬉しいです
- コーディングをする人
    - フロントエンドエンジニア・マークアップエンジニア
    - チームによってはデザイナーも該当するかもしれません
    - 余りにもコーディングしづらいUIデータをもらったとき、この記事をUI作成者へ共有していただければ話がしやすいかもしれません

### 補足：筆者の職種

私自身はデザイナーですが、業務ではUIのモックアップも作るし、HTML, CSS, Reactなどでコードを書いています。
そのため比較的中立の立場から、どちらかの立場に寄りすぎず意見を書けるだろうと思っています。

## 要素の名付け方やレイヤーの順番などを明文化しておく

コーディングの有無に関係なく必要な工程ですが、一応この記事でも触れておきます。

Frameの命名が日本語と英語が混ざっていたり、`Frame 42`のようになっていると見やすくありません。
どのFrameがどのオブジェクトを表しているのかが理解しやすいと明確に作業スピードが上がります。

またレイヤー順は上からor下から一直線で辿れるようにしておきましょう。
人力で修正するのは大変なので、こういうプラグインを使うのが楽です。

https://www.figma.com/community/plugin/742038190980789811/Sorter

そして少し発展的ですが、variantsを使用するコンポーネントはどんな値が渡ってきてどう変化するかや、コード上で再現したときの扱いやすさを考えておけると良いです。

- アイコンがボタンのテキストの左右につくことがある場合、どちらの方が使い勝手が良いか
    - `Left Icon = True / False`と`Right Icon = True / False`
    - `Icon = 'Left' / 'Right' / 'Both' /'None'`
- `input`に対してエラーのvariantを作るとして、エラーメッセージはコンポーネントの要素として含むか否か
- TypeScriptを使っていて、プロパティ名に`type`を使いたくなったとき、ややこしくなるのは間違いないので、違う言葉を使うのかプレフィックスなどで対処するのか

いずれの項目でも「絶対にこうすべき」があるわけではなく、チーム全員が認識を揃えやすいものを話し合うのが大事です。

ちなみに筆者がよく採用するのは

- ルートのFrame名はURL
    - ドメイン名は省略する
    - 同一URLで状態が違う場合は`()`書きで補足する
    - 例：`/post/[id] (not logged in user)` [^1]
- レイヤーは下から順
    - Auto Layoutが下から順を強制してくるため、全体通して下から順 [^2]
- 要素の名前は英語で、そのままクラス名として使えるのが理想
    - 極力`[file name]__[layer name]`など機械的にクラス名として使えるようにする
- Auto Layoutが入れ子になりすぎないようにSpacerコンポーネントを用意する
    - 余白の幅の違いを表現するためだけにAuto Layoutが入れ子になりすぎると見通しが悪くなる
    - また、レイヤーのグルーピングとHTMLの構造が大きく違ってくるので感覚として気持ち悪い
- （筆者はReactを触る機会が多いので）propsの単位とvariantsの単位が揃うようにする
    - 先ほどのアイコンつきボタンの例でいうと`<Button iconPosition='left' icon='check' />`のようなコンポーネントを作るとしたら`Icon = 'Left' / 'Right' /'None'`なvariantsをFigmaで定義する（=`'Both'`は廃止する）
    - `iconPosition='both'`を許容すると`icon=[check, arrow]`などにも対応しないといけなさそうで不必要なややこしさが生まれそう→いっそ廃止する判断

[^1]: `/`区切りでFrame名をつけるとExportした際にディレクトリが入れ子になってしまうので、最近は`_`区切りなどの方が良いかもしれない……と思っている
[^2]: 上から順が良いと思う時期と下から順が良いと思う時期が定期的にやってくる

## FigmaのStylesとコードの定数を揃える

おそらくFigmaを使っていれば様々なStylesを登録していることでしょう。

- Color
- Text
- Effect
- Grid

これらの種類分けや命名をコードとしっかり揃えておくのが重要です。

例えば以下のようになっていたらドキュメント化するのも相手に伝えるのも大変ですよね。

| カテゴリー | Figmaでの命名 | コードでの命名 |
| --- | --- | --- |
| 色のカテゴリー | main, sub | primary, secondary |
| 色の展開 | main100, main200 | primary-light, primary-dark |
| 文字の大きさ | title, headline, body | font-l, font-m, font-s |

Figmaで定義した色をコード化するのは、個人的にはデザイナーが担当した方が良いと思います。
CSS custom propertiesとしてなのか、Sass Variablesなのかなど、定義の仕方も色々あると思いますがエンジニアとも話しながら決められるのが良いですね。

## プロトタイプを作る・プロトタイプで確認する

静止したページ一覧だけを見ていてもイマイチ分からない箇所は多いと思います。

- メニューが開閉する
- モーダルウィンドウがオーバーレイする
- 条件によって要素の有無が変わる

こういったものはプロトタイプを有効活用しましょう。

オススメなのは、スタートFrameにハブっぽいものを用意しておくことです。

![初回起動と2回目以降起動とで動線が違う図](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/214677/b62303a7-939a-03c8-b948-ba62c4539d85.png)

画像の例では「初回起動」と「2回目以降起動」で動線を分けていますが、色々な使い方ができます。

- 「ログイン済みユーザーからの見え方」と「未ログインユーザーからの見え方」
- 「通常版」と「ベータ版」

といった具合で状況にあわせて用意するのが良いでしょう。

## コードを想定したConstraintsやResizingを設定する

例えばAuto Layout要素において、以下の2つは全く同じ見た目になるケースがありますが、それぞれから想起されるCSSは結構違います。

1. `Fixed`の親要素の中に`Fill container`の子要素が入っている
1. `Hug Contents`の親要素の中に`Hug Contents`の子要素が入っている

```css:1番「っぽい」コード
.parent {
  width: 1000px;
}

.child {
  width: 100%;
}
```

`css:2番「っぽい」コード
.parent {
  width: auto; /* ※autoは初期値なので、何も指定していない状態と同じ */
}

.child {
  padding: 10px; /* 幅を指定しているわけではなく、コンテンツにあわせて成り行きで幅が決まるようなコード */
}
`

これを見て分かるように、要素の大小や多寡によって実際の描画結果は変わってきてしまいます。

Auto Layoutに限らず、通常のConstraintsでも同じことです。
Figma上で静的に成立しているだけでなく、実際コードで表現したときにどうなるか？それに相応しいConstraintsやResizingはどうあるべきか？を考えてデータを作りましょう。

## Figma上の表示だけで全てを解決しようとしない

「Figmaデータの作り方」というタイトルなのに、なんてことを言い出すんだという話ですが……。

いくらcomponentsやvariantsが便利といえども、全ての条件を網羅しようと思うととんでもない工数がかかってしまいかねません。
ですが、コメントで一言注釈を書けば解決する……なんてこともあります。

今のFigmaでは無限スクロールを表現しようとしても無理です。
それなりの再現度で良ければ可能ですが、完璧ではありません。
ですが「最初は20件表示、一番下までスクロールしたら追加で20件表示、以降は無限スクロール」などのコメントを入れておけば割と伝えることでしょう。
（もちろん追加で要素が出現したときのアニメーションなどは考える余地がありますが）

ツールの機能に拘らずに、コミュニケーションをとる方がより良い場合もある、と頭の片隅に置いておいてください。