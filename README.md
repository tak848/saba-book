# saba-book

『[作って学ぶ]ブラウザのしくみ』（技術評論社）の学習用ブラウザ実装プロジェクト

## 概要

このリポジトリは、技術評論社から出版されている書籍『[作って学ぶ]ブラウザのしくみ』（ISBN: 978-4-297-14546-0）に沿って、Rust でブラウザを実装する学習プロジェクトです。

### 書籍情報

- **出版社**: 技術評論社
- **ISBN**: 978-4-297-14546-0
- **サポートページ**: https://gihyo.jp/book/2024/978-4-297-14546-0/support
- **参考実装**: https://github.com/d0iasm/sababook

## プロジェクト構成

```
saba-book/
├── saba_core/          # ブラウザのコアロジック (no_std)
│   └── src/
│       ├── lib.rs      # ライブラリのエントリポイント
│       └── url.rs      # URL解析実装 (第2章)
├── src/
│   └── main.rs         # メインバイナリ (Wasabi OS向け)
├── Cargo.toml          # ワークスペース設定
└── rust-toolchain.toml # Rustツールチェーン設定
```

## 必要な環境

- Rust nightly-2024-01-01（`rust-toolchain.toml`で自動設定）
- cargo

## セットアップ

```bash
# リポジトリのクローン
git clone https://github.com/tak848/saba-book.git
cd saba-book

# ビルド（rust-toolchain.tomlにより自動的に正しいバージョンが使用されます）
cargo build
```

## ビルドとテスト

```bash
# 全体のビルド
cargo build

# テストの実行
cargo test --all

# saba_coreのテスト
cargo test -p saba_core

# no_std環境向けビルド（wasm32）
cd saba_core
cargo build --target wasm32-unknown-unknown -Z build-std=core,alloc
```

## 実装状況

- [x] 第 1 章: 環境構築
- [x] 第 2 章: URL 解析
- [ ] 第 3 章: HTTP クライアント
- [ ] 第 4 章: HTML パーサと DOM 木
- [ ] 第 5 章: CSS パーサとスタイル計算
- [ ] 第 6 章: レイアウトとレンダリング
- [ ] 第 7 章: JavaScript サポート

## ドキュメント

- [CLAUDE.md](./CLAUDE.md) - AI 開発アシスタント向けのプロジェクトガイド
- [docs/](./docs/) - 学習メモと Rust の知見

## ライセンス

このプロジェクトは学習目的で作成されています。書籍のコードを参考にしていますが、独自に実装したものです。

## 参考資料

- [書籍サポートページ](https://gihyo.jp/book/2024/978-4-297-14546-0/support)
- [正誤表・補足情報](https://lowlayergirls.github.io/wasabi-help/)
- [公式参考実装](https://github.com/d0iasm/sababook)
