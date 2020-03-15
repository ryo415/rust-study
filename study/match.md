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
