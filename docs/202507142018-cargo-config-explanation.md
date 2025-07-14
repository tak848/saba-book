# Cargo設定の解説

このドキュメントでは、`Cargo.toml`に追加された設定について、RustとCargo初心者向けに解説します。

## 変更内容の概要

`git diff`で確認できる主な変更点：
- ワークスペース構成の拡張
- フィーチャーフラグの導入
- 条件付き依存関係の設定
- バイナリターゲットの明示的な設定

## 各設定の詳細説明

### 1. ワークスペース設定の変更

```toml
workspace = { members = ["net/wasabi", "saba_core"] }
```

**解説：**
- `workspace`は複数のクレート（パッケージ）を一つのプロジェクトで管理する仕組みです
- `members`に`net/wasabi`が追加されました（元々は`saba_core`のみ）
- これにより、`net/wasabi`ディレクトリ内のクレートも一緒にビルド・管理されます
- ワークスペースを使うメリット：
  - 依存関係の共有
  - 一括ビルド・テスト
  - 共通の`target`ディレクトリの使用

### 2. デフォルト実行ファイルの指定

```toml
default-run = "saba"
```

**解説：**
- プロジェクトに複数の実行可能ファイルがある場合、`cargo run`で実行されるデフォルトを指定します
- 明示的に指定しない場合は、`cargo run --bin saba`のように指定する必要があります

### 3. フィーチャー（機能フラグ）の追加

```toml
[features]
default = ["wasabi"]
wasabi = ["dep:net_wasabi", "dep:noli"]
```

**解説：**
- `features`は条件付きコンパイルの仕組みです
- `default`フィーチャーは何も指定しない時に有効になります（ここでは`wasabi`が有効）
- `wasabi`フィーチャーは`net_wasabi`と`noli`クレートを有効にします
- `dep:`プレフィックスは、その依存関係を有効にすることを意味します

**使用例：**
```bash
# デフォルト（wasabiフィーチャー有効）でビルド
cargo build

# wasabiフィーチャーを無効にしてビルド
cargo build --no-default-features

# 特定のフィーチャーを指定してビルド
cargo build --features "wasabi"
```

### 4. バイナリターゲットの明示的な設定

```toml
[[bin]]
name = "saba"
path = "src/main.rs"
required-features = ["wasabi"]
```

**解説：**
- `[[bin]]`は実行可能ファイルの設定です（`[[]]`は配列要素を表します）
- `name`：バイナリの名前
- `path`：ソースファイルのパス
- `required-features`：このバイナリは`wasabi`フィーチャーが有効な時のみビルドされます

### 5. 依存関係（dependencies）の変更

```toml
saba_core = { path = "./saba_core" }
net_wasabi = { path = "./net/wasabi", optional = true }
noli = { git = "https://github.com/hikalium/wasabi.git", branch = "for_saba", optional = true }
```

**解説：**
- `saba_core`：ローカルの`saba_core`ディレクトリのクレートを依存関係に追加
  - `path`：ローカルパスを指定
- `net_wasabi`：ローカルの`net/wasabi`ディレクトリのクレート
  - `optional = true`：フィーチャーが有効な時のみ使用
- `noli`：GitHubから取得
  - `git`：GitリポジトリのURL
  - `branch`：使用するブランチ
  - `optional = true`：フィーチャーが有効な時のみ使用

## 重要なポイント

1. **オプショナル依存関係**
   - `optional = true`の依存関係は、対応するフィーチャーが有効な時のみ使用されます
   - メモリ使用量やビルド時間の削減に役立ちます

2. **フィーチャーの活用**
   - この設定により、Wasabi OS環境用のビルドと通常のビルドを切り替えられます
   - 開発中は必要な機能だけを有効にして作業できます

3. **ワークスペースの利点**
   - 複数のクレートを効率的に管理
   - 共通の依存関係のバージョン管理が容易

## よくある使用パターン

```bash
# 通常のビルド（Wasabi機能あり）
cargo build

# Wasabi機能なしでビルド（軽量版）
cargo build --no-default-features

# テストの実行（ワークスペース全体）
cargo test --all

# 特定のクレートのみビルド
cargo build -p saba_core
```

この設定により、プロジェクトがより柔軟になり、異なる環境や用途に応じたビルドが可能になりました。