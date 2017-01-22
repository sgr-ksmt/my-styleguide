# Swift コーディング規約

## はじめに
- この規約は自分自身のSwiftでのコーディングの基準を定めるものです。
- あくまでも個人的にはこうでありたいという規約なので、  
関わるProject、業務毎に規約があればそれに従うものとして、これに則って頑なに抗っていくという表明ではありません。
- Swift3.0.1の言語仕様に準拠するものとします。
- 規約は都度自身がよりよいコーディングができるように改訂していきます。

## 目的
- 一貫したコーディングが行えるようにする
- 意図を明確にする
- 自身のコーディングを振り返り、適宜修正できるようにする

## :star: と :heart:
- :star: : 基本的にはこれに従います。
- :heart: : 必ず従わないといけないわけではないけど、なるべくこのスタイルでいきたいなという場合に付けます。

## スタイル
- :star: インデントは4、ソフトタブを用いる
- :star: 関数と関数の間は1行空ける
- :star: 定数、プロパティ宣言はクラス宣言の上部に記述する
- :star: `{}`(brace)は開始は最後の文字から1文字空けて記述し、終わりは改行する
- :star: ファイルの最後(EOF)にに1行改行を入れる
- :star: ドキュメントコメントを書く場合は、 `///` を使う
- :star: 適切な区切りを表現する時は、 `// MARK: - ` を使う

```swift
class SomeClass {
    private let name: String
    
    init(name: String) {
        self.name = name
    }
    
    /// do something
    func doSomething() {
        print(name)
    }
    
    // MARK: - action
    
    func doSomeAction() {
        //....
    }
}

```

## 定数と変数、プロパティ
- :star: 値の変更をしない場合は必ず `let` を用いる
- :heart: `lazy var` を用いる場合、生成処理が複数行に渡る場合は生成処理を行う関数を定義し、closureでの生成処理を避ける
    - 変数やプロパティの定義箇所が見づらくなるのと、コンパイル速度に微弱ながら影響がでるため
    - 1行のみ(単純な生成)の場合はこの限りではない

```swift
// not good
private lazy var someView: UIView = {
    let view: UIView = ...
    return view
}()

// good
private lazy var someView: UIView = self.makeSomeView()

private func makeSomeView() -> UIView {
    let view: UIView = ...
    return view    
}
// ok
private lazy var someView: UIView = UIView(frame: .zero)
```

- :star: 循環参照を防ぐために、 `weak` 属性が必要な場合は必ず付ける
    - delegateとして用いる変数、 `@IBOutlet` に対しては付与する

```swift
// good
weak var delegate: SomeDelegate
@IBOutlet titleLabel: UILabel!
```

## 型宣言と型の省略
- :star: 可能な場合は型宣言を省略する。

```swift
// not good
let viewController: SomeViewController = SomeViewController()

// good
let viewController = SomeViewController()
```

- :star: Enumの型やNestedしている型が定まり自明な場合も省略する
- :heart: その型自身を返す `static var/let` の場合も可読性が落ちない程度に省略する

```swift
// not good
let button = UIButton(frame: CGRect.zero)
let string = String(data: data, encoding: String.Encoding.utf8)

// good
let button = UIButton(frame: .zero)
let string = String(data: data, encoding: .utf8)
// =================

// not good
button.addTarget(self, action: #selector(doSomething), for: UIControlEvents.touchUpInside)

// good
button.addTarget(self, action: #selector(doSomething), for: .touchUpInside)
```

### 例外
ただし、以下のケースでは型省略をしなくても良い

- 型推論をさせてしまうことでコンパイル速度に影響がでる場合は型を明示する
- ジェネリクスを利用することでどの型が来るのか分かりづらい場合には型を明示する


## guard節
- :star: guard節を普通のBool値による判定に用いない
    - 条件判定が分かりづらく、Optional Bidingまで絡むと分かりづらくなるため
    - 基本的にはguard節は `guard let` で変数がunwrapできなかった場合にreturnさせる等の場合に用いる

```swift
// not good
let someFlag = ...
guard someflag else {
    return
}

// good
if !someFlag {
    return
}

// good
guard let view = views.first else {
    return
}
```

## if文
- :star: if文でOptioanlな変数がnilでないかを判定する場合は、 `if let _ = var`を用いる
    - ただし、nilであることを期待する判定の場合は、その限りではない

```swift
// not good
if var != nil {
}

// good
if let _ = var {
}

if var == nil {
}
```

- :star: 三項演算子は単純な条件であれば使って良い。三項演算子が2段階以上になるものや、複雑な計算が絡む場合は適宜分ける

```swift
// bad
let text = someCondition1() ? ( someCondition2() ? "!!!" : "???" ): "---"
```

## クロージャー
- :star: クロージャーの処理が1行の場合は省略できるものは省略する
    - 引数名、引数の型の省略
    - 返す値の型
    - `return`

```swift
// good
_ = [1, 2, 3].filter { $0 % 2 == 0 }
```

### 例外
但し、引数名の省略に関しては、コンパイル時間等に影響を与える、 `$` 記法によるアクセスで可読性が著しく損なう場合には  
引数名や引数の型を明示する

```swift
struct Card {
    let number: Int
    let name: String
}

_ = [Card(number: 10, name: "some card1"), Card(number: 13, name: "some card2")].filter { card: Card in
    card.number > 5
    }.sorted { (c1: Card, c2: Card) -> Bool in
        return c1.number < c2.number
}
```

- :star: 逆に、クロージャーの処理が複数行に渡って記述される場合や、クロージャーがネストする場合は引数名を明示する。
    - こちらもコンパイル時間等に著しく影響が出る場合は、引数の型、返す値の型を明示しても良い。

```swift
// good
_ = ["1", "2", "3"].flatMap { numString in
    guard let number = Int(numString) else {
        return
    }
    return number * 2
}
```

- :star: 関数に渡すクロージャーが1つである場合はTrailing closure記法で記述する
    - 関数に渡す引数がクロージャー1つのみの場合は関数名の後に`()`は付けない
    - ただし、よくある `success:failure:` でそれぞれにクロージャーを渡す場合はTrailing closure記法を避ける

```swift
// not good
_ = [1, 2, 3].map({ $0 *5 })
_ = [1, 2, 3].map() { $0 *5 }

API.send(request, success: { response in
    ...
}) { error in
    ...
}

// good
_ = [1, 2, 3].map { $0 *5 }

API.send(request, success: { response in
    ...
}, failure: { error in
    ...
})
```

## `self` と循環参照
- :star: 基本的には型、クラス内では`self.`を付けてのアクセス、関数呼び出しは以下の例外を除いて極力書かないようにする
    - 1: イニシャライザ内で、引数と同名のプロパティに値を代入するとき
    - 2: ローカル変数と同名のプロパティにアクセスしたり代入しないといけなくて混同してしまう場合(そうならないように変数名を定義すべき)
    - 3: `@escaping` なクロージャー内部で自身のプロパティにアクセスしたり関数を呼び出す場合
    - 4: 万一、globalな領域(toplevel)にある同名の関数や変数名と衝突してしまう場合(これも極力避けるべき)

- :star: `@escaping` なクロージャー内部で`self`或いはキャプチャーされる変数を参照する場合は、基本的には `[weak var]` で`weak`参照する。
    - `[unowned var]` は基本的には避ける。
        - ただし、そのタイミングで参照する変数が確実に存在する保証がもてる場合のみ検討する
    - `weak` で弱参照した場合、1行且つ簡潔な処理の場合は `var?.` でOptional Bidingを使っても良い
        - ただし、複雑な処理になる、複数行になる場合は、 `guard let` を活用して、`strongSelf` としてunwrapする。
        - `self` でunwrapは行わない。

```swift
// bad
API.send(request, success: { response in
    self.titleLabel.text = response.data.title
}, failure: nil)

// not good
API.send(request, success: { [unowned self] response in
    self.titleLabel.text = response.data.title
}, failure: nil)

// not good
API.send(request, success: { [weak self] response in
    self?.titleLabel.text = response.data.title
    self?.detailLabel.text = response.data.detail    
    self?.someView.isHidden = self?.shouldHiddenSomeView(with: response) ?? false
}, failure: nil)


// good
API.send(request, success: { [weak self] response in
    self?.titleLabel.text = response.data.title
}, failure: nil)

// good
API.send(request, success: { [weak self] response in
    guard let strongSelf = self else {
        return
    }
    strongSelf.titleLabel.text = response.data.title
    strongSelf.detailLabel.text = response.data.detail    
    strongSelf.someView.isHidden = strongSelf.shouldHiddenSomeView(with: response)
}, failure: nil)
```

## アクセスコントロール
- :star: 基本的にはスコープは狭くする。外に公開する必要がなければ `private` や `fileprivate` を付ける
- :star: 外に公開するが、代入を不可にする場合は、 `let` や `private(set)` でreadonlyにする

```swift
// good
class SomeModel {
    private let privateProperty: Int
    private(set) var someArray: [Int]
}
```

- :star: 基本的にそのモジュール内で閉じる場合には `public` を付けない。また、 `internal` は省略する

```swift
// not good
public let someNumber: Int = ...
internal var text: String = ...
```

- :star: モジュールとして切り出し、クラスやインターフェースを公開する場合には、適切に `public`, `open` を設定する

## Special Thanks
- [Swift - API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
- [raywenderlich/swift-style-guide](https://github.com/raywenderlich/swift-style-guide)
