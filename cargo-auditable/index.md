{% raw %}
## cargo-auditable {#cargo-auditable}

Rust の実行可能形式をビルドするのに使ったクレートの正確なバージョンがわかります。本番環境のバイナリを、既知のバグやセキュリティ脆弱性について大規模に監査できます。記録を手で管理する必要はありません。

これは、依存ツリーのデータを JSON 形式で、コンパイルされた実行可能形式の専用リンカセクションに埋め込むことで実現しています。

Linux、Windows、Mac OS を公式にサポートしています。v0.6.3 からは [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly) もサポートしています。そのほかの ELF ターゲットもすべて動くはずですが、CI ではテストしていません。

最終的な目標は、Cargo 自体がこの情報をバイナリにエンコードするようにすることです。Cargo の中に実装するための RFC があり、このプロジェクトはその道を開くものです: https://github.com/rust-lang/rfcs/pull/2801

## 使い方 {#usage}

```bash
# ツールをインストールする
cargo install cargo-auditable cargo-audit
# 依存のリストをバイナリに埋め込んでプロジェクトをビルドする
cargo auditable build --release
# バイナリの脆弱性をスキャンする
cargo audit bin target/release/your-project
```

`cargo auditable` はどの Cargo コマンドとも一緒に使えます。引数はすべてそのまま `cargo` に渡されます。

### nightly Rust の場合 {#on-nightly-rust}

nightly では、Cargo の [ネイティブな SBOM の前段階の機能](https://doc.rust-lang.org/cargo/reference/unstable.html#sbom) を使って、依存をより正確に記録できます:

```bash
CARGO_BUILD_SBOM=true cargo +nightly auditable build -Z sbom --release
```

[Cargo のバグ](https://github.com/rust-lang/cargo/issues/15695) のため、同じプロジェクトで `-Z sbom` なしの `cargo auditable` も使っていた場合は、先に `touch src/*` か `cargo clean` をする必要があるかもしれません。

### ほかのツールを通して使う {#through-other-tools}

`cargo` を直接呼び出しておらず、呼び出し方も変えられない場合は、`cargo auditable` を `cargo` のそのまま置き換えられる代替として使えます。詳しくは [こちら](https://github.com/rust-secure-code/cargo-auditable/blob/master/REPLACING_CARGO.md) を見てください。

### GitHub のリリース向け {#for-github-releases}

[`cargo dist`](https://github.com/axodotdev/cargo-dist) は、オプトインで `cargo auditable` をサポートしています。詳しくは [こちら](https://axodotdev.github.io/cargo-dist/book/supplychain-security/index.html) を見てください。

## 採用事例 {#adoption}

Microsoft は社内で `cargo auditable` を使っており、以前は [Go 用のデータ抽出ライブラリ](https://github.com/microsoft/go-rustaudit) をメンテナンスしていました。

複数の Linux ディストリビューションが、Rust のパッケージを `cargo auditable` でビルドしています: [Alpine Linux](https://www.alpinelinux.org/)、[NixOS](https://nixos.org/)、[openSUSE](https://www.opensuse.org/)、[Void Linux](https://voidlinux.org/)、[Chimera Linux](https://chimera-linux.org/)、[Wolfi OS](https://wolfi.dev) です。これらのリポジトリからパッケージをインストールしているなら、それを監査できます！

[Ubuntu 26.04](https://documentation.ubuntu.com/release-notes/26.04/) は [一部のパッケージで](https://documentation.ubuntu.com/release-notes/26.04/changes-since-previous-interim/#rust-cargo-auditable) これを使っており、ほかのパッケージも [オプトイン](https://documentation.ubuntu.com/project/contributors/language-specific/rust/cargo-auditable/) できます。[PPA](https://documentation.ubuntu.com/launchpad/user/reference/packaging/ppas/ppa/) でもオプトインできます。

[Chainguard](https://chainguard.dev/) は、[Rust のベースコンテナ](https://images.chainguard.dev/directory/image/rust/overview) に `cargo auditable` を含めています。常に `cargo auditable` を呼び出す `cargo` のラッパがデフォルトで入っているので、このコンテナでビルドした Rust アプリケーションはデフォルトで監査可能になります。

## FAQ {#faq}

### バイナリが肥大化しませんか？ {#doesnt-this-bloat-my-binary}

一言で言えば、しません。埋め込まれる依存のリストは、エントリが 400 以上ある大きな依存ツリーでも 4kB 未満です。これは通常、バイナリのサイズの 1/1000 から 1/10,000 程度にあたります。

### `cargo` で常に `cargo auditable` を使ってビルドするようにできますか？ {#can-i-make-cargo-always-build-with-cargo-auditable}

できます！たとえば Linux や macOS などでは、`.bashrc` に次を追加します:

```bash
alias cargo="cargo auditable"
```

bash 以外のシェルを使っている場合や、エイリアスを使えない場合は、[こちらを見てください。](https://github.com/rust-secure-code/cargo-auditable/blob/master/REPLACING_CARGO.md)

### このデータを利用するツールはありますか？ {#is-there-any-tooling-to-consume-this-data}

#### 脆弱性の報告 {#vulnerability-reporting}

* [cargo audit](https://crates.io/crates/cargo-audit) v0.17.3 以降は、バイナリの中のこのデータを検出し、脆弱性を報告できます。詳しくは [こちら](https://github.com/rustsec/rustsec/tree/main/cargo-audit#cargo-audit-bin-subcommand) を見てください。
* [trivy](https://github.com/aquasecurity/trivy) v0.31.0 以降は、バイナリの中のこのデータを検出し、脆弱性を報告します。最初から最後までの例は [v0.31.0 のリリースノート](https://github.com/aquasecurity/trivy/discussions/2716) を見てください。
* [grype](https://github.com/anchore/grype) v0.83.0 以降は、バイナリとコンテナイメージの中のこのデータを検出し、脆弱性を報告します。
* [osv-scanner](https://github.com/google/osv-scanner/) v2.0.1 以降は、コンテナイメージをスキャンするときに [このデータを読み取ります](https://github.com/google/osv-scalibr/pull/377)。

#### 依存のリストの復元 {#recovering-the-dependency-list}

* [syft](https://github.com/anchore/syft) v1.15.0 以降は、このデータを復元して、さまざまな形式に変換できます。それより古いバージョンでは `--catalogers all` という CLI オプションが必要です。
* [docker](https://docs.docker.com/build/metadata/attestations/sbom/) は、コンテナイメージに CycloneDX のドキュメントを埋め込めます。`docker buildx build --tag <namespace>/<image>:<version> --attest type=sbom --push .` でコンテナイメージをビルドし、`Dockerfile` の中で Rust のバイナリを `cargo auditable` でビルドすると、コンテナイメージに付く SBOM のアテステーションに Rust の依存が含まれます。これは [BuildKit Syft scanner](https://github.com/docker/buildkit-syft-scanner) によって実現されています。
* [blint](https://github.com/owasp-dep-scan/blint) v2.1.3 以降は、このデータを復元して CycloneDX として出力できます。
* [wasm-tools](https://github.com/bytecodealliance/wasm-tools) v1.227.0 以降は、WebAssembly からこのデータを復元できます。`wasm-tools metadata show` を試してください。
* [rust-audit-info](https://crates.io/crates/rust-audit-info) は、バイナリから依存のリストを復元して JSON で出力します。
* [auditable2cdx](https://crates.io/crates/auditable2cdx) は、バイナリから依存のリストを復元して CycloneDX で出力します。

### ほかの言語で書かれたツールでこのデータを読めますか？ {#can-i-read-this-data-using-a-tool-written-in-a-different-language}

読めます。このデータ形式は、別の実装との相互運用性を考えて設計されています。実際、パースするのに必要なのは [Python 5 行](PARSING.html) だけです。データのパースについてのドキュメントは [こちら](PARSING.html) を見てください。

それ以外にも、Syft はこのデータを読み取って、多くの形式に変換できます。`auditable2cdx` は、ほとんどのツールが理解できる CycloneDX に変換できます。この変換を使えば、変更できないツールにもこのデータを渡せます。

### データ形式は正確にはどうなっていますか？ {#what-is-the-data-format-exactly}

データ形式は [こちら](https://github.com/rust-secure-code/cargo-auditable/blob/master/cargo-auditable.schema.json) の JSON スキーマで記述されています。
JSON は Zlib で圧縮され、`.dep-v0` という名前のリンカセクションに置かれます。
パースについての詳しい情報は [こちら](PARSING.html) にあります。

### 組み込みプラットフォームではどうなりますか？ {#what-about-embedded-platforms}

監査用のデータはファームウェアイメージに埋め込まれますが、容量の限られたマイクロコントローラには書き込まれません。これにより、貴重な ROM を無駄にせずにイメージを監査可能に保てます。

### 再現可能なビルドに影響しますか？ {#does-this-impact-reproducible-builds}

このデータ形式は、再現可能なビルドを邪魔しないように特別に設計されています。タイムスタンプは含まれず、生成される JSON はソートされるので、コンパイルのたびに同じものになります。むしろ、あるバイナリのバージョンがすべてわかるようになるので、再現可能なビルドの *助け* になります。

### 機密情報が漏れませんか？ {#does-this-disclose-any-sensitive-information}

漏れません。URL とファイルパスはすべて伏せられますが、クレートの名前とバージョンはそのまま記録されます。現状でも、パニックのメッセージはこれらの情報をすべて、さらにそれ以上を明かしています。また、MIT をはじめとする多くのライセンスが求めているので、どのみち特定のオープンソースのクレートを使っていることを法的に開示する義務がある可能性が高いです。

### コンパイラのバージョンの記録はどうなりますか？ {#what-about-recording-the-compiler-version}

v1.73 以降では、コンパイラ自体が [それを埋め込みます](https://github.com/rust-lang/rust/pull/97550)。

それより古いバージョンでも、デバッグ情報にすでに入っています。Unix では `strings your_executable | grep 'rustc version'` を実行すると見られます。

### 静的リンクされた C ライブラリのバージョンの追跡はどうなりますか？ {#what-about-keeping-track-of-versions-of-statically-linked-c-libraries}

良い質問です。今のところ、それらが妥当な形で公開されているとは思いません。あればとても良い追加になりますが、最初のリリースには必須ではありません。後方互換性を保ったまま、あとで追加できます。[`-src` クレートの慣習](https://internals.rust-lang.org/t/statically-linked-c-c-libraries/17175?u=shnatsel) を採用すれば自然に実現でき、ほかの利点もあるので、おそらくそれが一番良い道でしょう。

### サプライチェーン攻撃を防げますか？ {#does-this-protect-against-supply-chain-attacks}

防げません。それには [`cargo-vet`](https://github.com/mozilla/cargo-vet) か [`cargo-crev`](https://github.com/crev-dev/cargo-crev) を使ってください。

[ソフトウェア部品表](https://en.wikipedia.org/wiki/Software_supply_chain)（SBOM）はサプライチェーン攻撃を防ぎません。攻撃が見つかったあとにその影響を評価することにさえ使えません。それなりの悪意あるライブラリなら、SBOM から自分自身を消すからです。これは Rust と Cargo だけでなく、ほぼすべての言語とビルドシステムに当てはまります。

サプライチェーン攻撃に対処するときに SBOM に頼らないでください！

### これを Cargo に取り込むのを妨げているものは何ですか？ {#what-is-blocking-uplifting-this-into-cargo}

[この機能を Cargo 自体に入れるための RFC](https://github.com/rust-lang/rfcs/pull/2801) は、[より基礎的な SBOM の RFC](https://github.com/rust-lang/rfcs/pull/3553) が済むまで、Cargo チームによって [延期](https://github.com/rust-lang/rfcs/pull/2801#issuecomment-2122880841) されました。

その RFC は現在実装されていて、[unstable なフィーチャ](https://doc.rust-lang.org/cargo/reference/unstable.html#sbom) として使えます。これにより、この機能を `cargo` 自体に入れるための RFC を再び提出する道が開かれました。
{% endraw %}
