{% raw %}
<p align="center">
  <a href="https://github.com/crev-dev/cargo-crev/discussions">
    <img src="https://img.shields.io/badge/commmunity-discussion-blue?style=flat-square" alt="community discussion">
  </a>
  <a href="https://github.com/crev-dev/cargo-crev/actions/workflows/ci.yml">
      <img src="https://github.com/crev-dev/cargo-crev/workflows/ci/badge.svg" alt="Github Actions CI Build Status">
  </a>
  <a href="https://crates.io/crates/cargo-crev">
      <img src="https://img.shields.io/crates/v/cargo-crev.svg?style=flat-square" alt="crates.io">
  </a>
  <br>
  
</p>


<p align="center">
  <img src="https://i.imgflip.com/5b8fqd.jpg" alt="jesus, that's a lot of dependencies" />
  <br/>
  <a href="https://jakelikesonions.com/post/158707858999/the-future-more-of-the-present">画像クレジット</a>
</p>


# cargo-crev {#cargo-crev}

> cargo（Rust）パッケージマネージャのための、暗号学的に検証可能な **c**ode **rev**iew（コードレビュー）システムです。

## はじめに {#introduction}

[Crev](https://github.com/crev-dev/crev/) は、言語やエコシステムに依存しない、
分散型の **c**ode **rev**iew（コードレビュー）システムです。

`cargo-crev` は、Crev を `cargo` と統合されたコマンドラインツールとして実装したものです。
このツールは、Rust のユーザがパッケージの依存の品質と信頼性を評価するのを助けます。

## 特徴 {#features}

`cargo-crev` はすでに次のことができます:

- 信頼できないクレートやセキュリティ脆弱性について警告する
- 依存についての役に立つ指標を表示する
- 依存の肥大化を見つけるのを助ける
- 最も疑わしい依存をレビューして、その結果を公開できるようにする
- ほかのユーザが作成したレビューを使う
- あなた自身のコードの信頼性を高める
- 評判の良いほかのユーザとの信頼の網 (web of trust) を作り、使っているコードの検証に役立てる

ほかにもたくさんのことができ、今後さらに増えていきます。

## はじめよう {#getting-started}

静的バイナリは [リリースページ](https://github.com/crev-dev/cargo-crev/releases) から入手できます。

[`cargo-crev` - Getting Started
Guide](https://github.com/crev-dev/cargo-crev/blob/main/cargo-crev/src/doc/getting_started.md)
に従ってください（さらに多くのドキュメントが [docs.rs](https://docs.rs/cargo-crev) にあります）。

`cargo-crev` は開発途中ですが、いつでも使える状態のはずです。
助けを求めたり、情報を得たり、フィードバックを送ったりするには
[discussions](https://github.com/crev-dev/cargo-crev/discussions) を使ってください。ありがとうございます\!

## 知ってもらう {#raise-awareness}

この取り組みを支持してくださるなら、プロジェクトを広めるのを手伝っていただけるとありがたいです。
あなたの Rust のプロジェクトの README に、次の注記を入れることを検討してください:

    It is recommended to always use [cargo-crev](https://github.com/crev-dev/cargo-crev)
    to verify the trustworthiness of each of your dependencies, including this one.

ありがとうございます\!

## 変更履歴 {#changelog}

変更履歴はこちらにあります:
<https://github.com/crev-dev/cargo-crev/blob/main/cargo-crev/CHANGELOG.md>
{% endraw %}
