# match
- enumで返ってくる結果に対して条件分岐する
- 例: guessとsecret\_numberを比較して等しい、小さい、大きいで出力を変える
```
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!"),
}
```

## パターン
- case文のようなもの
- 木のような構造を持つ値の中から特定のパターン合致を探し、処理を分岐させる
### 単純なパターンマッチ
```
fn foo() -> uint {
    // ...
		}

		let x: uint = foo();
		match x {
		    0 => println!("nothing"),
				    1 => println!("only one"),
						    _ => println!("many")
								}
```

### 厄介なパターン
```
let x = 'x';
let c = 'c';

match c {
    x => println!("x: {} c: {}", x, c),
}

println!("x: {}", x);
```
- 出力
```
x: c c: c
x: c
```
- パターンマッチの腕内で束縛を使用した場合、その束縛にはmatchの対象とした束縛が入るためxにはcが入った
