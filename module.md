# モジュール

ライブラリ周辺の環境は言語によらずあまり変わらないことがわかりました。
しかし、構造・使い方・作り方に関して、 **Rubyはかなり特異** だと言えます。

## 他の言語の場合

他の言語でのライブラリ読み込みを見ていきましょう。

### JavaScriptの場合

```js
// child_component.js
import React from 'react'

export default class ChildComponent extends React.Component {
  render() {
    return <h2>Child</h2>;
  }
}
```

```js
// parent_component.js
import React from 'react';
import ChildComponent from './child_component';

export defaut class ParentComponent extend React.PureComponent {
  render() {
    return (
      <div>
        <h1>Prent</h1>
        <ChildComponent />
      </div>
    );
  }
}
```

### Pythonの場合

```python
# myutil.py
import numpy as np

def mkarray(a):
  return np.array([-a, 0, a])
```

```python
# main.py
import myutil
import numpy as np

np.array([1, 2, 3]) + myutil.mkarray(3)
```

## Rubyの場合

一方Rubyでライブラリを読み込み利用するコードを書いてみます。
前者とは根本的な違いがあることがわかりますか?

```ruby
# api.rb
require 'faraday'

def google_search(word)
  Faraday.get('https://www.google.com/', q: word)
end
```


```ruby
require './api'

begin
  google_search('ruby')
rescue Faraday::BadRequestError => e
end
```

前者はライブラリを読み込む構文で **変数に受けている** のに対し、
Rubyでは **どこからとも無く** Faradayが生えてきています。
また、前者では `react` や `numpy` を親ファイル、子ファイルの両者で `import` しているのに対し、Rubyは `faraday` を一度しか `require` していません。
JavaやPHPでも同じで、こちらは変数に受けることはありませんが、ファイルごとにこのクラスを使うという宣言をしなくてはいけません。

これはつまり、Rubyの場合はプロセスのグローバル空間に読み込まれ、**グローバル変数** のように振る舞うということです[^1]。
もちろん、サンプルのように **関数を直接定義する** とグローバルに置かれてしまうので、公開ライブラリなどでは普通絶対厳禁です。

[^1]: より正確に言うと `main` というインスタンス内で変数や関数を定義しています

## それぞれの利点と欠点

JavaScriptやPythonの方法(以下 ローカル型と呼ぶ)と、Rubyの方法(以下 グローバル型と呼ぶ)ではそれぞれメリットとデメリットが存在します。

### スコープ

まず最大の違いはスコープです。
ローカル型は当然そのファイル内でしか存在しないので、**ファイルの中に登場人物が全て書かれる** ことになります。これはコードの把握に非常に有効で、大きな利点になります。
一方グローバル型では、`Faraday` の例ように他のファイルでインポートされたものが突然出てくることがあります[^2]。

一方、スコープがファイル内に限定されることから、ローカル型の場合は **全てのファイルで必要なものをインポートしないといけない** という問題があります。
どのファイルにも先頭数行に同じようなインポート文を書き連ねないといけないのです。
これは非常に面倒で、ウンザリする作業です。

また、ローカル型は **ライブラリをライブラリで拡張する** ことが原理的にできません。
これは、Rubyがその **オープンクラス** の仕様と相まって最強な部分で、
言語の組み込みクラスですらバリバリ拡張してしまう `activesupport` のようなものを当然のように享受している身としては、なかなか世界観を理解できない部分です。
例えば、 `react-immutable-proptypes` が `package.json` に書かれているなら、**prop-types 自体が拡張されて欲しい** と思うのです。
そんなことはできないので、実際にはあらゆるファイルに両者のライブラリを `import` しなくてはなりません。

これを実際にやろうとすると、`prop-types` に**プラグイン**のような仕組みを入れ、設定ファイルに記述させると言う方法が考えられます。
ちょうど `babel` では `presets` や `plugins` でそのようなことができるようになっていますが、もちろんそんな仕組みを作るのは非常に大変だというのは想像に難くないでしょう。
一方Rubyにもし `PropTypes` があるとしたら、

```
# react-immutable-proptypes.rb

require 'prop-types'

class PropTypes
  def map; end
  def list; end
end
```

こんな簡単に拡張できてしまいます。

[^2]: 実際は上位のファイルでインポートし、下位のファイルで使われることが多い

### 依存関係

依存関係の解決は、ローカル型に圧倒的なアドバンテージがあります。

グローバル型の場合、プロセス内の全体で1つのライブラリのメモリ空間を共有するので、**依存するライブラリのバージョン制約を解決する** 必要があります。
すなわち、*ライブラリA* と *ライブラリB* がともに *ライブラリX* を利用する依存関係がある場合、 *A* が要求する *X* のバージョンと *B* が要求する *X* のバージョンが矛盾しないようにしないといけません。
例えば *A* がバージョン `3.0` 以上、 *B* がバージョン `2.x` を使うという指定をした場合、*A* と *B* を同時に使うことができません。
この問題はよく、*ライブラリX* が `2.x` から `3.x` にアップグレードされた後、
メンテが活発でない *ライブラリB* が更新されないせいで、**ライブラリAの更新もできなくなり**、世の中から取り残される、という事例になって現れます。

一方、ローカル型の場合、ライブラリ間でメモリ空間を共有することはなく、
*A* は **Bの使ってるXのバージョンなぞ知ったことではない** という態度がとれます。
つまりそもそもバージョン制約を解決する必要が全く無いのです。
デメリットとして、上手くやらないと依存関係のネストでダウンロード時間と消費ストレージが爆発的に肥大化するという問題があります。
とはいえ、今時のストレージはライブラリに比べて無限に大きいので、

### 名前空間

ローカル型は、変数名でライブラリにアクセスできるので、**名前の変更** が容易です。
例えばPythonの世界では、超人気ライブラリである `numpy` はほぼ全て `import numpy as np` という形でインポートされ、短縮された `np` という名前が好んで使われます。

当然グローバルの名前空間を汚すことがないのはもちろん、名前を変更することで局所的な名前衝突の解決も簡単です。
複数のライブラリで関数名などが一致してしまった場合でも、インポート時に名前を変えることで衝突を回避できます。
例えばゲームの地図を表す `Map` と、データ構造を表す `Map` を同時にインポートする場合、例えばゲームの方を `GameMap` という名前に変更すればよいでしょう。

一方グローバル型では名前を変更することはできず、細心の注意を払わなければなりません。
同じ名前が存在すると、意図せず定義を上書きしてしまったり、名前の衝突でエラーになったりします。
しかし、Rubyはこのような仕様にも関わらず世界が崩壊していません。

Rubyの世界では、特にRailsとともに広まった **CoC(設定より規約)** という文化により、Gemにも暗黙的な規約が広く共有されています。
Rubygemsのリポジトリ名は早いものがちなので一意になり、リポジトリ名から機械的に変換した名前を **module名にする** という文化で名前の衝突を防いでいます。
例えば `rspec-rails` という名前のGemなら、 `RSpec::Rails` というモジュール名が使われます[^3]。

[^3]: Railsが普及する以前の古いものや一部のGemは例外がある

ただこれにも罠があり、あとからGemを導入しようとした時や **Ruby本体の組み込みクラスが増えた時**[^4] に、アプリケーションですでに使われている名前と被る場合はどうしようもありません。
特に、Railsのような名前付けに強い規約がある場合後からアプリケーション側の名前を逃がすのはとても大変です。

[^4]: 過去に `Warning` というモジュールが本体に追加されたときに、アプリケーションにすでに `warnings` テーブルがありとても苦労した

## まとめ

ローカル型のライブラリは非常に利点が多く、世の中のほとんどがローカル型になっていて世は正にローカル型一辺倒と言う感じです。

**しかし**、オープンクラスを利用して言語自体をガリガリ書き換えていくRubyのあの快感はやはり捨てがたく、
そして実際メチャクチャ便利でコーディング自体のめんどくささも極限まで減らします。
俗に、 **RailsはRubyVMの上で動くRailsと言う言語** と呼ばれることがあるほど、Rubyの拡張性は想像を絶するものがあります。
これ以上に **たのしいRuby** に貢献している要素は無いと思います。

とにかくとにかくとにかく! いちいち `import` 書くのマジでめんどくさくてもうウンザリ!!!!!