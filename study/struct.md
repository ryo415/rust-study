# 構造体
```
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```
- 構造体は上記のように宣言する
- 可変な構造体を作る場合は構造体全体を可変にするしかない
- rustでは一部のみを可変にすることはできない
```
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```
- 構造体のフィールドと格納する変数名が同じ場合フィールド名は省略できる
```
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```
- 元々あるインスタンスの値を元に一部のみ更新する更新記法がある
- 下記はemailとusernameのみ書き換え、他はuser1と同じにするという書き方
```
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```
## フィールドに名前のつけないタプル構造体
- 構造体名により追加の意味は含むものの、フィールドには名前がつかない構造体をタプル構造体と呼ぶ
- フィールドの型のみ指定される
```
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```
- 各フィールドへのアクセスはタプルと同様に`.`と添字で行う
## フィールドのないユニット様構造体
- 一切フィールドのない構造体を定義することもできる
- ユニット様構造体呼ばれ、トレイトのみを実装する形となる
## 構造体の所有権
- 上記の例として`&str`でなく`String`であるのには理由がある
  - 構造体インスタンスには全データを所有する必要がある
  - このデータは構造体全体が有効な間、ずっと有効である必要がある
- 構造体に他の何かに所有されたデータへの参照を保持させることもできるが、ライフタイム機能を使う必要がある
- 下記はライフタイム指定がないためエラーとなる
```
struct User {
    username: &str,
    email: &str,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}
```
- この様なことから`&str`でなく`String`を使っていた
# 構造体のトレイト継承
- `println!`マクロには標準で`{}`は`Display`として知られる整形をする
- これまでの基本形は標準で`Display`を実装している
- 構造体では`println!`が出力を整形する方法は自明でないため、出力方法を示す必要がある
- `{:?}`という指定子を書くと`println!`に`Debug`整形を出力する
- 構造体には`Debug`も明示的に示されていないため使える様にするには構造体定義の直前に`#[derive(Debug)]`を追加する
- `#[derive(Debug)]`はDebugトレイトを継承する注釈
```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
}
```
- `derive`注釈で使えるトレイトが多く提供されている

## メソッド記法
- メソッドは関数と似ているが構造体の文脈で定義される
- 最初の引数は必ず`&self`になる
- `&`がつく理由としては所有権を奪わず借用とするためである
- データ変更をする場合は`&mut self`となり、`self`として渡すのは稀である
```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```
- メソッドを定義するにはimplブロックで始める
- 定義したメソッドはそのインスタンスとドット繋ぎで呼び出せる
### `->`演算子
- CやC++では`object`がポインタの場合、その指す先のメソッド呼び出しは`object->something()`か`(*object).something()`となる
- Rustでは`&`か`&mut`、`*`は自動で付与される
- 下記のコードは同じものとなる
```
p1.distance(&p2);
(&p1).distance(&p2);
```
## 第2引数
- `&self`以外の引数は以下の様に指定する
```
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```
## 関連関数
- implブロック内にはselfを引数に取らない関数も定義できる
- これは対象となるインスタンスが存在しないため関数であり、メソッドではない
- `String::from`が関連関数の一つ
- 新規インスタンスを返すコンストラクタによく用いられる
```
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```
- 呼び出しには`::`を用い、上例だと`let sq = Rectangle::square(3);`の様に使う
