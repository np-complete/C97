# ライブラリ

プログラミング言語による文化の違いが強く現れるのがライブラリです。

## Rubyの場合

Rubyの世界では、紆余曲折があった結果、

- ライブラリ自体を **Gem**
- 中央集権的なGem配布リポジトリである **Rubygems**
- アプリケーションに必要なGemを管理する **Bundler**
- 利用ライブラリを専用DSLで記述する **Gemfile**

のセットで使われるようになりました。
もちろんRailsなどもRubygemsで配布され、Bundlerで依存を管理します。
Rubygems と、最近 Bundler もRuby本体に同梱されるようになり、
おそらくRubyの寿命が尽きるまで大きく変わることはないでしょう。

## 他の言語の場合

最近はどの言語も **中央集権リポジトリ** があり、
コマンドでライブラリをインストールするのはほぼ共通体験のようです。
一番の特徴は、依存を定義するファイルが **依存定義専用** か、それとも他の情報の一部かというところにありそうです。

### Nodeの場合

Nodeの `npm` も言語に組み込みですが、 `yarn` などの代替実装もあり混乱しています。
また、 `yarn add` などのコマンドを通して `package.json` を更新することも非常に珍しい特徴だといえます。

ライブラリは作業ディレクトリの `node_modules` 以下にインストールされ、プロジェクト間でライブラリ群を共有することはしません。
使っていないライブラリはガシガシ消されてしまうので、例えばブランチを行ったり来たりするたびに `yarn install` をしないといけないことがよく起こり、とてもめんどくさいです。

### PHPの場合

PHPには、ライブラリのようなものがいくつもあります。

- PHPのビルド時に一緒にコンパイルするもの
- C拡張ライブラリであるPECL
- PHPのライブラリであるPEAR

が、最近は `composer` に統一され一応現代的な依存管理ができるようになりました。
とはいえ、 **ライブラリごとにバージョン指定が必要な** ひどいJSONファイルを手書きしないといけないというつらい仕様のようです。

### Pythonの場合

Pythonは、**慣習的に** `requirements.txt` というファイルに利用ライブラリを列挙し、 `pip install -r requirements.txt` のコマンドでライブラリをインストールするようです。
バージョン固定したりもできるようですが、ただのテキストファイルで構造化されていたりDSLだったりしません。
ちょっとアレですがたしかに必要十分と言えるかもしれません。

### GOの場合

GOは非常に特殊で、ライブラリ配布の中央集権サーバを持っていません。
元々、 `go get` でGithub上のライブラリを取得し、ソースコード上にリポジトリURLを指定する、というめちゃめちゃマッチョな方法でライブラリを扱います。
`Gemfile` や `package.json` があるわけでもないので依存ライブラリが一つのテキストファイルにまとまっていることすらありません。

最近 `dep` と言うツールが登場し、ライブラリ管理の世代交代があったようです。
そのマッチョな方法から、複数のプロジェクトから同じディレクトリが参照され、さらにバージョン指定もできないので **ライブラリの破壊的変更に弱い** という問題がありました。
`dep` は、`node_modules` のように **プロジェクト毎にライブラリを持てる** ようになり、さらに **バージョン指定** も可能にしこの問題は解決されたようです。

### Javaの場合

Java世界では主に `gradle` や `maven`、 Scala では `sbt` が使われますが、これらは **ビルドシステム** であり、ビルドの定義の中に依存ライブラリの記述がある、と言う世界観のようです。
ビルドシステム自体は言語のツールセットに **同梱されていません** 。
また、 `maven` はビルドツールですが、中央集権リポジトリの名前でもあります。
`gradle` もmavenリポジトリを参照しに行くよう設定することがほとんどのようです。

ビルドの定義ファイルは **非常に複雑** で、まぁよくこんなの書いてられるなと感心します。
Javaの世界って何もかも拡張性とか普遍性とかそういうお題目で複雑怪奇で何でもできるツール作りたがるのがすごく嫌いです。

もっと昔の、 **jarファイルを手でダウンロードしてきてビルドパスに入れる** とかの時代よりかは数億倍マシですが。

## まとめ

最近はどの言語も **中央集権リポジトリ** が存在し、依存関係を **テキストファイルで管理** し、コマンドでインストールするところはほとんど同じです。
ただ、Rubyで使われる **Gemfile** は、**専用のDSLで依存関係のみを記述する** という点で特徴的です。
記述は非常にシンプルで書きやすく、DSL言語と言われるRubyの本領発揮を感じます。

また、地味だけど非常に重要な概念として、 Rubyの **Gem** に当たる言葉は他の言語には無いのではないかと感じました。
「Gemをインストールする」、「Gemが更新された」、「良いGem知らない?」 ものすごく単純なことだけど、シンプルな単語で確実な意思疎通ができるのは非常に重要です。
Rubyistの帰属意識の高さはこのあたりの語彙にあるのではないかと思いました。

「npmパッケージをリリースする」と「Gemをリリースする」では **ワクワク感** が全く違います。
「gem 作り方」 「npm 作り方」 「pip 作り方」 のワードでググると、 **Gemは検索結果がひと桁多い** のもそのあたりが関係しているのではないでしょうか。