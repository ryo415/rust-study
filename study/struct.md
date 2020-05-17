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
