---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# Ezno について

---

# 今日のひとこと

- 今日はかまぼこの日です
- 蒲鉾のポン酢和え
  - https://wadahachi.com/recipe/%E8%92%B2%E9%89%BE%E3%81%AE%E3%83%9D%E3%83%B3%E9%85%A2%E5%92%8C%E3%81%88/

---

# Agenda

- Ezno とは
- 特徴の紹介
- 今後の動き

---

# Ezno とは？

- JavaScript のコンパイラー
  - 型チェック、パフォーマンス向上などが特徴
- Rust 製
- "easy?no" が由来
- イギリスの Ben さんが開発している
- まだ開発の初期段階
- tsc の延長線上にあり、さらに発展させたもの

---

# 型の合成とチェック

Ezno の中核の機能

- 型の合成
  - 構文を分析し、その用語の特性を定式化すること
- 型チェック
  - 用語の情報と、それがどのように使われているかを比較し、実行時のエラーを防ぐこと
- TypeScript の型アノテーションと完全に互換性がある
  - 型アノテーションがなくても動作する

---

# Dependent Typing (依存型タイピング)

Ezno の重要な考え方の一つに、ソースコードに関する知識を「最大化」しようとすることがある

- プロパティの欠落により発生する可能性のあるランタイムエラー
- 決して実行されないコード
- 作業を軽減するために省略できる式
- ミューテーション

---

# Dependent Typing (依存型タイピング)

知識を「最大化」するために...

- まずは、 JavaScript の恩恵を受けるために、定数（number, boolean, string, etc...）への参照を型システムに含める必要がある
- TypeScript は定数等式(constant equality)、 ezno はさらに定数演算子評価(constant operator evaluation)などの機能を追加

---

# Dependent Typing (依存型タイピング)

どういうこと？🤔

```ts
// リテラル型は、特定の値だけを代入できる
const x: 5 = 4 + 2 // => Type 'number' is not assignable to type '5'.
const x: 5 = 4 + 1 // => Error here too.
const x: 5 = 5 // => OK
```

---

# Dependent Typing (依存型タイピング)

Ezno では...

```ts
// Ezno では演算子評価もしてくれる...
const x: 5 = 4 + 2 // => Type '6' is not assignable to type '5'
const x: 5 = 4 + 1 // => OK
const x: 5 = 3 + 2 // => OK
```

---

# Dependent Typing (依存型タイピング)

無効なプロパティの特定とデッドコードの検出が可能になる

```ts
const obj = {
  key3() { return 'test' }
}

obj["key" + (2 + 1)]().nonProp
// ["key" + (2 + 1)] というアクセスはできる
// => No property with "nonProp" とエラーが出る

// TypeScript だと
obj["key3"]()
```

```ts
const five = 5
if (five + 5 !== 10) { // Warning: Expression is always false
  // do stuff
}
```

---

# Objects

- Ezno は、オブジェクトや関数も定数として扱う
- したがって、前述の定数演算子評価も適用される

```ts
// これは TypeScript も同じ
if ({} === {}) { // Warning: Expression is always false
    // do stuff
}
```

---

# Objects

その他の例

```ts
// TypeScript だと Warning は出ない
const a = {}
const b = a
if (a === b) { // Warning: Expression is always true
    // do stuff
}
```

---

# Objects

その他の例

```ts
// TypeScript だと Warning は出ない
function func() {}
function func2() { return func }
if (func === func2()) { // // Warning: Expression is always true
    // do stuff
}
```

---

# 全ての関数パラメーターをジェネリックにする

- Ezno は全てのパラメーターを大半の言語が呼ぶ「ジェネリック」として扱う
- そうすることで、データの流れと、それに対するアクションをトレースすることができる
- 「ジェネリック」とは？
  - 具体的なデータ型に直接依存しない、抽象的かつ汎用的なコード記述を可能にするコンピュータプログラミング手法である
  - https://ja.wikipedia.org/wiki/ジェネリックプログラミング

---

# 全ての関数パラメーターをジェネリックにする

とりあえず TypeScript で考えてみる🤔

```ts
function chooseRandomString(v1: string, v2: string): string {
    return Math.random() <= 0.5 ? v1 : v2
}

function chooseRandomNumber(n1: number, n2: number): number {
    return Math.random() <= 0.5 ? n1 : n2
}

// やってる処理は同じなのに型だけが違うから共通化してしまいたい

function chooseRandomly<T>(v1: T, v2: T): T {
    return Math.random() <= 0.5 ? v1 : v2
}

chooseRandomly<string>('a', 'b') // => OK
chooseRandomly<string>(1, 2) // => Error
```

---

# 全ての関数パラメーターをジェネリックにする

Ezno の場合

```ts
// TypeScript だと、function addOne(x: number): number
// 返り値の型は number となる
function addOne(x: number) {
    return x + 1
}

// Ezno だと、合成された返り値の型は Add<T, 1> となる
// 引数の型: T は、 number で制限をされたジェネリクスとなる
// 全体としては addOne: <T extends number>(x: T): Add<T, 1>
```

---

# 未定義パラメーター(Untyped parameters)

- 型アノテーションを持たないパラメーターは、関数本体での使用に基づいて制約を推論する
- `id` 関数が合成されるとき、パラメーターの `a` がジェネリックであると推論される
- 関数の型は `<T extends any>(a: T) => T` となる

```ts
function id(a) {
  return a
}

// 全部通る
assertType<number>(id(2));
assertType<string>(id("Hello World"));
assertType<"x">(id("x"));
```

---

# 推定される一般的な制限事項

もう少し複雑な関数の場合 🤔

```ts
// Ezno の場合は、 obj と func は共に any のジェネリック
function runMap(obj, func) {
  return obj.map(func)
}
``` 

- `obj` は、 `map` オブジェクトを持つ必要があり、 `func` を渡せなければならない
- この際に Ezno は `runMap` 関数を下記のように型推論する
  - `<T extends { map: (a: U) => any }, U>(obj: T, func: U) => T["map"](U)`
  - `runMap` 関数の第一引数に明らかに `.map` が存在しないものを渡した時点でエラーが出るイメージ

---

# 推定される一般的な制限事項

- 関数の内部で想定される処理に即した推論を行ってくれるのが Ezno の利点
- ちなみに、同様に関数に対してジェネリクスを推論してくれる JS のタイプチェッカーに Hegel があるが、こちらは詳しく推論が機能していないらしい
- https://hegel.js.org/

---

# Effects / Events

- JS の問題の一つに、関数は「不純」になるというものがある
- 「不純」とは返される型を通じて追跡されない副作用を適用できることを意味する

```ts
// pure
function add(a, b) {
	return a + b
}

// impure
let newValue = 0
function add(a, b) {
	newValue = 100
	return a + b + newValue
}
```

---

# Effects / Events

- Ezno は、ある関数が果たすかもしれない副作用を追跡してくれる

```ts
const data = { x: 0 };

function getFive(obj) {
  obj.x += 1;
  return 5;
}

assertType<0>(data.x);
assertType<5>(getFive(data));
assertType<1>(data.x);
```

---

# Constant functions

- Ezno では、関数を単なる「形」ではなく、関数への「ポインタ」で一意に扱う
  - 変数や関数などが置かれたメインメモリ上の番地（メモリアドレス）を格納するための当別な変数のこと
  - https://ja.wikipedia.org/wiki/ポインタ_(プログラミング)
  - 「一意」であるということが重要そう🤔

---

# Constant functions

- 今まで定数での演算子計算がコンパイル時にされることの例を見てきた
- この考え方は多くの内部関数にも引き継がれる

```ts
// これが Ezno だと動作する
let x: 2 = Math.floor(Math.sqrt(5));

// TypeScript の場合
// => Type 'number' is not assignable to type '2'.
```

---

# その他恩恵を受けそうなアイデア

### JavaScript ランタイムに依存しない SSR

- フロントエンドの SSR 実装の問題点の一つは、 SSR の実装自体が JS 上で動作するため、 Node や Deno のようなサーバーランタイムにロックされてしまうこと
- ほとんどのケースで問題ないが、 Rust を使いたい！などが難しくなる
- `rusty_v8` のような、 他言語に JS ランタイムを埋め込んでいるものもあるが、型安全性の橋渡しができないなど、性能向上に繋げるのは難しい
- Ben さん的には、 Ezno を利用して他の言語と統合した型フォーマットの生成は可能であるものの、複雑なデータ変異にはまだ対応できるか微妙そうとのこと

---

# その他恩恵を受けそうなアイデア

### Linting

- 型があれば、書かれるコードも制限できる
- 詳細な型情報によって、物事の値に基づいて、ある動作をチェックしたり否定したりできる
  - `img` タグは `alt` 属性が必須
- 工夫次第でより良い開発体験が得られそう

---

# TypeScript に対するお気持ち

- Ezno には型アノテーション自体は厳密には必要ではない（制約として扱われるため）
- TypeScript は `return` アノテーションなどを真理の源として扱うので少し考え方が異なる

```ts
// Passes under tsc
function y(a: any): string {
	return a
}

y(2).slice(2)
```

- TypeScript の any の実装は少しファンキーで、 a を文字列にキャストすることができる

---

# TypeScript に対するお気持ち

- TypeScript の実装そのものをdisってるわけではなく、 TSC を取り込みつつ、イケていない部分を改善できるよう開発を続けている
- 今は完全な静的型解析ツールとして、今後は動的な構造に適用することを目標にしている（具体的なところは書いていなかった）

---

# 今後の動き

で、いつ使えるの？

- とりあえず動くものを早く出すのではなく、ゆっくり開発していきたい
- デモはまだ無いが、今年中には公開したい

---

# まとめ

- TypeScript の知見はそのまま流用できる
- 演算子評価の機能が凄そう
- 他には JSX 対応も...
  - モチベーションの一つとして React のとかく計算コストが高くなりがちな部分と上手く付き合って、静的解析能力を提供させるというものがある