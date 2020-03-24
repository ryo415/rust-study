# 繰り返し
## 繰り返しの種類
- loop
```
loop {
}
```
- while
```
while true {
}
```
- for
```
for x in 0..10 {
}
```

## 列挙
- .enumerate()でカウントができる
```
for (i, j) in (5..10).enumerate() {
  // i: カウント
	// j: 数値
  println!("i = {} and j = {}", i, j);
}
```

## ループラベル
- ループの手前にコロンでラベルをつけ、break,continue後にラベルを指定することでそのループに対してbreak,continueが適用できる
```
'outer: for x in 0..10 {
    'inner: for y in 0..10 {
        if x % 2 == 0 { continue 'outer; } // x のループを継続
        if y % 2 == 0 { continue 'inner; } // y のループを継続
        println!("x: {}, y: {}", x, y);
    }
}
```
