# Rust no_std環境のGitHub Actions CI構築ガイド

## 目次
1. [no_std環境でのビルドエラーと原因](#no_std環境でのビルドエラーと原因)
2. [rust-toolchain.tomlの重要性](#rust-toolchaintomlの重要性)
3. [GitHub Actionsでのrust-toolchain.toml活用](#github-actionsでのrust-toolchaintoml活用)
4. [no_std環境でのCI設定](#no_std環境でのci設定)
5. [トラブルシューティング](#トラブルシューティング)

## no_std環境でのビルドエラーと原因

### 典型的なエラー
```
error[E0463]: can't find crate for `core`
  |
  = note: the `wasm32-unknown-unknown` target may not be installed
  = help: consider downloading the target with `rustup target add wasm32-unknown-unknown`
  = help: consider building the standard library from source with `cargo build -Zbuild-std`

error[E0463]: can't find crate for `alloc`
```

### エラーの原因
no_std環境では、標準ライブラリ（`std`）を使用しないため、以下の問題が発生します：

1. **`core`クレートが見つからない**
   - `core`は言語の基本機能を提供するクレート
   - ターゲットプラットフォーム用にビルドされていない

2. **`alloc`クレートが見つからない**
   - ヒープメモリ割り当てを提供するクレート
   - 標準配布には含まれていない場合がある

3. **言語アイテムの不足**
   - `#[derive]`などの基本的な機能が使えない
   - `Result`、`Option`などの基本型が定義されていない

## rust-toolchain.tomlの重要性

### Single Source of Truth（単一の真実の源）
`rust-toolchain.toml`は、プロジェクトで使用するRustツールチェーンのバージョンと設定を定義するファイルです。

```toml
[toolchain]
channel = "nightly-2024-01-01"
components = ["rustfmt", "rust-src"]
targets = ["x86_64-unknown-linux-gnu"]
profile = "default"
```

### 利点
1. **一貫性の確保**
   - 開発者全員が同じバージョンを使用
   - CI環境と開発環境の統一

2. **自動的な適用**
   - `cargo`コマンド実行時に自動的に適用
   - 手動でのツールチェーン切り替えが不要

3. **バージョン管理**
   - Gitでバージョン管理可能
   - プロジェクトの履歴と共に追跡

### 注意点
⚠️ **プロジェクト固有の設定を変更しない**
- 書籍や教材に従っている場合は、指定されたバージョンを維持
- 特にnightlyバージョンは機能や動作が変わる可能性がある

## GitHub Actionsでのrust-toolchain.toml活用

### 基本的な使用方法
```yaml
- name: Install Rust
  uses: dtolnay/rust-toolchain@stable
```

このアクションは自動的に`rust-toolchain.toml`を検出し、指定された設定を使用します。

### 追加ターゲットのインストール
rust-toolchain.tomlに含まれていないターゲットが必要な場合：

```yaml
- name: Add wasm32 target
  run: rustup target add wasm32-unknown-unknown
```

### キャッシュの設定
ビルド時間短縮のためのキャッシュ設定：

```yaml
- name: Cache cargo registry
  uses: actions/cache@v4
  with:
    path: ~/.cargo/registry
    key: ${{ runner.os }}-cargo-registry-${{ hashFiles('**/Cargo.lock') }}

- name: Cache cargo index
  uses: actions/cache@v4
  with:
    path: ~/.cargo/git
    key: ${{ runner.os }}-cargo-index-${{ hashFiles('**/Cargo.lock') }}

- name: Cache cargo build
  uses: actions/cache@v4
  with:
    path: target
    key: ${{ runner.os }}-cargo-build-target-${{ hashFiles('**/Cargo.lock') }}
```

## no_std環境でのCI設定

### 完全なワークフロー例
```yaml
name: no_std compatibility check
runs-on: ubuntu-latest

steps:
- uses: actions/checkout@v4

- name: Install Rust
  uses: dtolnay/rust-toolchain@stable
  # rust-toolchain.tomlの設定が自動的に使用される

- name: Add wasm32 target
  run: rustup target add wasm32-unknown-unknown

- name: Build with build-std
  run: |
    cd your_no_std_crate
    cargo build --target wasm32-unknown-unknown -Z build-std=core,alloc --verbose
```

### 重要なポイント

#### 1. `-Z build-std`フラグ
```bash
cargo build --target wasm32-unknown-unknown -Z build-std=core,alloc
```
- `core`と`alloc`をソースからビルド
- nightlyツールチェーンが必要
- `rust-src`コンポーネントが必要

#### 2. 必要なコンポーネント
```toml
[toolchain]
components = ["rust-src"]  # build-stdに必要
```

#### 3. ターゲットの指定
```bash
--target wasm32-unknown-unknown
```
- no_std環境でよく使われるターゲット
- OSに依存しないバイナリを生成

## トラブルシューティング

### 問題1: ツールチェーンバージョンの不一致
**症状**: ローカルでは動作するがCIで失敗する

**解決策**:
1. rust-toolchain.tomlを確認
2. GitHub Actionsでrust-toolchain.tomlが使用されているか確認
3. 明示的なツールチェーン指定を削除

### 問題2: ターゲットが見つからない
**症状**: `can't find crate for core`エラー

**解決策**:
```yaml
- run: rustup target add wasm32-unknown-unknown
```

### 問題3: rust-srcコンポーネントがない
**症状**: `-Z build-std`使用時のエラー

**解決策**:
```toml
[toolchain]
components = ["rustfmt", "rust-src"]
```

### 問題4: nightlyが必要な機能
**症状**: `requires nightly`エラー

**確認事項**:
- rust-toolchain.tomlでnightlyが指定されているか
- CIがrust-toolchain.tomlを無視していないか

## ベストプラクティス

### 1. rust-toolchain.tomlを中心に構成
```yaml
# 悪い例：ツールチェーンを明示的に指定
- uses: dtolnay/rust-toolchain@nightly
  with:
    toolchain: nightly-2024-01-01

# 良い例：rust-toolchain.tomlを使用
- uses: dtolnay/rust-toolchain@stable
```

### 2. 環境固有の設定は最小限に
- 共通設定はrust-toolchain.tomlに
- CI固有の設定（キャッシュなど）のみワークフローに

### 3. エラーメッセージを理解する
- `can't find crate`：ターゲットまたはコンポーネントの不足
- `requires nightly`：安定版では使えない機能
- `can't find attribute`：言語アイテムの不足

### 4. 段階的なビルド確認
```yaml
# 1. 通常のビルド
- run: cargo build --all

# 2. no_stdクレートのビルド
- run: |
    cd no_std_crate
    cargo build

# 3. 特定ターゲット向けビルド
- run: |
    cd no_std_crate
    cargo build --target wasm32-unknown-unknown -Z build-std=core,alloc
```

## まとめ

no_std環境でのCI構築は、通常のRustプロジェクトより複雑ですが、以下のポイントを押さえれば成功します：

1. **rust-toolchain.tomlを正しく設定**し、単一の真実の源として使用
2. **必要なコンポーネント**（rust-src）を含める
3. **ターゲットの追加**を忘れない
4. **-Z build-std**フラグで必要なクレートをビルド

特に書籍や教材のプロジェクトでは、指定されたツールチェーンバージョンを厳守することが重要です。勝手にバージョンを変更すると、予期しない動作やエラーの原因となります。