# 文字列スライス
```
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```
- 文字列の初期位置ポインタとそこからの長さを持つ
- 書き方としては`&s[starting_index..ending_index]`
  - 0から始めたい場合は`..ending_index`の書き方で良い
  - 最後が元の文字列の最後の場合も省略できる`starting_index..`
  - 両方省略すると文字列全体のスライスを得る
- 文字列スライスは`%str`型
## 文字列リテラルはスライスである
```
let s = "Hello, world!";
```
- ここでのsは`&str`型であり、バイナリの特定の位置を指すスライス
- つまり、元の文字列自体は不変であり、`&str`も不変な参照となる
- そのため標準入力などで入れる場合は直接`String`型で入れなければならない

## 引数としての文字列スライス
```
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```
- こういった関数にて引数を`s: &str)`にすることで`String`にも`&str`にも対応できる
```
fn main() {
    let my_string = String::from("hello world");

    // first_wordは`String`のスライスに対して機能する
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_wordは文字列リテラルのスライスに対して機能する
    let word = first_word(&my_string_literal[..]);

    // 文字列リテラルは、すでに文字列スライス*な*ので、
    // スライス記法なしでも機能するのだ！
    let word = first_word(my_string_literal);
}
```
# 他のスライス
- 文字列以外のスライスも存在する
```
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```
- この場合は`&[i32]`という型になる
- データの保持としては文字列スライスと同じ
