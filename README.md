# between-spec

このリポジトリは、Between Specifications Layer BSL の正本を保持するためのリポジトリである。

BSL は、Between Core が定義する意味構造と境界条件を、機械可読な識別子、スキーマ、操作語彙、判定単位へ落とし込むための仕様層である。Core が原理と制約を定義し、BSL はそれを実装と運用の手前で扱える形へ具体化する。

本リポジトリの正本は、`ja/bsl/` 配下の章別ファイル群および `ja/v0.1/annex/` 配下の annex 文書である。`ja/dist/bsl-combined.md` は通読・配布のための結合版であり、編集対象ではない。

## Public Status

このリポジトリは、まず参照用の公開リポジトリとして公開している。

現時点では、Issue と Discussion は受け付けていない。
ドキュメントと参加導線の整備が完了するまでは、Pull Request はレビューせずに閉じる場合がある。

公開での対話窓口は、後日 `between-docs` で案内する。
セキュリティに関する連絡は `SECURITY.md` に従うこと。

<details>
<summary>English</summary>

This repository is published first as a reference repository.

At this stage, Issues and Discussions are closed.
Pull requests may be closed without review while the documentation and contribution path are still being prepared.

The public communication entry point will be opened later in `between-docs`.
For security matters, please follow `SECURITY.md`.
</details>

## 読み始める順番

まず `ja/README.md` を読む。
次に `ja/bsl/bsl-00-positioning-and-purpose.md` から `ja/bsl/bsl-09-architecture.md` までを順に読む。
用語の確認には `ja/bsl/bsl-glossary.md` を参照する。
補助 annex が必要な場合は `ja/v0.1/annex/` を参照する。

## Scope

本リポジトリが扱うのは、BSL の章別仕様、識別子体系、スキーマ、操作語彙、契約構造、annex 文書である。

## Non Scope

本リポジトリは、Between Core そのものの原理定義、導入用解説、チュートリアル、エッセイ、公開運用メモを直接の対象としない。Core の原理は `between-core` を参照し、解説資料は companion リポジトリで扱う。

## Related Repositories

`between-core` は、BSL が依拠する原理・正式定義の正本リポジトリである。
`between-docs` は、導入、解説、補助資料を扱う companion リポジトリである。
