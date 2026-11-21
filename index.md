{% raw %}
# Rust のサプライチェーン対策ツールの文書（日本語訳） {#supply-chain-tools}

Rust の依存のサプライチェーンを守るためのツールの文書を、日本語に訳したものです。

## cargo-auditable {#cargo-auditable}

ビルドした実行ファイルに、使った依存の一覧を埋め込むツールです。あとから、その実行ファイルに脆弱性のある crate が入っていないかを確かめられます。

- [cargo-auditable](cargo-auditable/)
- [埋め込んだデータのパース](cargo-auditable/PARSING.html)

## cargo-crev {#cargo-crev}

crate のコードレビューを書いて共有し、信頼できる人のレビューを「信頼の網（Web of Trust）」でたどって使うツールです。

- [cargo-crev](cargo-crev/)
- [入門ガイド](cargo-crev/cargo-crev/src/doc/getting_started.html)
- [信頼と信頼の網](cargo-crev/cargo-crev/src/doc/trust.html)

ほかの供給網対策のツールの訳: [cargo-vet](https://rustsecjp.github.io/cargo-vet-ja/)、[cargo-deny](https://rustsecjp.github.io/cargo-deny-ja/)
{% endraw %}
