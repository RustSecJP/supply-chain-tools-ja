{% raw %}
[`cargo auditable`](https://crates.io/crates/cargo-auditable) がバイナリに埋め込んだ依存ツリーを取り出す、高レベルのクレートです。

選択に応じて、JSON 文字列か Rust のデータ構造にデシリアライズします。

### 特徴 {#features}

 - バイナリのパースは、悪意のある入力への耐性を一から考えて設計されています。
 - すべての依存も含めて 100% メモリ安全な Rust です。（`serde_json` とその依存には `unsafe` が多少ありますが、それはシリアライズの部分だけで、ここでは使われません。）
 - クロスプラットフォームで移植性があり、クロスコンパイルも簡単です。[`std` のある任意の Rust ターゲット](https://doc.rust-lang.org/stable/rustc/platform-support.html) で動きます。
 - 実行中のプラットフォームだけでなく、サポートしているどのプラットフォームのバイナリもパースできます。
 - [OOM](https://en.wikipedia.org/wiki/Out_of_memory) や [zip 爆弾](https://en.wikipedia.org/wiki/Zip_bomb) から守るために、入力と出力の両方にサイズの上限を設定できます。

### 使い方 {#usage}

```rust
// デフォルトの上限を使う: 入力ファイルのサイズは 1GiB、監査データのサイズは 8MiB
let info = audit_info_from_file(&PathBuf::from("path/to/file"), Default::default())?;
```
`Read` のインスタンスや `&[u8]` からデータを読み込む関数も用意されています。
[ドキュメント](https://docs.rs/auditable-info) を見てください。

### 代わりの選択肢 {#alternatives}

[`rust-audit-info`](https://crates.io/crates/rust-audit-info) は、このクレートのコマンドラインインタフェースです。

このクレートが提供するものより低レベルのインタフェースが必要な場合は、
[`auditable-extract`](http://docs.rs/auditable-extract/) クレートと
[`auditable-serde`](http://docs.rs/auditable-serde/) クレートを使ってください。
{% endraw %}
