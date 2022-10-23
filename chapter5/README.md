# 第5章 ダックタイピングでコストを削減する

ダックタイピングとは、いかなる特定のクラスとも結びつかないパブリックインターフェースのことである。

どういうことか？

```ruby
def chech_health_condition(duck)
  duck.quack
end
```

上記のメソッドは引数がquackメソッドを持つことを期待している。

言い換えると、  
この引数duckはアヒルのように振る舞うオブジェクトであれば、クラスは何でも良いとうこと。

よってquackメソッドはどのクラスのオブジェクトが実装していてもいいので、いかなる特定のクラスとも結びつかないパブリックインターフェースを定義していることになる。

## ダックタイピングの効果

メソッドを定義するときに、「引数は何クラスにするか？」ではなく、「このメソッドの引数はどんなメソッドを持っているべきなのか？」という振る舞いに着目した考えになる。

オブジェクト同士のメッセージのやりとりで連携するというオブジェクト指向設計においては非常に重要な考えかた。

他のオブジェクトが自身の期待する振る舞いをしてくれると`信頼することができれば`、設計の柔軟性は非常に高くなる。

ただし、この柔軟性はうまく使わなければ恐ろしいほど複雑な設計になってしまうことに注意しなくてはなりません。

そうならないためにも、ダックタイプのインタフェースは慎重に設計しなければなりません。

## 5.1 ダックタイピングを理解する

ダックタイピングされていないコード
```ruby
class Trip
  attr_reader :bicycle, :customers, :vehicle

  def prepare(preparers)
    prepares.each do |preparer|
      case preparer
      when Mechanic
        preparer.preparer_bicycles(bicycles)
      when TripCoordinator
        preparer.buy_food(customers)
      when Driver
        preparer.gas_up(vehicle)
        preparer.fill_water_tank(vehicle)
      end
    end
  end
end

class Mechanic
  def prepare_bicycles(bicycles)
  end
end

class TripCoordinator
  def buy_food(customers)
  end
end

class Driver
  def gas_up(vehicle)
  end

  def fill_water_tank(vehicle)
  end
end
```

上記のコードからダックを見つけてダックタイピングされたコードにリファクタリングする。

↓

```ruby
class Trip
  attr_reader :bicycle, :customers, :vehicle

  def prepare(preparers)
    prepares.each do |preparer|
      preparer.prepare_trip(self)
    end
  end
end

# 全ての準備者（Preparer）はprepare_tripに応答するダック
class Mechanic
  def prepare_trip(trip)
    trip.bicycles.each do |bicycle|
      prepare_bicycle(bicycle)
    end
  end
end

class TripCoordinator
  def prepare_trip(trip)
    buy_food(trip.customers)
  end
end

class Driver
  def prepare_trip(trip)
    vehicle = trip.vehicle
    gas_up(vehicle)
    fill_water_tank(vehicle)
  end
end
```

### ポリモーフィズム

オブジェクト指向プログラミングでのポリモーフィズムは、多岐にわたるオブジェクトが、同じように応答できる能力を指す。

メッセージの送り手は、受け手のクラスを気にすることなく、受け手はそれぞれが独自化した振る舞いを提供する。

## 5.2 ダックを信頼するコードを書く

### 隠れたダックを認識する

- 隠れたダックの存在示唆パターン
  - クラスで分岐するcase文
  - kind_of?とis_a?
  - respond_to?

### 賢くダックを選ぶ

以下の例では一見ダックを示唆するパターンに見えるが、firstの依存先であるIntegerやHashへの依存はRubyのコアクラスへの依存であり、安全な依存である。

ダックタイプを新たに作るかどうかはコストを下げることが物差しとなる。

Rubyの基本クラスに変更を加えるのは、モンキーパッチといい、とてもリスクの高い実装となる。

```ruby
def first(*args)
  if args.any?
    if args.first.kind_of?(Integer) || (loades? && !args.first.kind_of?(Hash))
      to_a.first(*args)
    else
      apply_finder_options(args.first).first
    end
  else
    find_first
  end
end
```

## 5.3 ダックタイピングへの恐れを克服する

静的型付けだからといって安全なわけではない。

コンパイル時のエラーで防げるのはわずかで、実行時のエラーが防げることはない。

動的型付けで効率を重視した方が良いのではないか。

ダックタイピングは動的型付けの上に成り立つ。

ダックタイピングをするときに、静的型付けに慣れている人が考えること
https://developers.wonderpla.net/entry/2022/03/24/110000

RubyでJavaのインターフェースっぽいことをする
https://gitpress.io/@rhiroe/ruby-interface

プログラムにおいてのシグネチャの意味
https://zenn.dev/t_kitamura/articles/90bc98a3787044
