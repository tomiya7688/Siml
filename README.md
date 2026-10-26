# Siml

**Siml (Simple Language)** は、新しいOS、新しい命令セット、実験的なCPUなどへ持ち込みやすい、**コンパイラ実装の単純さ**を最優先するシステムプログラミング言語です。

Simlは、Cより高速な実行性能や巨大プロジェクトの最短ビルド時間を主目的にはしません。最重要条件は、実用的な最小Simlコンパイラが、同等のfreestanding / OS用途に使える最小Cコンパイラより単純であり、[Bitlang VM Assembly](https://github.com/tomiya7688/Bitlang-VM) で読める規模の実装として書けることです。Bitlang VM Assembly の低水準仕様は [Assam Core](https://github.com/tomiya7688/Assam) を基礎にしています。

Simlだけで、機械に直接触れる低水準部分からカーネル、ドライバ、高水準のユーザー空間まで、OS全体を書けることを目指します。

**正しいコードを生成する無最適化コンパイラは、完全なSimlコンパイラです。** 最適化は任意であり、最小コンパイラからできるだけ分離します。

## 最小実用版までの方針

1. GoまたはRustで参照コンパイラを作る。
2. Bitlang VM Assemblyで最小の初期立ち上げ用コンパイラを作る。
3. 低水準コンパイラに明示的な複雑さの上限を設ける。
4. その実装を、言語が本当に単純かを確かめる試験と、新しい環境へ移植する開発者向けの読める参照実装の両方に使う。
5. Siml自身でSimlコンパイラを書く。
6. SimlコンパイラがSimlコンパイラ自身をコンパイルできる状態にする。

## 文書

- [設計原則](docs/design-principles.md)
- [整数仕様](docs/integer-semantics.md)
- [ポインタ仕様](docs/pointer-semantics.md)
- [未決定事項](docs/open-decisions.md)
- [英語翻訳](docs/en/README.md)

## 文書の整形

リポジトリのルートでNode.js 24とnpmを使います。初回は `npm ci --ignore-scripts` で、固定されたPrettier 3.6.2を導入します。整形するファイルをコマンドの末尾に指定します。

```sh
npm run format -- docs/pointer-memory-model.md docs/open-decisions.md docs/en/open-decisions.md
npm run format:check -- docs/pointer-memory-model.md docs/open-decisions.md docs/en/open-decisions.md
npm run format:check:ci
```

`format:check:ci` は、CIで確認する文書とformatter設定の全リストを検査します。対象外の文書を変更する場合は、そのファイルを指定して `format` と `format:check` を実行してください。

状態: 初期設計中
