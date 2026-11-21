{% raw %}
## rust-audit-info {#rust-audit-info}

[`cargo auditable`](https://crates.io/crates/cargo-auditable) がバイナリに埋め込んだ依存ツリーを取り出すコマンドラインツールです。

プラットフォーム固有の形式（[ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)、[PE](https://en.wikipedia.org/wiki/Portable_Executable)、[Mach-O](https://en.wikipedia.org/wiki/Mach-O)）のパースを引き受けて、展開した JSON を出力します。

このツールは意図的に最小限にしてあり、脆弱性のスキャンはそれ自体では実装していません。ですが、自分で脆弱性スキャナを作るときには役に立ちます。コマンドラインツールではなく Rust のライブラリを探しているなら、[`auditable-info`](https://docs.rs/auditable-info/) を見てください。

### 特徴 {#features}

 - 実行中のプラットフォームだけでなく、サポートしているどのプラットフォームのバイナリもパースできます。
 - 外部への依存がない、約 400Kb の自己完結した実行可能形式にコンパイルされます。
 - バイナリのパースは、悪意のある入力への耐性を一から考えて設計されています。
 - すべての依存も含めて 100% メモリ安全な Rust です。依存ツリーのどこにもメモリ安全でないコードはありません。
 - クロスプラットフォームで移植性があり、クロスコンパイルも簡単です。[`std` のある任意の Rust ターゲット](https://doc.rust-lang.org/stable/rustc/platform-support.html) で動きます。
 - [OOM](https://en.wikipedia.org/wiki/Out_of_memory) や [zip 爆弾](https://en.wikipedia.org/wiki/Zip_bomb) から守るために、入力と出力の両方にサイズの上限を設定できます。

### 使い方 {#usage}

```bash
Usage: rust-audit-info FILE [INPUT_SIZE_LIMIT] [OUTPUT_SIZE_LIMIT]

The limits are specified in bytes. The default values are:

    INPUT_SIZE_LIMIT: 1073741824 (1 GiB)
    OUTPUT_SIZE_LIMIT: 67108864 (64 MiB)

```

RAM の使用量は最大で `INPUT_SIZE_LIMIT + OUTPUT_SIZE_LIMIT` に、最大 1MB のオーバーヘッドを加えたものになります。

標準入力から読み込む必要がある場合は、`FILE` として `/dev/stdin` を渡してください。

### 依存 {#dependencies}

```
$ cargo geiger

Metric output format: x/y
    x = unsafe code used by the build
    y = total unsafe code found in the crate

Symbols: 
    🔒  = No `unsafe` usage found, declares #![forbid(unsafe_code)]
    ❓  = No `unsafe` usage found, missing #![forbid(unsafe_code)]
    ☢️   = `unsafe` usage found

Functions  Expressions  Impls  Traits  Methods  Dependency

0/0        0/0          0/0    0/0     0/0      🔒 rust-audit-info 0.5.2
0/0        0/0          0/0    0/0     0/0      🔒 └── auditable-info 0.6.2
0/0        0/0          0/0    0/0     0/0      🔒     ├── auditable-extract 0.3.2
0/0        0/0          0/0    0/0     0/0      🔒     │   └── binfarce 0.2.1
0/0        0/0          0/0    0/0     0/0      🔒     └── miniz_oxide 0.6.2
0/0        0/0          0/0    0/0     0/0      🔒         └── adler 1.0.2

0/0        0/0          0/0    0/0     0/0
```
{% endraw %}
