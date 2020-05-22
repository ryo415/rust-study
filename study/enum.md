# ENUM
```
enum IpAddrKind {
    V4,
    V6,
}
```
- 上記の様な形式で定義すると他の場所で独自のデータ型として使用できる
## Enumの値
```
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```
- 上記の様に下記のポイントとしてインスタンスを生成する
  - 識別子を元に名前空間分けされている
  - 2連コロンを使って名前空間を区別する
- こうすることでV4もV6も同じ型`IpAddrKind`となったため`IpAddrKind`型をとる関数が作れる
```
fn route(ip_type: IpAddrKind) { }

route(IpAddrKind::V4);
route(IpAddrKind::V6);
```
- 各enumの列挙子に直接データを格納することもできる
```
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```
- `IpAddr`の定義は`V4`と`V6`の両方に`String`値が紐づけられていることを示す
- enumは`V4`は4つの数値で`V6`は引き続き`String`で格納することが可能
```
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```
- しかしながらIPアドレスについては標準ライブラリに定義が存在する
- 定義のされ方としてはenumにて2種の異なる構造体の型で定義されている
```
struct Ipv4Addr {
    // 省略
}

struct Ipv6Addr {
    // 省略
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```
- 他の例としては以下の様なものがある
```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```
- 4つの列挙子
  - Quit: 紐づけられたデータなし
  - Move: 匿名構造体
  - Write: 単独の`String`オブジェクト
  - ChangeColor: 3つの`i32`値
- 上記のenumは下記の`Message`型の元であることを除き、下記の構造体を定義していることに類似する
```
struct QuitMessage; // ユニット構造体
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // タプル構造体
struct ChangeColorMessage(i32, i32, i32); // タプル構造体
```
- enumと構造体の似た点としてimplを用いてメソッドを定義することができる
```
impl Message {
    fn call(&self) {
        // method body would be defined here
        // メソッド本体はここに定義される
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```
- 上記ではm自身が`self`となる
## Option enumとNull値に勝る利点
- Rustにはnull機能がない
- Nullは10億ドルの失敗 by トニー・ホーア
- nullの問題はnullの値をnullでない値の様に使用したらエラーが出ること
- 問題自体は概念ではなく実装にある
- Rustにはnullはないが、値が存在するか不在かという概念のenumならある
- `Option<T>`で標準ライブラリに定義されている
```
enum Option<T> {
    Some(T),
    None,
}
```
- 初期化処理にさえ含まれるため明示的にスコープに導入する必要がない
- また列挙子もそうなっており`Some`と`None`を`Option::`なしで直接使える
```
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```
- `None`を使ったら`Option<T>`の型が何になるか教えなければならない
- `None`値だけでは`Some`列挙子が何型を保持するか推論できない
### nullより`Option<T>`の方が好ましい理由
- `Option<T>`と`<T>`は異なる型のため、コンパイラが`Option<T>`値を確実に有効な値かの様に使用させてくれない
```let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```
- 上記は`Option<i8>`と`<i8>`は異なる型のためエラーとなる
- `i8`の様な型の値がある場合、コンパイラが有効な値であることを確認してくれる
- `Option<i8>`がある時のみ、値を保持していない可能性を心配し、コンパイラがその様な場面があるか確かめる
- つまり`T`型の処理をする前に`Option<T>`を`T`に変換する必要がある
- そうすることで問題を捕捉する助けとなる
- `Some`列挙子から`T`型への変換は`Option<T>`のドキュメントにてメソッドを参照

# matchフロー演算子
- 値を比較し、マッチしたパターンに応じて実行してくれるフロー演算子
- 値が適合する最初のパターンのコードブロックに落ちる
```
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```
- `パターン => 動作するコード`の様に記述
- 各アームのコードは式であり、アームの式の結果が`match`式全体の戻り値になる
- 動作コードが複数行の場合、下記の様に書く
```
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```
## 値に束縛されるパターン
- マッチのアームの利点として、パターンにマッチした値の一部に束縛できる点がある
```
#[derive(Debug)] // すぐに州を点検できるように
enum UsState {
    Alabama,
    Alaska,
    // ... などなど
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```
- `Coin::Quarter`列挙子にマッチすると`state`変数はそのクォータのstateの値に束縛される
- `value_in_cents(Coin::Quarter(UsState::Alaska))`と呼び出すと`coin`は`Coin::Quarter(UsState::Alaska)`となる
- つまり`state`には`UsState::Alaska`という値が入る
## `Option<T>`とのマッチ
- `match`を用いて`Option<T>`を扱うこともできる
```
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```
- 1つ目の`plus_one`では`Some(i)`が呼ばれ`Some(5+1)`が返る
- 2つ目では`None`のためそのまま`None`が返る

### マッチは包括的
- enumの全パターンを包括していない`match`はコンパイラが通さない
```
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```
- 上記だと`None`がカバーされていないためエラーとなる
- 特に`Option<T>`の場合、`None`の処理をするのを忘れない様にしてくれる

### `_`というプレースホルダー
- `_`としてパターンを書くとどんな値にもマッチする様になる
```
ome_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```
- 上記だと`1,3,5,7`以外の場合は`_`にマッチする様になる
# `if let`で簡潔なフロー制御
- `if let`記法で一つのパターンにマッチする値を扱うことができる
- `Option<u8`にマッチするが、値が3のときだけ実行したい場合
```
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```
- 上記だと`Some(3)`以外に`_ => ()`を追加しなければならない
- `if let`を使用するともっと短く書くことができる
```
if let Some(3) = some_u8_value {
    println!("three");
}
```
- `if let`は統合記号で区切られたパターンと式をとり、式が`match`に与えられる
- しかしながら`match`で強制された包括性チェックを失う
- `if let`では`else`を入れることができる
```
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```
- 上記は`match`で下記の様に記述するのと等価である
```
let mut count = 0;
match coin {
    // {:?}州のクォーターコイン
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```
- `match`か`if let`かは簡潔性をとるか、包括性をとるかで選択する
