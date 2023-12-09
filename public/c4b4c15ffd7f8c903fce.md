---
title: Rustの基礎
tags:
  - Rust
private: false
updated_at: '2023-12-09T18:55:19+09:00'
id: c4b4c15ffd7f8c903fce
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

最近、Rustを勉強すると良いと先輩に教えてもらったので、勉強を始めてみました。ドキュメントが結構長いので、備忘録として投稿しようと思います。ついでに、誰かRustを初めて勉強しようと言う人がいれば、参考になればと思います。

勉強中なので、少しずつ記事を更新していきます。

# Rustとは

Rustは信頼性が高く高速なソフトウェアを構築することが容易にできるプログラミング言語である。ランタイムやガベージコレクションを使わずに、高速でメモリの安全性も確保できるプログラムを作ることができる。他の言語との統合も容易で、パフォーマンスが重要なサービスや組み込みシステムの構築に利用できる。

# 変数や定数

## let

Rustの変数は`let`で宣言できる。型は指定してもしなくても良い。

```Rust:src/main.rs
fn main() {
    let x = 5;
    println!("The value of x is {}", x);
    let y: u32 = 100;
    println!("The value of y is {}", y);
}
```

`let`で宣言した変数はそのスコープ内でのみ使用することができる。またJavascriptなどとは違い、`let`のみで宣言した変数は値を書き換えることができず、コンパイルした時に以下のようなエラーが出る。

```Rust:src/main.rs
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

```bash
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables` due to previous error
```

値を変更することができる変数を宣言する時は`mut`を一緒に使う。

```Rust:src/main.rs
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

### シャドーイング

変数は同じ名前で、型を変えても宣言することができる。前に示したように、変数は宣言したスコープ内でのみ利用できるので、以下のような実行結果になる。

```Rust:src/main.rs
fn main() {
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }
    println!("The value of x is: {}", x);
}
```

```bash
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/variables`
The value of x in the inner scope is: 12
The value of x is: 6
```

シャドーイングをすると、Rustは新しくメモリを確保する。

```Rust:src/main.rs
fn main() {
    let mut x = "test1";
    println!("{0}: {1:p}", x, &x);
    x = "test2";
    println!("{0}: {1:p}", x, &x);
    let x = "test3";
    println!("{0}: {1:p}", x, &x);
}
```

```bash
$ cargo run
   Compiling playground v0.0.1 (/playground)
    Finished dev [unoptimized + debuginfo] target(s) in 1.09s
     Running `target/debug/playground`
test1: 0x7ffd5fe1c790
test2: 0x7ffd5fe1c790
test3: 0x7ffd5fe1c850
```

## const

定数を宣言するときは、型を指定して`const`を使う。他の言語と同じように、値を変更することができない。Rustの慣習として、定数名は全て大文字で単語の区切りはアンダーバー（`_`）で宣言する。定数はグローバルなスコープを含めてすべての場所で宣言することができ、そのスコープ内でのみ利用可能である。

```Rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

定数は以下の点で変更不可能な変数と異なる。

- `mut`を使えない
- 型を指定する必要がある
- どこでも宣言できる
- いつ実行しても値は同じ
- 同じ名前で宣言できない（シャドーイングできない）

# データ型

変数は通常、型を指定しなくてもコンパイラが推測してくれるが、文字列から数値への変換など、複数の型が予想できる場面では指定する必要がある。

```Rust
let guess: u32 = "42".parse().expect("Not a number!");
```

ここで型を指定しないで実行すると以下のようにコンパイルエラーになる。

```bash
$ cargo run
   Compiling playground v0.0.1 (/playground)
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^ consider giving `guess` a type

For more information about this error, try `rustc --explain E0282`.
error: could not compile `playground` due to previous error
```

## スカラー

スカラー型は大きく分けて、整数，浮動小数点数，ブール，文字の４種類がある。

### 整数

整数には以下の表に示す型を使える。型指定をしないときの型は`i32`となる。

|Length|Signed|Unsigned|
|--|--|--|
|8-bit|`i8`|`u8`|
|16-bit|`i16`|`u16`|
|32-bit|`i32`|`u32`|
|64-bit|`i64`|`u64`|
|128-bit|`i128`|`u128`|
|arch|`isize`|`usize`|

`isize`と`usize`は利用しているCPUの性能に依存し、32bit版なら32bit、64bit版なら64bitになる。

`52u32`など数値の後に型を指定することもできる。また、100万なら`1_000_000`など、読み易いように整数を区切ることもできる。他にも以下の表のように整数を表現できる。

|Number literals|Example|
|--|--|
|10進数|`98_222`|
|16進数|`0xff`|
|8進数|`0o77`|
|2進数|`0b1111_0000`|
|バイト (`u8`のみ)|`b'A'`|

計算した結果がサイズを超えるとエラーになる。このオーバーフローを扱う以下のようなメソッドが用意されている（詳しくは[u32のドキュメント](https://doc.rust-lang.org/std/primitive.u32.html#method.checked_add)を参照）。

```Rust:src/main.rs
fn main() {
    let x: u8 = 255;
    //オーバーフローが起こるとラッピングされる
    println!("x: {:?}", x.wrapping_add(1));
    //オーバーフローが起こるとNoneが返る
    println!("x: {:?}", x.checked_add(1));
    //オーバーフローが起こるとラッピングされた値とtrueのタプルが返る
    println!("x: {:?}", x.overflowing_add(1));
    //オーバーフローが起こると境界値が返る
    println!("x: {:?}", x.saturating_add(1));
}
```

```bash
$ cargo run
   Compiling playground v0.0.1 (/playground)
    Finished dev [unoptimized + debuginfo] target(s) in 1.07s
     Running `target/debug/playground`
x: 0
x: None
x: (0, true)
x: 255
```

### 浮動小数点数

Rustには`f32`と`f64`の２種類の浮動小数点数型があり、それぞれ32ビットと64ビットのサイズになるので、`f64`の方が精度が高い。デフォルトでは`f64`になる。

```Rust:src/main.rs
fn main() {
    let x = 2.0; // f64
    let y: f32 = 3.0; // f32
}
```

### 四則演算

```Rust:src/main.rs
fn main() {
    // 加算
    let sum = 5 + 10;

    // 減算
    let difference = 95.5 - 4.3;

    // 積算
    let product = 4 * 30;

    // 除算
    let quotient = 56.7 / 32.2;
    let floored = 2 / 3; // 結果は0

    // 剰余
    let remainder = 43 % 5;
}
```

### ブール

ブール型`bool`は`true`または`false`の値を取り、サイズは1バイトになる。

```Rust:src/main.rs
fn main() {
    let t = true;
    let f: bool = false; // with explicit type annotation
}
```

### 文字

文字型`char`は4バイトでユニコード文字を一つ保持し、表すときはシングルクォーテーションを使う。

```Rust:src/main.rs
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

## 複合型

複数の値を一つの型にグループ化できる。Rustにはタプルと配列の２種類ある。

### タプル

タプルは複数のデータ型の、複数の値を一つにグループ化することができる。一度宣言されると長さを変更することができず、要素の削除や追加はできない。タプルの宣言は以下のようにする。

```Rust:src/main.rs
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

上の例ではデータ型を指定しているが、指定しなくても宣言することができる。

Rustでは以下の例のように、タプルの各値を別々の変数に割り当てることができる（非構造化）。

```Rust:src/main.rs
fn main() {
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("The value of y is: {}", y); //6.4
}
```

タプルの要素にアクセスするときは、以下のようにピリオド（`.`）を利用してインデックスを指定する。

```Rust:src/main.rs
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0; //500
    let six_point_four = x.1; //6.4
    let one = x.2; //1
}
```

### 配列

配列はタプルと異なり、全ての値が同じデータ型にならなければならない。また、配列の長さは宣言した時に決められて、宣言後に要素を追加したり、削除したりすることはできない。要素の数がわかっている時や、そもそも要素を変更しないような時に利用する。

データ型や要素数を指定しなくても宣言することができるが、指定する場合は以下のようにする。

```Rust:src/main.rs
fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
}
```

また、全ての要素の値を同一にして、一気に宣言することもできる。以下の例では全要素の値が3の長さ5の配列を宣言している。

```Rust:src/main.rs
fn main() {
    let a = [3; 5]; //[3,3,3,3,3]と同じ
}
```

配列の要素には以下のようにアクセスする。

```Rust:main.rs
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0]; //1
    let second = a[1]; //2
}
```

スタンダードライブラリには配列に似ているベクトル型があり、こちらはサイズを変更することができる。

# 関数

Rustの関数は、`fn`を使って以下の例のように宣言する。宣言の順番に関係はない。引数を使うときは、各引数のデータ型を指定しなくてはならない。Rustの慣習として、関数の名前は小文字の単語をアンダーバー（`_`）で連結したものを使う。

```Rust:main.rs
fn main() {
    println!("Hello, world!");

    test_function_no_params();
    print_labeled_measurement(5, 'h');
}

fn test_function_no_params() {
    println!("Test function.");
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}
```

```bash
$ cargo run
   Compiling playground v0.0.1 (/playground)
    Finished dev [unoptimized + debuginfo] target(s) in 2.60s
     Running `target/debug/playground`
Hello, world!
Test function.
The measurement is: 5h
```

## ステートメントとエクスプレッション

いくつかの処理を実行する指示をステートメント、なんらかの計算結果を返すものをエクスプレッションと呼ぶ。関数の中身は複数のステートメントにより構成され、たまにエクスプレッションで終わる。ステートメントはセミコロン`;`で終了し、値を返さない。したがって、以下のような関数はエラーが出る。

```Rust:src/main.rs
fn main() {
    let x = (let y = 6);
}
```

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: variable declaration using `let` is a statement

error[E0658]: `let` expressions in this position are experimental
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: see issue #53667 <https://github.com/rust-lang/rust/issues/53667> for more information
  = help: you can write `matches!(<expr>, <pattern>)` instead of `let <pattern> = <expr>`

warning: unnecessary parentheses around assigned value
 --> src/main.rs:2:13
  |
2 |     let x = (let y = 6);
  |             ^         ^
  |
  = note: `#[warn(unused_parens)]` on by default
help: remove these parentheses
  |
2 -     let x = (let y = 6);
2 +     let x = let y = 6;
  |

For more information about this error, try `rustc --explain E0658`.
warning: `functions` (bin "functions") generated 1 warning
error: could not compile `functions` due to 2 previous errors; 1 warning emitted
```

スコープもエクスプレッションになることができ、以下のように記述することができる。ここで、スコープの中身の最後にセミコロンを入れてはいけない。

```Rust:src/main.rs
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

```bash
$ cargo run
   Compiling playground v0.0.1 (/playground)
    Finished dev [unoptimized + debuginfo] target(s) in 1.44s
     Running `target/debug/playground`
The value of y is: 4
```

## 返り値

関数が値を返すようにするためには、最初の宣言でなんのデータ型の値を返すのかを`->`を使って指定しなくてはならない。`return`ステートメントを使って明示的に値を返すこともできるが、Rustでは暗黙的に一番最後のエクスプレッションを返す。

```Rust:src/main.rs
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/functions`
The value of x is: 6
```

ここで、`plus_one`関数の最後を`x + 1;`とセミコロンをつけてコンパイルすると以下のようなエラーが出る。

```bash
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error[E0308]: mismatched types
 --> src/main.rs:7:24
  |
7 | fn plus_one(x: i32) -> i32 {
  |    --------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
8 |     x + 1;
  |          - help: consider removing this semicolon

For more information about this error, try `rustc --explain E0308`.
error: could not compile `functions` due to previous error
```

# 条件分岐・繰り返し

## `if`エクスプレッション

条件分岐の利用には以下のように`if`を利用する。条件部分はブール値が返るエクスプレッションのみ指定可能。`else`は指定しなくても大丈夫。

```Rust:src/main.rs
fn main() {
    let number1 = 3;
    let number2 = 7;

    // if condition { expressions }
    if number1 > 5 && number2 > 5  {
        println!("both number1 and number2 are greater than 5");
    } else if !(number1 > 5 || number2 > 5) {
        println!("both number1 and number2 are less than 5");
    } else {
        println!("either number1 or number2 is greater than 5");
    }
}
```

条件の後はブロックを記述する。ブロック内にはエクスプレッションを記述する。そのため、以下のようにブロックで値を返すこともできる。

```Rust:src/main.rs
fn main() {
    let condition = true;

    let result = if condition { "Yes" } else { "No" };

    println!("is condition true?: {}", result);
}
```

ただし、各ブロックで返る値の型は同じでなければならない。違う型の値が返るとき以下のようなコンパイルエラーが出る。

```bash
$ cargo run
   Compiling playground v0.0.1 (/playground)
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:4:48
  |
4 |     let result = if condition { "Yes" } else { 1 };
  |                                 -----          ^ expected `&str`, found integer
  |                                 |
  |                                 expected because of this

For more information about this error, try `rustc --explain E0308`.
error: could not compile `playground` due to previous error
```

## 繰り返し

Rustの繰り返し処理には`loop`, `while`, `for`がある。

### `loop`による繰り返し

`loop`を利用すると、`break`を使って明示的に止めない限り、ブロック内に書かれた処理が永遠に実行される。また、他の言語と同様に`continue`を利用して、ループ内の残りの処理をスキップすることができる。たとえば以下のように使うことができる。

```Rust:src/main.rs
fn main() {
    let mut count = 0;
    loop {
        if count > 5 { break; }
        count += 1;

        if count % 2 == 1 { continue; }
        println!("count: {}", count);
    }
}
```

```text
count: 2
count: 4
count: 6
```

`break`や`continue`は、これらのステートメントが記述されたブロックの繰り返しに対してのみ有効である。しかし、`loop`にはオプションとしてラベルをつけることができ、`break`や`continue`の後にラベルをつけることによって、対象の繰り返し処理を指定することができる。ラベルはシングルクォーテーション（`'`）で始める。以下の例では、外側の`loop`に対して`'counting_up`というラベルが付けられていて、`count`が`2`の時に外側の`loop`を終了している。

```Rust:src/main.rs
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```

```text
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2
```

また、`loop`は`break`の後に値を指定することによって、その値を返すことができる。

```Rust:src/main.rs
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

```text
The result is 20
```

### `while`による繰り返し

`while`の後に条件を指定することによって、他の言語と同様の繰り返しをすることができる。

```Rust:src/main.rs
fn main() {
    let mut count = 0;
    // while condition { expressions }
    while count <= 5 {
        count += 1;

        if count % 2 == 1 { continue; }
        println!("count: {}", count);
    }
}
```

```text
count: 2
count: 4
count: 6
```

`while`にもラベルをつけて`break`や`continue`に指定することができる。しかし、`while`は値を返すことができない。

### `for`による繰り返し

`for`による繰り返しは、Pythonなどと同じような使い方で、配列などを指定して各要素に対して同じ処理を実行することができる。

```Rust:src/main.rs
fn main() {
    let test_array = ["one", "two", "three"];
    // for element in array { expressions }
    for str in test_array {
        println!("{}", str);
    }

    for number in (9..21).step_by(3).rev() {
        println!("number: {}", number);
    }
}
```

```text
one
two
three
number: 18
number: 15
number: 12
number: 9
```

上の2番目の`for`文のように`Range`を利用することもできる。詳しくは次のドキュメントを参照（[Range in std::ops](https://doc.rust-lang.org/std/ops/struct.Range.html)）。

# 参考文献

- <https://www.rust-lang.org/>
- <https://doc.rust-lang.org/std/primitive.u32.html>
