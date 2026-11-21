{% raw %}
# 入門ガイド {#getting-started-guide}

## 要約 {#tldr}

``` bash
# セットアップ
cargo install cargo-crev
cargo crev trust --level high https://github.com/dpc/crev-proofs
cargo crev repo fetch all

# 検証
cargo crev verify --show-all

# レビュー
cargo crev open $crate_name
cargo crev review $crate_name

# レビューを共有する
# これをフォークする: https://github.com/crev-dev/crev-proofs/fork
cargo crev id set-url https://github.com/$your_github_username/crev-proofs
cargo crev publish

# もっとレビューを手に入れる
cargo crev id query all
cargo crev trust # ここにほかの人の URL か Id を入れる

# 前回から変わった部分だけをレビューする
cargo crev crate diff $crate_name | less
cargo crev review --diff $previous_version -- $crate_name
```

## はじめに {#introduction}

[Crev](https://github.com/crev-dev/crev) は、共同でのコードレビューにもとづいて、
依存のセキュリティと信頼性を検証するシステムです。Crev のユーザは、
パッケージ/ライブラリ/クレートのソースコードをレビューし、その結果をほかの人と共有します。
そして Crev は、信頼の網 (web of trust) を使って信頼できる
[レビュー](https://web.crev.dev/rust-reviews/crates/) を選び、
プロジェクトの依存の評判を判断します。

Crev は
[言語に依存しません](https://github.com/crev-dev/crev/#implementations) が、
主な実装は Rust/[Cargo](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html)
のクレート向けの [`cargo
crev`](https://github.com/crev-dev/cargo-crev/tree/main/cargo-crev) です。

> このプロジェクトとドキュメントは開発途中です。足りないところや、
> 間違っているところ、古くなっているところがあれば知らせてください。[crev の gitter
> チャネルに参加](https://gitter.im/dpc/crev) して助けを求めるか、GitHub の issue を開いてください。
> このドキュメントの改善への協力は、どんなものでも大歓迎です。

## インストール {#installing}

`cargo-crev` は Rust で書かれたコマンドラインツールです。[Releases ページからビルド済みの
バイナリをダウンロード](https://github.com/crev-dev/cargo-crev/releases) するか、
ソースからインストールできます:

``` bash
cargo install cargo-crev
```

コンパイルで問題が起きたときは、詳しい情報とトラブルシューティングについて
[ビルド手順を確認してください](compiling.html)。

## 実行する {#running}

`git` がふつうローカルの git リポジトリの中で使われるのと同じように、
`cargo crev` は Rust の `cargo` プロジェクトの中で使うことを想定しています。
`cargo crev` を使う前に、カレントディレクトリを Rust のプロジェクトに移動してください。

## サブコマンドを使う {#using-subcommands}

``` text
$ cargo crev
cargo-crev 0.18.0

USAGE:
    cargo crev <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    config     Local configuration
    crate      Crate related operations (review, verify...)
    id         Id (own and of other users)
    proof      Find a proof in the proof repo
    repo       Proof Repository
    trust      Add a Trust proof by an Id or a URL
    goto       Shortcut for `crate goto`
    open       Shortcut for `crate open`
    publish    Shortcut for `repo publish`
    review     Shortcut for `crate review`
    update     Shortcut for `repo update`
    verify     Shortcut for `crate verify`
```

サブコマンドによって、`cargo-crev` が行う動作が決まります。`cargo-crev` の
サブコマンドのいくつかには、さらにもう一段のサブコマンドがあります。こうした入れ子の
サブコマンドのいくつかには、`crate verify` に対する `verify` のようなショートカットがあります。
各サブコマンドについてのヘルプを見るには、`-h` フラグを使ってください。

注意: `cargo-crev` のほとんどのサブコマンドは省略できます。たとえば `cargo crev
cr v` のようにします。

## 検証する {#verifying}

ユーザとして `cargo crev` を使うふつうの目的は、今のクレートのすべての依存が
信頼でき、深刻なバグや欠陥がないことを検証することです。

依存のリストと、それぞれの今の信頼性の状態は、`cargo crev crate verify`
コマンドで見られます。これは最も重要で、最もよく使われるサブコマンドの 1 つです。

見てみましょう:

``` text
$ cargo crev verify --show-all
status reviews issues owner      downloads    loc lpidx geiger flgs crate                version      latest_t
none     0   2  0   0  0  3   197K   5129K    179    84      0 CB__ wasm-bindgen-shared  0.2.70
none     0   1  0   0  0  1   497K   1603K    268    29      0 ____ bytesize             1.0.1
none     0   1  0   0  0  1  3459K   7936K    238    94      1 ____ subtle               1.0.0
none     0   0  0   0  0  3   120K    854K    167    50      0 ____ signature            1.3.0
none     2   2  0   0  0  1     9K      9K      3    15      0 ____ default              0.1.2
```

### 列 {#columns}

右側の `crate` と `version` は、ほかの列の値がどのクレート（のどのバージョン）
について計算され、表示されているかを示します。

`status` 列は、各クレートの検証状態を表示します。`pass` という値は、
十分な数の信頼できる仲間がそのクレートをレビューしたことを示します。
`none` はレビューが足りないこと、`flagged` や `dangerous` は問題の報告があるクレートを示します。
依存の検証は、`status` 列の値がすべて `pass` のときにだけ成功とみなされます。

`crev` を使い始めたばかりなら、あなたの Rust のプロジェクトにはおそらく 100 を超える
依存があり、そのどれも検証を通っていないでしょう。まさにそれが `crev` が作られた
理由です。あなたのソフトウェアは、インターネットの見知らぬ人たちが作った、
あなたが一度も見たことのないコードを含む 100 以上のライブラリを、暗黙のうちに信頼しています。
解決できない問題に思えるかもしれませんが、`crev` の目標は、それを実際に
できるようにすることです。

`cargo crev verify --help`:

- reviews - その特定のバージョンについてのレビューの数と、利用できるすべての
  バージョンについてのレビューの数（合計）
- issues - 報告された issue の数（信頼できる情報源から/すべて）
- owner
  - 非再帰モードの場合: crates.io でのオーナーの数（既知/すべて）
  - 再帰モードの場合:
    - crates.io でのオーナーの総数
    - 部分集合を除いたオーナーグループの総数
- downloads - crates.io での、その特定のバージョンとすべてのバージョンの
  ダウンロード数
- loc - Rust のコードの行数
- lpidx - 「left-pad」指数（コードの行数に対するダウンロード数の比）
- geiger - Geiger スコア: `unsafe` な行の数
- flgs - 特定の種類のパッケージを示すフラグ
  - CB - Custom Build（ビルド時に任意のコードを実行する）
  - UM - Unmaintained（メンテナンスされていない）クレート
- name - クレートの名前
- version - クレートのバージョン
- latest\_t - 信頼できる最新のバージョン

## ほかのユーザからレビューを取ってくる {#fetching-reviews-from-other-users}

レビューは、crev のユーザの公開 git リポジトリに保存されています。`cargo crev update`
や `cargo crev repo fetch trusted` は、既知のリポジトリを自動で更新します。
個別に取ってくることもできます。`crev` の作者のすべての *プルーフ* を
取ってきてみましょう:

``` text
> cargo crev repo fetch url https://github.com/dpc/crev-proofs
Fetching https://github.com/dpc/crev-proofs... OK
Found proofs from:
      70 FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE
```

このコマンドは、ある git ユーザの公開されている *プルーフリポジトリ* から `git fetch` を行い、
今後のためにローカルのキャッシュに保存します。*プルーフリポジトリ* は、
*プルーフ* を含むただの git リポジトリです。

## *信頼プルーフ* を作る {#building-trust-proofs}

レビューはあるのに、そのクレートが信頼されていないことがあります。これは、
あなたがレビューした人を信頼していないときに起きます。

ほとんどのプロジェクトでは、すべての依存を自分でレビューすることはできません。
誰かを信頼する必要があります。crev のユーザを信頼しているという事実は、*信頼プルーフ*
として公開で記録されます。これにより、信頼できるレビュアの公開ネットワークを作れます。

ユーザは CrevID で個別に信頼できます。これが最も安全な選択肢です。
`dpc` を信頼するには、次を実行します:

``` text
$ cargo crev id trust FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE
```

ID のロックを解除すると、*プルーフ* を作るためのテキストエディタが開きます:

``` text
# Trust for FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE https://github.com/dpc/crev-proofs
trust: medium
comment: ""
```

これを編集して、関係を調整できます。プルーフの編集は、`git commit` で
コミットメッセージを編集するのにならっています。ご覧のとおり、エディタの中で
[役に立つドキュメント](https://github.com/crev-dev/cargo-crev/blob/main/crev-lib/rc/doc/editing-trust.md)
を読めます。

## `CrevID` を作る {#creating-a-crevid}

あなた自身もレビュアになれます。ほかの人があなたのレビューを使えるようになります。
*プルーフリポジトリ* にする公開 git リポジトリが必要です。
慣例として、リポジトリの名前は `crev-proofs` にします。

- GitHub のユーザは、[テンプレートをフォークする](https://github.com/crev-dev/crev-proofs/fork)
  だけで済みます（[GitLab でも同じ](https://gitlab.com/crev-dev/crev-proofs) です）。
- ほかのユーザは手作業で作れます。**注意**: `cargo-crev` は master
  ブランチがすでにあることを必要とするので、作るリポジトリには少なくとも
  1 つのコミットが必要です。

それから、次のように `cargo crev id new` を実行します:

``` text
$ cargo crev id new --url https://github.com/YOUR-USERNAME/crev-proofs
https://github.com/YOUR-USERNAME/crev-proofs cloned to /home/YOUR-USERNAME/.config/crev/proofs/Sp87YXeDKUyh4jImm23bCp1Gr-6eNkMoQogWbftNobQ
CrevID will be protected by a passphrase.
There's no way to recover your CrevID if you forget your passphrase.
Enter new passphrase:
```

このコマンドは、アイデンティティを暗号化するよう求め、バックアップ用に暗号化された
データを出力します。そのデータをコピーして、確実な場所に保管してください。

複数の ID を生成して使うこともできますが、ふつうは 1 つで十分です。今の
`CrevID` は次のように確認できます:

``` text
$ cargo crev id current
2CxdPgo2cbKpAfaPmEjMXJnXa7pdQGBBeGsgXjBJHzA https://github.com/YOUR-USERNAME/crev-proofs
```

変更（レビュー、信頼プルーフ）をプッシュするには、次を実行します:

``` bash
cargo crev publish
```

## 推移的な実効信頼 {#transitive-effective-trust}

作業が済んで、プルーフを保存してエディタを閉じたら、信頼しているすべての ID を
問い合わせられるはずです。

``` text
$ cargo crev id query trusted
FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE medium https://github.com/dpc/crev-proofs
YWfa4SGgcW87fIT88uCkkrsRgIbWiGOOYmBbA1AtnKA low    https://github.com/oherrala/crev-proofs
2CxdPgo2cbKpAfaPmEjMXJnXa7pdQGBBeGsgXjBJHzA high   https://github.com/YOUR-USERNAME/crev-proofs
```

少し驚くかもしれません。あなたがたった今 *信頼プルーフ* に署名した
`FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE` だけでなく、ほかのユーザも
信頼しています。

それは、[ユーザ `dpc` がすでにユーザ `oherrala`
を信頼していた](https://github.com/dpc/crev-proofs/blob/2d250e26bed95927a76551c7969cd108ebb1946c/FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE/trust/2019-04.proof.crev#L1)
からです。
`crev` での信頼は推移的です。あなたがユーザ `b` を信頼し、ユーザ `b` がユーザ
`c` を信頼しているなら、あなたは暗黙のうちにユーザ `c` を信頼しています。
`crev` でのあなた個人の *信頼の網* (*Web of Trust*) とは、まさにこのことです。

疑い深い人には最初は怖く思えるかもしれませんが、怖がる必要はありません。

私たちはここで「不可能」を成し遂げようとしています。ほかの人の作業を再利用しなければ、
大したことはできません。そして、得られる助けはどんなものでも使うべきです。

それでも心配なら、`cargo crev` には実効信頼の計算を設定する方法がたくさんあることを
知っておいてください。*信頼の網* の深さや、必要な冗長性のレベルも制御できます。
また、`c` の推移的な実効信頼レベルは、常に `b` への直接の信頼レベル以下になります。

## コードをレビューする {#reviewing-code}

もう一度 `cargo crev crate verify` を試してください。

ある程度運が良ければ、今では少なくともいくつかの依存が検証を通っているはずです。

ですが、結局は誰かがレビューをしなければならず、少なくともときには、
あなた自身がしなければなりません。

`cargo crev crate verify` の出力にざっと目を通し、コードの行数（`loc`）が少ない
クレートを選んでください。最初のレビューは、小さく簡単なものから始めるのがよいでしょう。

これを書いている時点で、`cargo crev` にはクレートのソースコードをレビューする方法が
2 つあります:

- コマンドラインや Vim のようなテキストエディタが好きな人には
  `cargo crev crate goto` コマンド
- IDE のユーザには `cargo crev crate open`

### `cargo crev crate goto` を使ってコードをレビューする {#reviewing-code-using-cargo-crev-crate-goto}

`default` というクレートをレビューしたい場合は、次を実行します:

``` text
$ cargo crev crate goto default
Opening shell in: /home/YOUR-USERNAME/.cargo/registry/src/github.com-1ecc6299db9ec823/default-0.1.2
Use `exit` or Ctrl-D to return to the original project.
Use `review` and `flag` without any arguments to review this crate.
```

出力の説明のとおり、`cargo crev crate goto` は新しいシェルを開きます。
そのカレントディレクトリは、`cargo` 自体が保存しているクレートのソースコードの
コピーに設定されています。

あとは `Vim` やほかのコマンド、テキストエディタを自由に使って、
クレートの中身を調べてください。最初は `tree -alh` や `ls` がよく使われ、
そのあとに `vi <path_to_rs_file>` が続きます。

[`cargo tree`](https://crates.io/crates/cargo-tree)（`cargo 1.44.0` 以降は
`cargo` の一部です）、
[`cargo-audit`](https://crates.io/crates/cargo-audit)、
[`cargo-outdated`](https://crates.io/crates/cargo-outdated) を使うことも検討してください。

では、レビューを始めましょう\! 初めての体験かもしれませんが、これが
`crev` の核心です。誰も実際にコードをレビューしなければ、信頼は築けません。
徹底的にやるよう心がけてください。ですが同時に、無理をしすぎたり、
怖がってまったくレビューしなくなったりはしないでください。

実際のレビューが済んだら、いよいよ *レビュープルーフ* を作って署名します。

`cargo crev crate review`（レビューの結果が否定的だった場合は `cargo crev flag`）を
呼び出すか、一時的なレビュー用シェルを抜けて `cargo
crev review <cratename>` を使います。

### `cargo crev open` を使ってコードをレビューする {#reviewing-code-using-cargo-crev-open}

IDE のユーザなら、`crev` にクレートのソースコードを好きな IDE で
開かせることができます。

例。VSCode のユーザは次を実行できます:

``` text
$ cargo crev open <crate> --cmd "code --wait -n" --cmd-save
```

`--cmd-save` を付けると、`crev` は `--cmd` パラメータを今後のために覚えておくので、
毎回くり返す必要がなくなります。IDE ごとに使うべき `--cmd` は異なることがあります。
わからないときは `crev` の gitter チャネルで助けを求めてください。
コマンドはあとで `cargo crev config edit` で変更できます。

### レビューを作成する {#creating-a-review}

コードをレビューしたら、*レビュープルーフ* を作成します:

``` bash
cargo crev crate review <cratename>
```

*信頼プルーフ* の編集と同じように、レビューを作成するときは *レビュープルーフ* の
ドキュメントを編集する必要があります。

``` text
# Package Review of default 0.1.2
review:
  thoroughness: low
  understanding: medium
  rating: positive
comment: ""


# # Creating Package Review Proof
#
# A Package Review Proof records results of your review of a version/release
# of a software package.
#
# ## Responsibility
#
# It is important that your review is truthful. At very least, make sure
# to adjust the `thoroughness` and `understanding` correctly.
#
# Other users might use information you provide, to judge software quality
# and trustworthiness.
#
# Your Proofs are cryptographically signed and will circulate in the ecosystem.
# While there is no explicit or implicitly legal responsibiltity attached to
# using `crev` system, other people will most probably use it to judge you,
# your other work, etc.
#
#
# ## Data fields
#
(...)
```

ここでも、役に立つコメント部分が *レビュープルーフ* の基本的な指針を説明しています。
[こちら](https://github.com/crev-dev/cargo-crev/blob/main/crev-lib/rc/doc/editing-package-review.md)
で読めます。

いちばん大事なのは、とにかく正直であることです。

作業を終えて *プルーフ* を保存する前に、[実際に署名された既存の
*レビュープルーフ*](https://github.com/dpc/crev-proofs/blob/2d250e26bed95927a76551c7969cd108ebb1946c/FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE/reviews/2018-12-packages-Ua7DxQ.proof.crev#L84)
を見てみましょう

``` text
-----BEGIN CREV PACKAGE REVIEW-----
version: -1
date: "2018-12-19T22:00:24.644210896-08:00"
from:
  id-type: crev
  id: FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE
  url: "https://github.com/dpc/crev-proofs"
package:
  source: "https://crates.io"
  name: either
  version: 1.5.0
  digest: uBbgCVotv_8z4SEOjremFmvMG4JPhUROC19OLjPPLNE
review:
  thoroughness: medium
  understanding: high
  rating: strong
comment: "Simple `Either` type."
-----BEGIN CREV PACKAGE REVIEW SIGNATURE-----
IBPz20fpI6x3nWJJ1pRsHqGVq3b6yQxyYppIlVPUEZIL3h9AYrV-u7UJMPu5sqCWski91mX8qOE5D3_2bgksDQ
-----END CREV PACKAGE REVIEW-----
```

もう気づいたかもしれませんが、あなたが編集しているドキュメントは完全な
*レビュープルーフ* ではありません。多くの詳細は `cargo
crev` が自動で埋めます。

`crev` のプルーフは Yaml のドキュメントで、GPG に似た区切りで囲まれ、
`cargo crev id new` のときに生成された秘密鍵で署名されています。

Yaml はよく使われるシリアライズ形式です。読みやすい形式です。また、
将来ドキュメントの形式を拡張しやすくもなります。

ドキュメントを保存して、エディタを閉じましょう。

これで、`cargo crev repo query
review <cratename>` の出力にあなたのプルーフが表示されるはずです:

``` text
$ cargo crev repo query review default
version: -1
date: "2019-06-19T23:32:13.683894969-07:00"
from:
  id-type: crev
  id: 2CxdPgo2cbKpAfaPmEjMXJnXa7pdQGBBeGsgXjBJHzA
  url: "https://github.com/YOUR-USERNAME/crev-proofs"
package:
  source: "https://crates.io"
  name: default
  version: 0.1.2
  revision: 583039a6a4233b6aa64dcba7a23f5ae4419a9a72
  digest: YuxzyXhCHZYMi4__Hj_hCzkQyxRLrZjDqL8usLqA4QY
review:
  thoroughness: low
  understanding: medium
  rating: positive
```

おめでとうございます\!

## *プルーフ* を公開する {#publishing-your-proofs}

*プルーフ* を作るたびに、`crev` はそれを、今の `CrevID` に結びついた
*プルーフリポジトリ* のローカルコピーに記録します。

このリポジトリには `cargo crev repo git` コマンドでアクセスできます。

``` text
$ cargo crev repo git log
commit a308421882822bd2256574b6e966a114dd4bfc6e (HEAD -> master)
Author: You <your_email@example.org>
Date:   Wed Jun 19 23:44:20 2019 -0700

    Add review for default v0.1.2
(...)
```

準備ができたら、`cargo crev repo publish` で最近の *プルーフ* を
公開リポジトリにプッシュできます。

これであなたの作業は公開されました。残るのは、ほかの人がそれを見つけられるように
することだけです。誰かがあなたの `CrevId` に対する *信頼プルーフ* を作るまで（`trust:
none` の設定であっても）、あなたの *プルーフリポジトリ* は簡単には見つかりません。

ブログ記事を公開したり、ツイートしたり、[`crev` の gitter
チャネル](https://gitter.im/dpc/crev) にメッセージを送ったり、[crev の *プルーフ
リポジトリ* の公式のブートストラップ用 wiki ページの
リスト](https://github.com/crev-dev/cargo-crev/wiki/List-of-Proof-Repositories)
に追加したりして、ほかの人にあなたの `CrevID` を *WoT* に含めてもらうよう頼めます

こうした場所は、ほかの人の *プルーフリポジトリ* をもっと見つけるのにも使えます。

## このあとは {#follow-up}

この短いガイドは、使い始めるためだけのものです。

`cargo crev` にはすでにもっと多くの機能が実装されており、今後もさらに
追加されていきます。とくに:

- 1 つの `CrevId` を複数のコンピュータで共有するつもりなら、
  `export` と `import` コマンドを必ず試してください。
- 差分レビューが使えます。クレート全体をレビューする代わりに、
  すでに信頼されているバージョンと今のバージョンの差分をレビューできます
  （`diff` と `review --diff` コマンド）。
- セキュリティ上の欠陥や深刻な欠陥は `review --advisory` で報告でき、
  `verify` の出力の `issues` に表示されます。
{% endraw %}
