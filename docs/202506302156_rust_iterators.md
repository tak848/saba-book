# Rustのイテレータ完全ガイド

## 目次
1. [イテレータとは](#イテレータとは)
2. [基本的な使い方](#基本的な使い方)
3. [イテレータの作成方法](#イテレータの作成方法)
4. [主要なイテレータメソッド](#主要なイテレータメソッド)
5. [collect()の詳細](#collectの詳細)
6. [実践的な使用例](#実践的な使用例)
7. [パフォーマンスと最適化](#パフォーマンスと最適化)
8. [よくある間違いと対処法](#よくある間違いと対処法)

## イテレータとは

イテレータは、**値の列を順番に処理するための抽象化**です。Rustでは`Iterator`トレイトとして定義されています。

### 基本的な定義
```rust
pub trait Iterator {
    type Item;  // イテレータが生成する値の型
    
    fn next(&mut self) -> Option<Self::Item>;
    
    // 他にも多数のメソッドがデフォルト実装されている
}
```

### 特徴
- **遅延評価**: 必要になるまで計算を実行しない
- **メモリ効率的**: 全要素を一度にメモリに展開しない
- **コンポーザブル**: メソッドチェーンで複雑な処理を構築
- **型安全**: コンパイル時に型チェック

## 基本的な使い方

### 1. for文での使用（最も一般的）
```rust
let numbers = vec![1, 2, 3, 4, 5];

// Vecは直接イテレータとして使える
for num in numbers {
    println!("{}", num);
}
// 注意: numbersの所有権が移動する

// 参照のイテレータ
let numbers = vec![1, 2, 3, 4, 5];
for num in &numbers {  // または numbers.iter()
    println!("{}", num);  // numは&i32型
}
// numbersはまだ使える

// 可変参照のイテレータ
let mut numbers = vec![1, 2, 3, 4, 5];
for num in &mut numbers {  // または numbers.iter_mut()
    *num *= 2;  // 各要素を2倍にする
}
```

### 2. next()メソッドで手動制御
```rust
let mut iter = vec![1, 2, 3].into_iter();

// next()はOption<T>を返す
assert_eq!(iter.next(), Some(1));
assert_eq!(iter.next(), Some(2));
assert_eq!(iter.next(), Some(3));
assert_eq!(iter.next(), None);  // もう要素がない

// whileループでの使用
let mut iter = vec!["apple", "banana", "orange"].into_iter();
while let Some(fruit) = iter.next() {
    println!("{}", fruit);
}
```

### 3. matchでのパターンマッチング
```rust
let mut iter = "hello".chars();

match iter.next() {
    Some(ch) => println!("First char: {}", ch),
    None => println!("Empty string"),
}
```

## イテレータの作成方法

### 配列・ベクタからのイテレータ
```rust
// 配列
let arr = [1, 2, 3, 4, 5];
let iter1 = arr.iter();        // &i32のイテレータ
let iter2 = arr.into_iter();   // 配列の場合は&i32のイテレータ

// Vec
let vec = vec![1, 2, 3, 4, 5];
let iter1 = vec.iter();        // &i32のイテレータ
let iter2 = vec.into_iter();   // i32のイテレータ（所有権移動）
let iter3 = vec.iter_mut();    // &mut i32のイテレータ

// スライス
let slice = &[1, 2, 3, 4, 5];
let iter = slice.iter();       // &i32のイテレータ
```

### 文字列からのイテレータ
```rust
let text = "Hello, 世界!";

// 文字単位
let chars: Vec<char> = text.chars().collect();
// ['H', 'e', 'l', 'l', 'o', ',', ' ', '世', '界', '!']

// バイト単位
let bytes: Vec<u8> = text.bytes().collect();
// UTF-8エンコードされたバイト列

// 行単位
let multiline = "line1\nline2\nline3";
let lines: Vec<&str> = multiline.lines().collect();
// ["line1", "line2", "line3"]

// split系メソッド
let csv = "apple,banana,orange";
let items: Vec<&str> = csv.split(',').collect();
// ["apple", "banana", "orange"]

// splitn: 最大n個に分割
let path = "user/documents/file.txt";
let parts: Vec<&str> = path.splitn(2, '/').collect();
// ["user", "documents/file.txt"]
```

### 範囲（Range）からのイテレータ
```rust
// 基本的な範囲
for i in 0..5 {
    println!("{}", i);  // 0, 1, 2, 3, 4
}

// 包含的範囲
for i in 0..=5 {
    println!("{}", i);  // 0, 1, 2, 3, 4, 5
}

// ステップ付き
for i in (0..10).step_by(2) {
    println!("{}", i);  // 0, 2, 4, 6, 8
}

// 逆順
for i in (0..5).rev() {
    println!("{}", i);  // 4, 3, 2, 1, 0
}
```

### 無限イテレータ
```rust
// 繰り返し
let mut repeat = std::iter::repeat("Hello");
assert_eq!(repeat.next(), Some("Hello"));
assert_eq!(repeat.next(), Some("Hello"));
// 永遠に"Hello"を返す

// サイクル
let mut cycle = vec![1, 2, 3].into_iter().cycle();
assert_eq!(cycle.next(), Some(1));
assert_eq!(cycle.next(), Some(2));
assert_eq!(cycle.next(), Some(3));
assert_eq!(cycle.next(), Some(1));  // 最初に戻る

// カウントアップ
let count = 0..;  // 0から無限にカウント
for (i, val) in count.zip(&[10, 20, 30]) {
    println!("{}: {}", i, val);  // 0: 10, 1: 20, 2: 30
}
```

## 主要なイテレータメソッド

### 変換系メソッド

#### map: 各要素を変換
```rust
let numbers = vec![1, 2, 3, 4, 5];

// 各要素を2倍にする
let doubled: Vec<i32> = numbers.iter()
    .map(|x| x * 2)
    .collect();
// [2, 4, 6, 8, 10]

// 文字列の長さを取得
let words = vec!["hello", "world", "rust"];
let lengths: Vec<usize> = words.iter()
    .map(|s| s.len())
    .collect();
// [5, 5, 4]

// 複雑な変換
struct Person {
    name: String,
    age: u32,
}

let people = vec![
    Person { name: "Alice".to_string(), age: 30 },
    Person { name: "Bob".to_string(), age: 25 },
];

let names: Vec<String> = people.iter()
    .map(|p| p.name.clone())
    .collect();
// ["Alice", "Bob"]
```

#### flat_map: mapして平坦化
```rust
let sentences = vec!["Hello world", "Rust programming"];

// 各文を単語に分割して平坦化
let words: Vec<&str> = sentences.iter()
    .flat_map(|s| s.split_whitespace())
    .collect();
// ["Hello", "world", "Rust", "programming"]

// ネストした配列を平坦化
let nested = vec![vec![1, 2], vec![3, 4, 5]];
let flat: Vec<i32> = nested.into_iter()
    .flat_map(|v| v)
    .collect();
// [1, 2, 3, 4, 5]
```

### フィルタリング系メソッド

#### filter: 条件に合う要素だけ残す
```rust
let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 偶数だけ
let evens: Vec<i32> = numbers.iter()
    .filter(|x| *x % 2 == 0)
    .copied()  // &i32 を i32 に変換
    .collect();
// [2, 4, 6, 8, 10]

// 複数条件
let result: Vec<i32> = numbers.iter()
    .filter(|x| *x % 2 == 0 && *x > 5)
    .copied()
    .collect();
// [6, 8, 10]
```

#### filter_map: filterとmapを同時に
```rust
let strings = vec!["1", "2", "abc", "4", "xyz"];

// 数値に変換できるものだけを変換
let numbers: Vec<i32> = strings.iter()
    .filter_map(|s| s.parse().ok())
    .collect();
// [1, 2, 4]

// Optionを返す関数と組み合わせ
fn get_even(n: i32) -> Option<i32> {
    if n % 2 == 0 { Some(n) } else { None }
}

let nums = vec![1, 2, 3, 4, 5, 6];
let evens: Vec<i32> = nums.into_iter()
    .filter_map(get_even)
    .collect();
// [2, 4, 6]
```

### 集約系メソッド

#### fold: 畳み込み（reduce）
```rust
let numbers = vec![1, 2, 3, 4, 5];

// 合計を計算
let sum = numbers.iter()
    .fold(0, |acc, x| acc + x);
// 15

// 文字列の結合
let words = vec!["Hello", " ", "World"];
let sentence = words.iter()
    .fold(String::new(), |mut acc, s| {
        acc.push_str(s);
        acc
    });
// "Hello World"

// 最大値を見つける
let max = numbers.iter()
    .fold(i32::MIN, |max, &x| if x > max { x } else { max });
// 5
```

#### reduce: 初期値なしの畳み込み
```rust
let numbers = vec![1, 2, 3, 4, 5];

// 合計（Option<T>を返す）
let sum = numbers.iter()
    .copied()
    .reduce(|a, b| a + b);
// Some(15)

// 空の場合
let empty: Vec<i32> = vec![];
let sum = empty.iter()
    .copied()
    .reduce(|a, b| a + b);
// None
```

### その他の便利なメソッド

#### take/skip: 一部を取り出す
```rust
let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 最初の3つ
let first_three: Vec<i32> = numbers.iter()
    .take(3)
    .copied()
    .collect();
// [1, 2, 3]

// 最初の3つをスキップ
let skip_three: Vec<i32> = numbers.iter()
    .skip(3)
    .copied()
    .collect();
// [4, 5, 6, 7, 8, 9, 10]

// 組み合わせ：4番目から6番目まで
let middle: Vec<i32> = numbers.iter()
    .skip(3)
    .take(3)
    .copied()
    .collect();
// [4, 5, 6]
```

#### zip: 2つのイテレータを組み合わせる
```rust
let names = vec!["Alice", "Bob", "Charlie"];
let ages = vec![30, 25, 35];

// タプルのベクタを作成
let people: Vec<(&str, i32)> = names.iter()
    .zip(ages.iter())
    .map(|(name, age)| (*name, *age))
    .collect();
// [("Alice", 30), ("Bob", 25), ("Charlie", 35)]

// 長さが違う場合は短い方に合わせる
let short = vec![1, 2];
let long = vec![10, 20, 30, 40];
let pairs: Vec<(i32, i32)> = short.iter()
    .zip(long.iter())
    .map(|(a, b)| (*a, *b))
    .collect();
// [(1, 10), (2, 20)]
```

#### enumerate: インデックス付き
```rust
let fruits = vec!["apple", "banana", "orange"];

for (index, fruit) in fruits.iter().enumerate() {
    println!("{}: {}", index, fruit);
}
// 0: apple
// 1: banana
// 2: orange

// インデックスを1から始める
let indexed: Vec<(usize, &str)> = fruits.iter()
    .enumerate()
    .map(|(i, s)| (i + 1, *s))
    .collect();
// [(1, "apple"), (2, "banana"), (3, "orange")]
```

#### find/position: 検索
```rust
let numbers = vec![1, 2, 3, 4, 5];

// 条件に合う最初の要素
let first_even = numbers.iter()
    .find(|x| *x % 2 == 0);
// Some(&2)

// 条件に合う最初の要素のインデックス
let position = numbers.iter()
    .position(|x| *x == 3);
// Some(2)

// 見つからない場合
let not_found = numbers.iter()
    .find(|x| *x > 10);
// None
```

#### any/all: 条件チェック
```rust
let numbers = vec![2, 4, 6, 8, 10];

// いずれかが条件を満たすか
let has_odd = numbers.iter()
    .any(|x| x % 2 != 0);
// false

// すべてが条件を満たすか
let all_even = numbers.iter()
    .all(|x| x % 2 == 0);
// true
```

## collect()の詳細

### 基本的な使い方
```rust
// Vec<T>への収集
let vec: Vec<i32> = (1..=5).collect();

// HashSetへの収集
use std::collections::HashSet;
let set: HashSet<i32> = vec![1, 2, 2, 3, 3, 3].into_iter().collect();
// {1, 2, 3}

// HashMapへの収集
use std::collections::HashMap;
let pairs = vec![("a", 1), ("b", 2), ("c", 3)];
let map: HashMap<&str, i32> = pairs.into_iter().collect();
// {"a": 1, "b": 2, "c": 3}
```

### Result/Optionの収集
```rust
// すべて成功の場合
let strings = vec!["1", "2", "3"];
let numbers: Result<Vec<i32>, _> = strings.iter()
    .map(|s| s.parse::<i32>())
    .collect();
// Ok([1, 2, 3])

// 一つでも失敗があれば全体が失敗
let strings = vec!["1", "2", "abc"];
let numbers: Result<Vec<i32>, _> = strings.iter()
    .map(|s| s.parse::<i32>())
    .collect();
// Err(ParseIntError { ... })

// Optionも同様
let options = vec![Some(1), Some(2), Some(3)];
let collected: Option<Vec<i32>> = options.into_iter().collect();
// Some([1, 2, 3])

let options = vec![Some(1), None, Some(3)];
let collected: Option<Vec<i32>> = options.into_iter().collect();
// None
```

### 文字列への収集
```rust
// charsから文字列
let chars = vec!['H', 'e', 'l', 'l', 'o'];
let string: String = chars.iter().collect();
// "Hello"

// 文字列スライスの結合
let words = vec!["Hello", " ", "World"];
let sentence: String = words.iter()
    .copied()
    .collect();
// "Hello World"

// joinを使った結合
let words = vec!["apple", "banana", "orange"];
let csv = words.join(",");
// "apple,banana,orange"
```

### 型推論とturbofish
```rust
// 型注釈
let vec: Vec<i32> = (1..=5).collect();

// turbofish構文
let vec = (1..=5).collect::<Vec<i32>>();

// 部分的な型指定
let vec: Vec<_> = (1..=5).collect();  // 要素の型は推論
```

## 実践的な使用例

### CSVデータの処理
```rust
let csv_data = "name,age,city
Alice,30,Tokyo
Bob,25,Osaka
Charlie,35,Kyoto";

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    city: String,
}

let people: Vec<Person> = csv_data
    .lines()
    .skip(1)  // ヘッダーをスキップ
    .filter_map(|line| {
        let parts: Vec<&str> = line.split(',').collect();
        if parts.len() == 3 {
            Some(Person {
                name: parts[0].to_string(),
                age: parts[1].parse().ok()?,
                city: parts[2].to_string(),
            })
        } else {
            None
        }
    })
    .collect();
```

### ファイルパスの処理
```rust
let file_paths = vec![
    "/home/user/doc.txt",
    "/home/user/image.png",
    "/home/user/data.csv",
    "/home/user/photo.jpg",
];

// 拡張子でフィルタリング
let images: Vec<&str> = file_paths.iter()
    .filter(|path| {
        path.ends_with(".png") || path.ends_with(".jpg")
    })
    .copied()
    .collect();
// ["/home/user/image.png", "/home/user/photo.jpg"]

// ファイル名だけ抽出
let filenames: Vec<&str> = file_paths.iter()
    .filter_map(|path| path.split('/').last())
    .collect();
// ["doc.txt", "image.png", "data.csv", "photo.jpg"]
```

### 統計計算
```rust
let scores = vec![85, 92, 78, 95, 88, 91, 87];

// 平均値
let average: f64 = scores.iter()
    .map(|&x| x as f64)
    .sum::<f64>() / scores.len() as f64;

// 最大値・最小値
let max = scores.iter().max().unwrap();
let min = scores.iter().min().unwrap();

// 標準偏差
let variance: f64 = scores.iter()
    .map(|&x| {
        let diff = x as f64 - average;
        diff * diff
    })
    .sum::<f64>() / scores.len() as f64;
let std_dev = variance.sqrt();
```

### グループ化
```rust
use std::collections::HashMap;

#[derive(Debug)]
struct Student {
    name: String,
    grade: char,
}

let students = vec![
    Student { name: "Alice".to_string(), grade: 'A' },
    Student { name: "Bob".to_string(), grade: 'B' },
    Student { name: "Charlie".to_string(), grade: 'A' },
    Student { name: "David".to_string(), grade: 'B' },
];

// 成績でグループ化
let mut grouped: HashMap<char, Vec<String>> = HashMap::new();
for student in students {
    grouped.entry(student.grade)
        .or_insert_with(Vec::new)
        .push(student.name);
}
// {'A': ["Alice", "Charlie"], 'B': ["Bob", "David"]}
```

## パフォーマンスと最適化

### 遅延評価の利点
```rust
// 悪い例：不要な中間Vecを作成
let result: Vec<i32> = vec![1, 2, 3, 4, 5]
    .into_iter()
    .map(|x| x * 2)
    .collect::<Vec<i32>>()  // 中間Vec
    .into_iter()
    .filter(|x| x > 5)
    .collect();

// 良い例：一度のイテレーションで処理
let result: Vec<i32> = vec![1, 2, 3, 4, 5]
    .into_iter()
    .map(|x| x * 2)
    .filter(|x| x > 5)
    .collect();
```

### iter() vs into_iter()
```rust
let vec = vec![1, 2, 3];

// iter(): 参照のイテレータ（Vecは残る）
let sum1: i32 = vec.iter().sum();
println!("{:?}", vec);  // OK: vecはまだ使える

// into_iter(): 所有権を取るイテレータ（Vecは消費される）
let vec2 = vec![1, 2, 3];
let sum2: i32 = vec2.into_iter().sum();
// println!("{:?}", vec2);  // エラー: vec2は移動済み
```

### collect()の容量指定
```rust
// 悪い例：再割り当てが発生する可能性
let result: Vec<i32> = (0..1000)
    .filter(|x| x % 2 == 0)
    .collect();

// 良い例：事前に容量を確保
let result: Vec<i32> = (0..1000)
    .filter(|x| x % 2 == 0)
    .collect::<Vec<_>>()
    .into_iter()
    .collect::<Vec<_>>();

// または with_capacity を使用
let mut result = Vec::with_capacity(500);
result.extend((0..1000).filter(|x| x % 2 == 0));
```

### 並列イテレータ（Rayon）
```rust
use rayon::prelude::*;

let numbers: Vec<i32> = (0..1_000_000).collect();

// 通常のイテレータ
let sum1: i32 = numbers.iter()
    .map(|x| x * x)
    .sum();

// 並列イテレータ
let sum2: i32 = numbers.par_iter()
    .map(|x| x * x)
    .sum();
```

## よくある間違いと対処法

### 1. 借用チェッカーエラー
```rust
// エラー: moveされたVecを使おうとしている
let vec = vec![1, 2, 3];
let doubled = vec.into_iter().map(|x| x * 2).collect::<Vec<_>>();
// println!("{:?}", vec);  // エラー

// 解決法: iter()を使う
let vec = vec![1, 2, 3];
let doubled: Vec<i32> = vec.iter().map(|x| x * 2).collect();
println!("{:?}", vec);  // OK
```

### 2. 型推論エラー
```rust
// エラー: collectの結果の型が不明
// let result = vec![1, 2, 3].iter().map(|x| x * 2).collect();

// 解決法1: 型注釈
let result: Vec<i32> = vec![1, 2, 3].iter().map(|x| x * 2).collect();

// 解決法2: turbofish
let result = vec![1, 2, 3].iter().map(|x| x * 2).collect::<Vec<i32>>();
```

### 3. ライフタイムエラー
```rust
// エラー: 一時的な値への参照
// let iter = vec![1, 2, 3].iter();

// 解決法: 変数に束縛
let vec = vec![1, 2, 3];
let iter = vec.iter();
```

### 4. 無限ループ
```rust
// 危険: 無限イテレータをcollect
// let all: Vec<i32> = (0..).collect();  // メモリ不足でクラッシュ

// 解決法: takeで制限
let first_100: Vec<i32> = (0..).take(100).collect();
```

### 5. パフォーマンスの問題
```rust
// 非効率: 複数回のイテレーション
let vec = vec![1, 2, 3, 4, 5];
let sum = vec.iter().sum::<i32>();
let count = vec.iter().count();
let avg = sum as f64 / count as f64;

// 効率的: 一度のイテレーション
let (sum, count) = vec.iter()
    .fold((0, 0), |(sum, count), x| (sum + x, count + 1));
let avg = sum as f64 / count as f64;
```

## まとめ

Rustのイテレータは、関数型プログラミングの概念を取り入れた強力な機能です。主な利点は：

1. **遅延評価**により、必要な時にだけ計算を実行
2. **メソッドチェーン**で読みやすく表現力豊かなコード
3. **型安全性**により、コンパイル時にエラーを検出
4. **ゼロコスト抽象化**で、手書きループと同等の性能

イテレータを使いこなすことで、より安全で効率的、かつ表現力豊かなRustコードを書くことができます。特に`collect()`は、イテレータの結果を様々なコレクション型に変換できる強力なメソッドです。

実際の開発では、単純なループの代わりにイテレータメソッドを使うことで、意図が明確で保守しやすいコードを書くことができます。