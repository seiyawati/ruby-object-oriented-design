# 第2章 単一責任のクラスを設計する

クラスはどのようなものであるべきか？  
第一段階ではとにかく「シンプル」であることを目指す。  
今現在求められている動作を行いつつ、後から変更も簡単になっているべきである。

## 2.1 クラスに属するものを決める

どのようなクラスを作るかで、アプリケーションの全体像は大きく変わる。  
だから、メソッドを正しくグルーピングしてクラスにまとめることは、とても重要。  
しかし、プロジェクトの初期段階では、正しくグルーピングすることは出来ない。完成が近づくにつれ、そして完成後も機能の変更や追加が発生する度に、変更を余儀なくされる。

変更を安全かつ楽に行えるかは、設計によって決まる。  
*設計とは、アプリケーションの可変性を保つために技巧を凝らすことであり、完璧を目指すための行為ではない。*

変更を簡単に行えるのがよい設計だが、それを考える前にまず、「変更が簡単である」とはどういうことなのか定義しておきたい。

- 変更が簡単であるとは？
  - 変更は副作用をもたらさない
  - 要件の変更が小さければ、コードの変更も相応して小さい
  - 既存のコードは簡単に再利用できる
  - 最も簡単な変更方法はコードの追加である。ただし追加するコードはそれ自体変更が容易なものとする

この定義を満たすためには、コードは以下のような性質を持っているべき。  

- *見通しがよい(Transparent)*： 変更するコードにおいても、そのコードに依存する別の場所のコードにおいても、変更がもたらす影響が明白である
- *合理的（Reasonable）*： どんな変更であっても、かかるコストは変更がもたらす利益にふさわしい
- *利用性が高い（Usable）*： 新しい環境、予期していなかった環境でも再利用できる
- *模範的（Exemplary）*： コードに変更を加える人が、上記の品質を自然と保つようなコードになっている

頭文字をとってTRUEと呼ぶが、TRUEなコードは、現在のニーズを満たしつつ、将来のニーズを満たすための変更を加えることも出来る。

そして、それぞれのクラスが単一の責任を持つようにすることが、TRUEなコードを書くための最初の一歩である。

## 2.2 単一の責任を持つクラスをつくる

変更が簡単なアプリケーションは、再利用が簡単なクラスから構成される。

2つ以上の責任を持つクラスは、再利用が難しい。  
複数の責任が絡み合ってしまっており、ある用途に対する変更が、意図せず別の用途に影響を与える可能性がある。  
そういうクラスに変更を加えれば、予期せずアプリケーションが壊れるかもしれない。

しかし、クラスをすぐに設計し直すべきかと言えば、必ずしもそうではない。  
未来は不確かである以上、時間を掛けて設計しても、それが間違っていたという可能性は常にある。  
全ての選択には対価を伴うのであり、常にトレードオフを意識しながら決断していくしかない。

将来に対する不確実性が高い状態では、時間を掛けて設計するよりも、そのままにしておき、必要性が高まってから設計したほうがいいかもしれない。  
しかし、再利用しづらく、模範的でないコードをそのままにしておくデメリットも、確かに存在する。
どちらかよいと、一方的に決めることは出来ない。「いますぐ改善」と「あとで改善」の間の緊張状態は常に存在し、そのなかで折り合いをつけていくしかない。

## 2.3 変更を歓迎するコードを書く

未来においてどんな変更が行われるか分からなくても、変更を受け入れやすいコードを構成することは出来る。  
そのためのテクニックを紹介していく。これらのテクニックは、変更を歓迎するコードを書くために役立てられる。

### データではなく振る舞いに依存する*

どこからでも参照されるデータではなく、一箇所で定義される振る舞いに依存する。  
そうすることで、変更に強いコードになる。  
変更に伴う影響が次々に波及していくのを、予防することが出来る。

具体的には、インスタンス変数や複雑なデータ構造を、メソッドで隠蔽する。  

まずはインスタンス変数。  
常にアクセスメソッドで包み、直接参照しないようにする。変数を持っているクラスからも隠蔽し、常にメソッドから値を取得するようにする。

以下の例では、@cogを直接参照している。
```ruby
class Gear
  def initialize(chainring, cog)
    @chainring = chainring
    @cog = cog
  end

  def ratio
    @chainring / @cog.to_f # <- Bad
  end
end
```

このように書いてしまうと、@cogの定義に変更があったときの対応が困難になる。  
上記の例では@cogを参照しているのは一箇所だけだが、複数の箇所から参照されていると、その全ての箇所を変更するはめになる。  

以下は、アクセサメソッドを使った例。cogの定義が一箇所で済んでいるので、変更があった場合もここだけ書き換えればよい。
```ruby
class Gear
  def initialize(chainring, cog)
    @chainring = chainring
    @cog = cog
  end

  def chainring
    @chainring
  end

  def cog
    @cog
  end

  def ratio
    chainring / cog.to_f # <- Good
  end
end
```

今回は例示のために明示的にchainringとcogを定義したが、attr_reader :chainring, :cogと書くだけでアクセサメソッドを定義することが出来る。
```ruby
class Gear
  attr_reader :chainring, :cog

  def initialize(chainring, cog)
    @chainring = chainring
    @cog = cog
  end

  def ratio
    chainring / cog.to_f # <- Good
  end
end
```

次に、複雑なデータ構造の隠蔽。  
以下のObscuringReferencesクラスは、初期化のときに受け取ったデータを@dataに保管し、それを使ってdiametersメソッドで直径を計算している。
```ruby
class ObscuringReferences
  attr_reader :data
  def initialize(data)
    @data = data
  end

  def diameters
    data.collect {|cell|
      # [0]はリム [1]はタイヤ
      # 車輪の直径 = リム + (タイヤの厚み * 2)
      cell[0] + (cell[1] * 2)
    }
  end
end

value = [[622, 20], [622, 23]]
p ObscuringReferences.new(value).diameters # [662, 668]
```

@dataは複雑な構造で、リムとタイヤのサイズが入った2次元配列になっている。  
この複雑なデータ構造を隠蔽できていないことが、ObscuringReferencesの問題点。

diametersがデータ構造について知りすぎているし、それに依存してしまっている。  
構造が変わった場合はコードの変更も発生するが、構造に依存しているコードが多ければ多いほど、変更の影響も大きくなる。  
作業が大変になるし、バグが混入するリスクも高まる。

複雑な構造への直接の参照は混乱を招く。  
リムが[0]に、タイヤが[1]に入っているという知識はただ一箇所で把握されるべきであり、それは明らかにdiametersの責務ではない。  

複雑さは隔離し、隠蔽するべき。  
そのためのテクニックとして、 Ruby ではStructを使える。
```ruby
class RevealingReferences
  attr_reader :wheels
  def initialize(data)
    @wheels = wheelify(data)
  end

  def diameters
    wheels.collect {|wheel|
      # 車輪の直径 = リム + (タイヤの厚み * 2)
      wheel.rim + (wheel.tire * 2)
    }
  end

  Wheel = Struct.new(:rim, :tire)
  def wheelify(data)
    data.collect {|cell|
      # [0]はリム [1]はタイヤ
      Wheel.new(cell[0], cell[1])
    }
  end
end
```
新しいdiametersはデータ構造について何も知らず、車輪の直径の計算方法という、本来知っているべきことだけを知っている。  
データ構造についての知識は、wheelifyの一箇所でのみ把握されている。  
このように書けば、データ構造が変わっても変更はこの一箇所のみで済む。  
複雑さは隔離し隠蔽することで、変更に強いコードになる。


### あらゆる箇所を単一責任にする

変更や再利用を簡単にするために、クラスだけでなくメソッドも単一責任にする。  

先程のRevealingReferencesは、複雑なデータ構造を隠蔽することには成功したが、diametersが単一責任になっていないという問題を抱えている。  
直径の計算と繰り返し処理の2つの責任を持っている。  
これを、diameterという直径の計算のみを行うメソッドも作る形で、リファクタリングする。
```ruby
class RevealingReferences
  attr_reader :wheels
  def initialize(data)
    @wheels = wheelify(data)
  end

  def diameters
    wheels.collect {|wheel|
      diameter(wheel)
    }
  end

  def diameter(wheel)
      # 車輪の直径 = リム + (タイヤの厚み * 2)
      wheel.rim + (wheel.tire * 2)
  end

  Wheel = Struct.new(:rim, :tire)
  def wheelify(data)
    data.collect {|cell|
      # [0]はリム [1]はタイヤ
      Wheel.new(cell[0], cell[1])
    }
  end
end
```

この節で紹介してきたリファクタリングを積み重ねていくと、次のような恩恵を受けることが出来る。  
- 隠蔽されていた性質を明らかにする
  - メソッドの単一責任を徹底していくことで、クラス全体の役割が明確になる。そのクラスがどんな性質を持っているのか見えてくる。
- コメントをする必要がなくなる
  - 小さい単位でメソッドを作り、そのメソッドに適切な名前をつけていけば、その名前がコメントの役割を果たす。
- 再利用を促進する
  - 小さなメソッドは再利用しやすい。そして小さなメソッドで構成されたアプリケーションでは、後から変更を加える他のプログラマも、既存のパターンに従ってメソッドを小さく作るようになる。
- 他のクラスへの移動が簡単
  - 小さなメソッドは簡単に動かせるため、変更が楽。そのため、設計の見直しや改善に対する敷居が下がる。

設計においては「保留」も選択肢の一つ  
状況が不透明なのに安易に決定してはいけない。  
せっかく変更可能なコードを書いているのだから、その恩恵を活かして、決定はあとに残しておく。

Gearクラスを例にすると、wheelに関する振る舞いをWheelクラスとして切り出すべきかが問題となる。  
その判断をするための情報があるならよいが、現時点では判断のための情報が足りない、というケースもあり得る。  
そういうときは、wheelに関する振る舞いを、Gearクラスに入れたままにしつつ隔離するという手法を取れる。  
Ruby の場合、Strcutにブロックを渡してメソッドを定義する。  
```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(chainring, cog, rim, tire)
    @chainring = chainring
    @cog = cog
    @wheel = Wheel.new(rim, tire)
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end

  Wheel = Struct.new(:rim, :tire) do
    def diameter
      rim + (tire * 2)
    end
  end
end
```
Gearを綺麗にしつつ、Wheelを別のクラスに切り出すべきかの決定を遅らせている。

クラスが持っている責任は単一であるよう、徹底する。  
本質でない責任は別のクラスに分離し、それがまだ出来なければ隔離する。

## 2.4 ついに、実際の Wheel の完成

アプリケーションのユーザーから、「車輪の円周も計算できるようにしたい」という要望が来た。  
この要望によって、WheelをGearから独立させて使いたいというニーズが生まれた。円周の計算（直径×円周率）は、ギアとは独立して存在しているから。  
この要望やニーズこそが、前節で触れた「判断をするための情報」。これにより、WheelをGearから独立させるべきという判断が可能になった。

前節で既にWheelの振る舞いを隔離しておいたおかげで、Gearからの独立は簡単に出来る。  
Wheelを切り出してクラスに変え、円周を計算するためのcircumferenceメソッドを追加すればよい。
```ruby
class Gear
  attr_reader :chainring, :cog, :wheel
  def initialize(chainring, cog, wheel = nil)
    @chainring = chainring
    @cog = cog
    @wheel = wheel
  end

  def ratio
    chainring / cog.to_f
  end

  def gear_inches
    ratio * wheel.diameter
  end
end

class Wheel
  attr_reader :rim, :tire
  def initialize(rim, tire)
    @rim = rim
    @tire = tire
  end

  def diameter
    rim + (tire * 2)
  end

  def circumference
    diameter * Math::PI
  end
end

wheel = Wheel.new(26, 1.5)
p wheel.circumference # 91.106186954104

p Gear.new(52, 11, wheel).gear_inches # 264.72727272727275

p Gear.new(52, 11).ratio # 4.7272727272727275
```

## 2.5 まとめ

一つのことに専念するクラスは、その一つのことを、アプリケーションの他の部位から隔離する。  
この隔離によって、悪影響を及ぼすことのない変更と重複のない再利用が可能になり、変更可能でメンテナンス性の高いソフトウェアになる。
