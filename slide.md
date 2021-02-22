#### Misokatsu.rb #1

- - -

### Ruby 3.0 で変わった private と attr_xxx

---

#### 自己紹介
- - -

* 名前：osyo
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 趣味で Ruby にパッチを投げたり気になった bugs.ruby のチケットをブログにまとめたりしている                     <!-- .element: class="fragment" -->
* Kaigi on Rails                 <!-- .element: class="fragment" -->
    * [ActiveRecord の歩み方](https://speakerdeck.com/osyo/activerecord-falsebu-mifang)
* 銀座Rails                   <!-- .element: class="fragment" -->
    * [これからの Ruby と今の Ruby について](https://speakerdeck.com/osyo/korekarafalse-ruby-tojin-false-ruby-nituite)
    * [12月25日にリリースされる Ruby 3.0 に備えよう！](https://speakerdeck.com/osyo/12yue-25ri-niririsusareru-ruby-3-dot-0-nibei-eyou)
* Burikaigi2021                 <!-- .element: class="fragment" -->
    * [Ruby 2.0 から Ruby 3.0 を駆け足で振り返る](https://speakerdeck.com/osyo/ruby-2-dot-0-kara-ruby-3-dot-0-woqu-kezu-dezhen-rifan-ru)

---

Misokatsu.rb はワシが名付けたんじゃよ

![](https://i.gyazo.com/d0d469782d85ae66d4652d322a8ea7f5.png)

---

## 今日話すこと
### Ruby 3.0 で変わった private と attr_xxx

---

#### public / private / protected のおさらい
- - -

* public / private / protected はメソッドのアクセス参照を制御する機能
    * private にするとレシーバを付けてメソッドが呼べなくしたり等

```ruby
class X
  # これは Ruby 3.0 で入ったエンドレスメソッド定義
  def hoge = "hoge"

  # 引数の名前のメソッドを private 化する
  private :hoge

  # レシーバを付けない場合は呼び出せる
  def foo = hoge
end

# private 化されたメソッドはレシーバ付きで呼べない
x = X.new
p x.foo
# error: private method `hoge' called for #<X:0x0000562a3e262460> (NoMethodError)
x.hoge
```

---

#### Ruby 3.0 の public / private / protected について
- - -

* Ruby 3.0 では private などの引数に配列を渡せるようになった
    * 元々複数の引数を渡すこともできたが配列も渡せるようになった

```ruby
class X
  def hoge = "hoge"
  def foo  = "foo"
  def bar  = "bar"

  # 配列を渡して複数のメソッドを private 化できる
  # 以下のように可変長引数で書くことは以前からできてた
  # private :hoge, :foo, :bar
  private [:hoge, :foo, :bar]
end

pp X.private_instance_methods(false)
# => [:hoge, :foo, :bar]
```

---

#### attr_accessor / attr_reader / attr_writer のおさらい
- - -

* attr_accessor / attr_reader / attr_writer はアクセッサメソッドを定義するメソッド

```ruby
class User
  # ゲッターメソッド #id メソッドが定義される
  attr_reader :id

  # セッターメソッド #name= メソッドが定義される
  attr_writer :name

  # アクセッサメソッド #age, #age= メソッドが定義される
  attr_accessor :age
end

homu = User.new
homu.name = "homu"
homu.age = 14
# これはエラーになる
homu.id = 1
```

---

#### Ruby 3.0 の attr_accessor / attr_reader / attr_writer
- - -

* Ruby 3.0 では attr_accessor などの戻り値が『定義されたメソッドの配列』を返すようになった

```ruby
class User
  # ゲッターメソッド名が返ってくる
  pp attr_reader :id, :status
  # => [:id, :status]

  # セッターメソッド名が返ってくる
  pp attr_writer :name
  # => [:name=]

  # ゲッターとセッターの両方のメソッドが返ってくる
  pp attr_accessor :age, :type
  # => [:age, :age=, :type, :type=]
end
```

---

### Ruby 3.0 で private と attr_accessor を
### 組み合わせると…？

---

## private と attr_accessor を
## ワンラインで適用できる

---

#### private attr_accessor :hoge, :foo と定義できる
- - -

* private は配列を受け取ることができる
* attr_accessor は配列を返す
* この2つを組み合わせることでワンラインで完結することができる

```ruby
class User
  # attr_reader の結果を public 化する
  public attr_reader :id

  #  attr_reader の結果を private 化する
  private attr_writer :name

  # attr_accessor すべてを private 化する
  private attr_accessor :age, :type
end

pp User.private_instance_methods(false)
# => [:age, :name=, :type, :type=, :age=]
```

---

#### おまけ：クラスメソッドを private 化する
- - -

* class スコープ内ではインスタンスメソッドを private 化する
* クラスメソッドを private 化する場合はスコープを変更する

```ruby
class X
  # クラスメソッドを定義する
  def self.hoge = "hoge"

  # これはインスタンスメソッドを private 化するのでエラーになる
  private :hoge

  # 特異クラスのスコープ内で private 化する
  class <<self
    # ここは X クラスの特異クラスのスコープになるので
    # X クラスの特異クラスのメソッドがプライベート化される
    private :hoge
  end
end

# error: private method `hoge' called for X:Class (NoMethodError)
X.hoge
```

>>>

* 以下のように attr_accessor も特異クラスのスコープで定義できる

```ruby
class X
  class <<self
    # X の特異クラスに対してメソッドを定義する
    # すなわちクラスメソッドとしてメソッドが定義される
    private attr_accessor :hoge, :foo, :bar
  end
end

pp X.private_methods(false)
# => [:hoge, :foo, :foo=, :bar, :bar=, :hoge=, :initialize, :inherited]
```

---

## まとめ

---

#### まとめ
- - -

* Ruby 3.0 から private の引数や attr_accessor の戻り値が少し変わった     <!-- .element: class="fragment" -->
    * private はメソッド名を配列で受け取ることができるようになった
    * attr_accessor は定義されたメソッド名を配列を返すようになった
* 少し変わっただけでめっちゃ便利に使えるようになった       <!-- .element: class="fragment" -->
* private + attr_accessor はかなり前から要望が来ていたのでやっと流行った機能          <!-- .element: class="fragment" -->
    * 古くは 2012年の頃からチケットが立っていた
    * [[Feature #6198] public/protected/private with attr_\*](https://bugs.ruby-lang.org/issues/6198)
* 一応非互換な変更ではあるので注意してね！！！    <!-- .element: class="fragment" -->

---

#### 参照
- - -

* [[Feature #17314] Provide a way to declare visibility of attributes defined by attr\* methods in a single expression](https://bugs.ruby-lang.org/issues/17314)



---


### ご清聴
### ありがとうございました

