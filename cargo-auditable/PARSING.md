{% raw %}
## `cargo auditable` が埋め込んだデータのパース {#parsing-the-data-embedded-by-cargo-auditable}

この形式は単に Zlib と JSON を使っているだけなので、パーサの実装は簡単なはずです。次は Python で書いた最低限のパーサです:

```python3
import lief, zlib, json
binary = lief.parse("/path/to/file")
audit_data_section = next(filter(lambda section: section.name == ".dep-v0", binary.sections))
json_string = zlib.decompress(audit_data_section.content)
audit_data = json.loads(json_string)
```
Linux では、[`rust-audit-info`](rust-audit-info/index.html) を使えない場合、シェルで Linux バイナリ用のパーサを間に合わせで組むことさえできます:
```bash
objcopy --dump-section .dep-v0=/dev/stdout $1 | pigz -zd -
```

### ステップ 0: パーサがすでにあるか確認する {#step-0-check-if-a-parser-already-exists}

次のパース用ライブラリがあります:

 - Rust 用の [`auditable-info`](https://docs.rs/auditable-info/)
 - Go 用の [`go-rustaudit`](https://github.com/microsoft/go-rustaudit)

また、どの言語からでもサブプロセスとして呼び出せる単体のバイナリ [`rust-audit-info`](rust-audit-info/index.html) も提供しています。バイナリの面倒な処理をすべて引き受けて、JSON を出力します。

### ステップ 1: バイナリから圧縮されたデータを取り出す {#step-1-obtain-the-compressed-data-from-the-binary}

使っている言語で推奨されている ELF/Mach-O/PE パーサを使って、実行可能形式から `.dep-v0` セクションを取り出します。Apple のプラットフォーム（Mach-O 形式）では、このセクションは `__DATA` セグメントにあります。ほかの形式にはセグメントという概念はありません。

### ステップ 2: データを展開する {#step-2-decompress-the-data}

データは [Zlib](https://en.wikipedia.org/wiki/Zlib) で圧縮されています。単に展開してください。

プロセスをメモリの枯渇から守りたい場合は、[zip 爆弾](https://en.wikipedia.org/wiki/Zip_bomb) を避けるために出力のサイズを制限してください。正当な監査データなら、8 MiB あれば十分すぎるはずです。

### ステップ 3: JSON をデシリアライズする {#step-3-deserialize-the-json}

展開したデータを JSON としてパースします。正しい形式の JSON は UTF-8 であることが保証されています。UTF-8 でないデータを拒否するのは、パーサの振る舞いとして妥当です。

JSON スキーマは [こちら](https://github.com/rust-secure-code/cargo-auditable/blob/master/cargo-auditable.schema.json) にあります。

### セキュリティ上の考慮事項 {#security-considerations}

#### 依存ツリーの再構築 {#reconstructing-the-dependency-tree}

ビルドに使われたクレートのバージョンを得るだけでなく、依存ツリーを再構築する必要がある場合は、先にデータを検証する必要があります。この形式は技術的には、次の不正な状態をエンコードできてしまいます:

1. ルートパッケージが 0 個
1. ルートパッケージが 2 個以上
1. 循環する依存

依存ツリーをたどる前に、依存グラフに循環がないこと（たとえば [トポロジカルソート](https://en.wikipedia.org/wiki/Topological_sorting) をして確かめます）と、`root: true` のパッケージが 1 つだけであることを確認してください。

（循環する依存をエンコードできない形式も試しましたが、扱いやすくはなりませんでした。同じ問題が起きて対処しなければならず、その場所が違うだけでした。また、圧縮にも向いていませんでした。）

#### バイナリのパース {#binary-parsing}

バイナリをパースするライブラリの多くは、セキュリティを考えて設計されておらず、悪意のある入力にさらされることを想定していませんでした。そのため、任意コード実行のために [簡単に悪用できます](https://lcamtuf.blogspot.com/2014/10/psa-dont-run-strings-on-untrusted-files.html)。とくにバイナリのパースは、メモリ安全性のバグの温床です。

使っている言語の ELF/PE/Mach-O パーサが古くて大きな C のかたまりなら、代わりに私たちの Rust ライブラリを使うことを検討してください。これは悪意のある入力への耐性を特に考えて設計されています。すべての依存も含めて 100% 安全な Rust で実装されているので、そうした問題の影響を受けません。

それには、[`rust-audit-info`](rust-audit-info/index.html) をサブプロセスとして呼び出すか、使っている言語のバインディング生成ツールを使って [`auditable-info`](https://docs.rs/auditable-info/) ライブラリクレートへのバインディングを書きます。「call Rust from $LANGUAGE」でググってみてください。
{% endraw %}
