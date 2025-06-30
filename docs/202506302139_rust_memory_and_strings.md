# Rust のメモリ管理と文字列型について

## 目次

1. [no_std と alloc クレート](#no_stdとallocクレート)
2. [メモリ割り当ての種類](#メモリ割り当ての種類)
3. [文字列型の詳細](#文字列型の詳細)
4. [他言語との比較](#他言語との比較)

## no_std と alloc クレート

### #![no_std]とは

`#![no_std]`は、Rust 標準ライブラリ（`std`）を使用しないことを宣言するアトリビュートです。

#### 使用する場面

- 組み込みシステム開発
- カーネル開発
- WebAssembly
- ベアメタルプログラミング
- 最小限のバイナリサイズが必要な場合

#### 制限事項

標準ライブラリを使わないことで以下が使えなくなります：

- `std::thread`（スレッド）
- `std::fs`（ファイルシステム）
- `std::net`（ネットワーク）
- `std::process`（プロセス操作）
- `std::env`（環境変数）

### alloc クレート

`alloc`クレートは、OS に依存しないヒープメモリ割り当て機能を提供します。

```rust
#![no_std]
extern crate alloc;

use alloc::vec::Vec;
use alloc::string::String;
use alloc::boxed::Box;
```

#### 利用可能な主な型

- `Vec<T>`: 動的配列
- `String`: 可変長文字列
- `Box<T>`: ヒープ上の単一値
- `Rc<T>`: 参照カウント型スマートポインタ
- `Arc<T>`: アトミック参照カウント型（スレッドセーフ）

## メモリ割り当ての種類

### Rust の alloc クレート

Rust ネイティブのメモリ管理システムで、所有権システムと統合されています。

```rust
use alloc::vec::Vec;
use alloc::string::String;
use alloc::boxed::Box;

// Vec<T>の例
let mut numbers = Vec::new();
numbers.push(42);
numbers.push(100);
// スコープを抜けると自動的に解放

// Stringの例
let mut text = String::from("Hello");
text.push_str(", World!");
// 所有権が移動するか、スコープを抜けると解放

// Box<T>の例
let boxed_value = Box::new(42);
// Dropトレイトにより自動解放
```

#### メリット

- メモリ安全性が保証される
- 二重解放やメモリリークを防ぐ
- 所有権システムによる自動管理
- ゼロコスト抽象化

### C 言語の malloc

C 標準ライブラリのメモリ割り当て関数です。

```rust
use core::alloc::{GlobalAlloc, Layout};
use libc;

unsafe {
    // 100バイトのメモリを割り当て
    let layout = Layout::from_size_align(100, 1).unwrap();
    let ptr = libc::malloc(layout.size()) as *mut u8;

    if ptr.is_null() {
        // 割り当て失敗の処理
        panic!("Memory allocation failed");
    }

    // メモリを使用（初期化されていないので注意）
    *ptr = 42;

    // 必ず手動で解放
    libc::free(ptr as *mut libc::c_void);
}
```

#### 特徴

- 初期化されていないメモリを返す（ゴミ値が含まれる）
- 手動で`free`を呼ぶ必要がある
- メモリリークや二重解放のリスク
- FFI で C 言語ライブラリと連携する際に使用

### C 言語の calloc

要素数とサイズを指定してゼロ初期化されたメモリを割り当てます。

```rust
unsafe {
    // 10個のi32（各4バイト）をゼロで初期化して割り当て
    let count = 10;
    let size = std::mem::size_of::<i32>();
    let ptr = libc::calloc(count, size) as *mut i32;

    if ptr.is_null() {
        panic!("Memory allocation failed");
    }

    // すべての要素が0で初期化されている
    for i in 0..count {
        assert_eq!(*ptr.add(i), 0);
    }

    // 必ず手動で解放
    libc::free(ptr as *mut libc::c_void);
}
```

#### malloc との違い

- メモリがゼロで初期化される
- 要素数 × サイズで指定（配列向き）
- わずかに遅い（初期化のオーバーヘッド）

### 使い分けガイドライン

| ケース                     | 推奨               | 理由                   |
| -------------------------- | ------------------ | ---------------------- |
| 通常の Rust 開発           | `alloc`クレート    | メモリ安全性、自動管理 |
| FFI（C 言語連携）          | `malloc`/`calloc`  | C 言語との互換性       |
| パフォーマンスクリティカル | 状況次第           | ベンチマークで判断     |
| 組み込み開発               | カスタムアロケータ | 制約に応じて選択       |

## 文字列型の詳細

### str（文字列スライス）

```rust
// &str型の特徴
let s1: &str = "Hello";  // 静的文字列リテラル
let s2: &'static str = "World";  // 明示的にstaticライフタイム

// Stringから&strへの変換
let owned = String::from("Rust");
let borrowed: &str = &owned;  // または owned.as_str()
```

#### 内部構造

```rust
// &strは実質的に以下の構造
struct StrSlice {
    ptr: *const u8,  // UTF-8バイト列へのポインタ
    len: usize,      // バイト長
}
```

#### 特徴

- **不変参照**：内容を変更できない
- **借用型**：データを所有しない
- **UTF-8 保証**：常に有効な UTF-8
- **サイズ**：コンパイル時にサイズ不明（`?Sized`）

### String（所有型文字列）

```rust
// Stringの作成方法
let s1 = String::new();
let s2 = String::from("Hello");
let s3 = "World".to_string();
let s4 = "Rust".to_owned();
let s5 = format!("{} {}", "Hello", "World");

// 可変操作
let mut s = String::from("Hello");
s.push(' ');          // 1文字追加
s.push_str("World");  // 文字列追加
s.insert(5, ',');     // 指定位置に挿入
s.remove(5);          // 指定位置から削除
```

#### 内部構造

```rust
// Stringは実質的にVec<u8>のラッパー
struct String {
    vec: Vec<u8>,  // UTF-8バイト列を格納するベクタ
}

// Vec<u8>の構造
struct Vec<u8> {
    ptr: *mut u8,    // ヒープ上のデータへのポインタ
    len: usize,      // 現在の長さ
    capacity: usize, // 割り当て済み容量
}
```

#### メモリレイアウト

```
スタック:        ヒープ:
┌─────────┐     ┌─────────────────┐
│ ptr     │────>│ H | e | l | l | o |
├─────────┤     └─────────────────┘
│ len: 5  │
├─────────┤
│ cap: 10 │
└─────────┘
```

### 変換メソッドの詳細

#### &str → String

```rust
let s: &str = "hello";

// 以下はすべて同じ結果
let string1 = s.to_string();
let string2 = String::from(s);
let string3 = s.to_owned();
let string4 = format!("{}", s);

// パフォーマンス: String::from() ≈ to_owned() > to_string() > format!()
```

#### String → &str

```rust
let string = String::from("hello");

// 以下はすべて同じ結果
let str1: &str = &string;
let str2: &str = string.as_str();
let str3: &str = string.as_ref();  // AsRef<str>トレイト
let str4: &str = &string[..];      // スライス記法
```

### char 型（Unicode 文字）

```rust
// charは32ビット（4バイト）のUnicodeスカラー値
let c1: char = 'A';     // ASCII
let c2: char = 'あ';    // 日本語
let c3: char = '🦀';    // 絵文字

// charの操作
let c = 'a';
let upper = c.to_uppercase();  // 'A'
let is_alpha = c.is_alphabetic();  // true
let digit = '5'.to_digit(10);  // Some(5)

// 文字列との相互変換
let mut s = String::from("Hello");
s.push('!');  // char を追加

// イテレーション
for ch in "🦀Rust".chars() {
    println!("{}", ch);  // 🦀, R, u, s, t
}
```

#### UTF-8 エンコーディングの例

```rust
let s = "A";      // 1バイト: 0x41
let s = "あ";     // 3バイト: 0xE3 0x81 0x82
let s = "🦀";     // 4バイト: 0xF0 0x9F 0xA6 0x80

// バイト数とchar数の違い
let text = "Hello🦀";
assert_eq!(text.len(), 9);        // バイト数
assert_eq!(text.chars().count(), 6);  // 文字数
```

## 他言語との比較

### 文字型の比較

| 言語   | 1 文字型 | サイズ   | 説明                 |
| ------ | -------- | -------- | -------------------- |
| Rust   | `char`   | 4 バイト | Unicode スカラー値   |
| Go     | `rune`   | 4 バイト | int32 のエイリアス   |
| Java   | `char`   | 2 バイト | UTF-16 コード単位    |
| C      | `char`   | 1 バイト | ASCII またはバイト   |
| Python | なし     | -        | 文字列の要素も文字列 |

### 文字列型の比較

| 言語   | 不変文字列    | 可変文字列      | 特徴                   |
| ------ | ------------- | --------------- | ---------------------- |
| Rust   | `&str`        | `String`        | UTF-8、所有権システム  |
| Go     | `string`      | `[]byte`        | UTF-8、不変            |
| Java   | `String`      | `StringBuilder` | UTF-16、不変           |
| C      | `const char*` | `char*`         | エンコーディング未定義 |
| Python | `str`         | なし            | UTF-8（Python3）、不変 |

### メモリ管理の比較

| 言語   | 管理方式        | 特徴                           |
| ------ | --------------- | ------------------------------ |
| Rust   | 所有権システム  | コンパイル時チェック、自動解放 |
| Go     | GC              | ランタイム GC、一時停止あり    |
| Java   | GC              | 世代別 GC、チューニング可能    |
| C      | 手動            | malloc/free、プログラマ責任    |
| Python | 参照カウント+GC | 即座に解放、循環参照は GC      |

## ベストプラクティス

### 文字列を扱う際の指針

1. **関数の引数**: `&str`を使う

```rust
fn process_text(text: &str) {
    // StringもVec<u8>も受け取れる
}
```

2. **関数の戻り値**: 状況に応じて選択

```rust
// 新しい文字列を作る場合
fn create_message() -> String {
    format!("Hello, {}", "World")
}

// 既存の文字列を参照する場合
fn get_prefix(s: &str) -> &str {
    &s[..5]
}
```

3. **構造体のフィールド**: ライフタイムを考慮

```rust
// 所有する場合
struct Message {
    content: String,
}

// 借用する場合（ライフタイム注釈が必要）
struct MessageRef<'a> {
    content: &'a str,
}
```

### パフォーマンスの考慮事項

1. **不要なアロケーションを避ける**

```rust
// 悪い例
let s = "hello".to_string() + " " + "world";

// 良い例
let s = format!("hello {}", "world");
// または
let mut s = String::with_capacity(11);
s.push_str("hello ");
s.push_str("world");
```

2. **容量を事前に確保**

```rust
let mut s = String::with_capacity(100);
// 100バイト分のメモリを事前確保
```

3. **Cow（Clone on Write）の活用**

```rust
use std::borrow::Cow;

fn process(input: &str) -> Cow<str> {
    if input.contains("old") {
        Cow::Owned(input.replace("old", "new"))
    } else {
        Cow::Borrowed(input)  // クローンしない
    }
}
```

## まとめ

Rust のメモリ管理と文字列型は、安全性とパフォーマンスを両立させる設計になっています。`no_std`環境でも`alloc`クレートを使うことで、動的メモリ割り当てが可能です。文字列型は`&str`（借用）と`String`（所有）の使い分けが重要で、それぞれ適切な場面があります。他の言語から来た開発者は、特に所有権システムと借用の概念に慣れる必要がありますが、これによりメモリ安全性が保証されます。
