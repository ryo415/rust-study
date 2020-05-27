# モジュール
- `mod`キーワードで新規モジュールを宣言
- 標準では関数、型、定数、モジュールは非公開
- `pub`キーワードで要素を後悔することができる
- `use`キーワードでモジュールやモジュール内の定義をスコープに入れることができる
## `mod`とファイルシステム
- `cargo new`の際に`--lib`オプションをつけると空のテストとしてモジュールを作成してくれる
- `mod`定義配下に`fn`として関数を定義する
```
{
    fn connect() {
    }
}
```
- この関数を`network`モジュール外から呼び出す場合、`network::connect()`の様に`::`を用いて呼び出す
- 同じ`src/lib.rs`内に複数のモジュールを用意することができる
```
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```
- 異なるモジュール内のため関数名がお互い衝突しない
- モジュール内にモジュールを作成することもできる
- モジュールが肥大化した場合に関連機能をまとめたり、機能を切り離す場合に有用
```
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```
- `network::connect()`と`network::client::connect`はどちらも`connect`だが名前空間が異なるため、干渉しない
- こうすることで下記の様な階層構造を作ることができる
```
communicator
 ├── network
 └── client

communicator
 └── network
     └── client
```
## モジュールを別ファイルに移す
- ファイルシステムの様になることで`src/lib.rs`や`src/main.rs`に全て入れなくてよくなる
```
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```
- 上記を階層構造として表すと以下の様になる
```
communicator
 ├── client
 └── network
     └── server
```
- `client`モジュールを抽出し、ここでは`client`の宣言のみに置き換えると下記の様になる
```
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```
- `mod client;`の行は以下の様な意味になる
```
mod client {
    // contents of client.rs
}
```
- ここまで準備ができたら`src/client.rs`ファイルに関数を作成していく
```
fn connect() {
}
```
- `src/lib.rs`にて`mod`を使って`client`モジュールを宣言しているため、ここでは`mod`宣言をする必要はない
- ここで`mod client`を記載すると`client`内に`client`サブモジュールを作成することになる
- また、コンパイルする際はバイナリクレートでなくライブラリクレートのため`cargo build`を使う
- 同様に`network`モジュールも抽出すると`src/lib.rs`は以下の様になる
```
mod client;

mod network;
```
- `src/network.rs`は下記に様になる
```
fn connect() {
}

mod server {
    fn connect() {
    }
}
```
- ここで`server`を抽出すると`src/network.rs`は以下の様にできる
```
fn connect() {
}

mod server;
```
- ここで`server.rs`を作成する場合は下記の様にする必要がある
  - 親モジュール名である`network`という新規ディレクトリを作成
	- `src/network.rs`ファイルを`network`ディレクトリに移動し、`src/network/mod.rs`にする
	- サブモジュールファイルとして`src/network/server.rs`を作成する
- モジュール構成
```
communicator
 ├── client
 └── network
     └── server
```
- ファイル構成
```
└── src
    ├── client.rs
    ├── lib.rs
    └── network
        ├── mod.rs
        └── server.rs
```
- `server.rs`ファイルが`src`ディレクトリに置けなかったのは`src`直下だと`server`が`network`のサブモジュールとすることを検知できないため
```
communicator
 ├── client
 └── network
     └── client
```
- 例として上記の様なモジュール構成があった時、`src`直下で検知しようとするとどこの`client`用なのか`network::client`用なのかわからなくなってしまう
## モジュールファイルシステムのまとめ
- 規則まとめ
  - `foo`という名前のモジュールにサブモジュールがなければ`foo`の定義は`foo.rs`に書くべき
  - `foo`というモジュールにサブモジュールがあったなら、`foo`の定義は`foo/mod.rs`に書くべき
- これらのルールは再帰的に適用される
- `foo`というモジュールに`bar`サブモジュールがあり、`bar`にはサブモジュールが無い場合ファイル構造は以下の様になる
```
├── foo
│   ├── bar.rs (`foo::bar`内の定義を含む)
│   └── mod.rs (`mod bar`を含む、`foo`内の定義を含む)
```
# `pub`で公開を制御する
- ファイル構造を整理することで`cargo build`は通ったが使用されていない警告が出る
- 外部からこのライブラリを呼び出して使用してみる
- まず作成したライブラリのsrc内に`main.rs`としてバイナリを作成する
```
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```
- `extern crate`でライブラリクレートをスコープに入れる
- これにより`communicator::client`や`communicator::network`が使える様になる
- `src/main.rs`はバイナリクレートのルートファイルとして扱われ、`src/lib.rs`とは区別される
- ここまで作成したモジュールは全て`communicator`モジュール内にあり、クレートトップの階層のモジュールをルートモジュールという
- プロジェクトのサブモジュール内で外部クレートを使用しているとしても`extern crate`はルートモジュール(`src/main.rs` `src/lib.rs`)に書くべき
- 現状`client`モジュールが非公開であるエラーが出る
- Rustでは全コードの初期状態は非公開となっている
- 初期状態では非公開のため自プログラムで使用されていなければコンパイラが未使用警告を出す
- 公開にすることでコンパイラが自分のプログラムで使用されるべきという要求をなくし、未使用警告を止める
## 関数を公開にする
- 公開にするには`pub`をつける
```
pub mod client;

mod network;
```
- そうすると`client`内の`connect()`が非公開であるエラーが出るため`client()`も`pub`にする
```
pub fn connect() {
}
```
- 未使用警告はライブラリ無いでこの関数を呼び出している箇所全てを誤って削除した際に検出できる様になる
- 他の関数も未使用警告がでており、公開APIにしたいため`src/network/mod.rs`にも`pub`をつける
```
pub fn connect() {
}

mod server;
```
- 要素の公開性は以下の様なルールになっている
  - 要素が公開なら、どの親モジュールを通してもアクセス可能
  - 要素が非公開なら、直接の親モジュールとその親の子モジュールのみアクセス可能
- 具体例
```
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```
- `outermost::middle_function()`は`pub`のためアクセス可能
- `outermost::middle_secret_function()`は`try_me`が`outermost`でも`outermost`の子モジュールでないためアクセスできない
- `outermost::inside::inner_function()`と`outermost::inside::secret_function()`は`inside`が非公開のため`try_me`からアクセスできない
# 異なるモジュール名を参照する
```
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules();
}
```
- 上記の様にモジュールをフルパス指定で関数を呼び出すと長ったらしくなる
## `use`キーワードで名前をスコープに導入する
- `use`キーワードを使うことで関数呼び出しを短縮できる
```
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```
- 上記は`use a::series::of;`によって`a::series::of`モジュールをバイナリクレートのルートスコープに持ってきている
- 関数ただ一つだけをスコープに入れたい場合は下記の様にする
```
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```
- 関数名まで入れることで直接関数を参照できる
- enumもある種の名前空間のため`use`でスコープに導入することができる
```
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```
- `Green`は`use`に含んでいないため`TrafficLight`名前空間を指定する
## Globで全ての名前をスコープに導入する
- ある名前空間全てを一度にスコープに導入するには`*`表記が使用できる
```
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::*;

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```
- `TrafficLight::*`は`TrafficLight`名前空間に存在する全ての公開要素をスコープに導入する
- 名前衝突などを考慮するとあまり使うべきでは無いが便利ではある
## `super`を使用して親モジュールにアクセスする
- 例で作成した中のモジュールとそれについての階層は以下の様になっている
```
pub mod client;

pub mod network;

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```
```
communicator
 ├── client
 ├── network
 |   └── client
 └── tests
```
- 他のモジュールに隣接する`tests`モジュールが存在し、`it_works`関数を含む
- `it_works`関数から`client::connect`関数を呼び出すことを考える
```
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```
- 上記だとコンパイルが失敗する
- 原因は、パスが現在のモジュールに相対的になり、ここでは`tests`になっているためである
- 唯一の例外は`use`文内であり、パスはクレートのルートに相対的になる
- `tests`モジュールは`client`モジュールがスコープに存在する必要がある
- 呼び出すためにはモジュール階層を一つ上がる必要がある
- `tests`モジュールにて先頭にコロンを使用することでルートからのフルパスで指定できる
```
::client::connect();
```
- あるいは`super`を使用してモジュール階層を上がることもできる
```
super::client::connect();
```
- この場合はコロンと`super`にて違いは無いが、階層が深くなると`super`で現在のモジュールからショートカットできる
- また、コロン指定をしているとモジュール階層を変更した際に、更新が煩雑になる
- 各テストで`super::`を書くのを省略する方法として`use`が使用できる
- `use`にて`super`を使うことでルートでなく親のモジュールから始めることができる
```
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```
