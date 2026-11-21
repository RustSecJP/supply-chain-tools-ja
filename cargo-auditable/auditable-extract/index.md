{% raw %}
[`cargo auditable`](https://github.com/rust-secure-code/cargo-auditable) が実行可能形式に埋め込んだ依存ツリーの情報を取り出します。

このクレートは、プラットフォーム固有のバイナリ形式（[ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)、
[PE](https://en.wikipedia.org/wiki/Portable_Executable)、
[Mach-O](https://en.wikipedia.org/wiki/Mach-O)、[WASM](https://en.wikipedia.org/wiki/WebAssembly)）をパースして、圧縮された監査データを得ます。

ほかのバイナリをパースするクレートとは違って、悪意のある入力への耐性を特に考えて設計されています。
（すべての依存も含めて）100% 安全な Rust で書かれており、ヒープのメモリ確保を一切行いません。

## 使い方 {#usage}

**注意:** これはバイナリのパースだけを実装した低レベルのクレートです。直接使うべき場面はめったにありません。
おそらく、代わりに高レベルの [`auditable-info`](https://docs.rs/auditable-info) クレートを使うほうがよいでしょう。

次のコード片は、このクレートを使った抽出の流れ全体を示しています。安全な Rust で書かれた [`miniz_oxide`](http://docs.rs/miniz_oxide/) による展開と、
[`auditable-serde`](http://docs.rs/auditable-serde/) による任意の JSON のパースも含みます:

```rust,ignore
use std::io::{Read, BufReader};
use std::{error::Error, fs::File, str::FromStr};
!
fn main() -> Result<(), Box<dyn Error>> {
    // 入力を読み込む
    let f = File::open("target/release/hello-world")?;
    let mut f = BufReader::new(f);
    let mut input_binary = Vec::new();
    f.read_to_end(&mut input_binary)?;
    // 圧縮された監査データを取り出す
    let compressed_audit_data = auditable_extract::raw_auditable_data(&input_binary)?;
    // 好きな Zlib 実装で展開する。miniz_oxide がおすすめ
    use miniz_oxide::inflate::decompress_to_vec_zlib;
    let decompressed_data = decompress_to_vec_zlib(&compressed_audit_data)
        .map_err(|_| "Failed to decompress audit data")?;
    let decompressed_data = String::from_utf8(decompressed_data)?;
    println!("{}", decompressed_data);
    // 監査データを Rust のデータ構造にパースする
    let dependency_tree = auditable_serde::VersionInfo::from_str(&decompressed_data);
    Ok(())
}
```
{% endraw %}
