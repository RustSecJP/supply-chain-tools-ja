{% raw %}
# resolverver: Cargo のリゾルババージョンを得る {#resolverver-get-the-cargo-resolver-version}

あるワークスペースで使われている [Cargo のリゾルババージョン](https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions)
を知ることは、Cargo とやりとりする一部のツールにとって重要です。
必要になればわかります。

### 使い方 {#usage}

リゾルババージョンはワークスペース全体に対するグローバルな性質なので、
個々のパッケージの Cargo.toml ではなく、**ワークスペース** の Cargo.toml を
読むことが重要です。

[`cargo_metadata`](https://crates.io/crates/cargo_metadata) クレートを使って見つける方法は次のとおりです:

```rust
use cargo_metadata;
use resolverver;

// ワークスペースの Cargo.toml を見つけて読み込む
let metadata = cargo_metadata::MetadataCommand::new().no_deps().exec().unwrap();
let toml = std::fs::read_to_string(metadata.workspace_root.join("Cargo.toml")).unwrap();

// 使われているリゾルババージョンを推定する
let resolver_version = resolverver::from_toml(&toml).unwrap();
println!("Resolver version in this workspace is: {resolver_version:?}");
```

### 注意点 {#caveats}

Cargo には [`resolver.incompatible-rust-versions`](https://doc.rust-lang.org/cargo/reference/config.html#resolverincompatible-rust-versions) という設定オプションがあり、
ほかのすべてが V2 リゾルバを使うべきだと示している場合でも、V3 リゾルバを有効にすることがあります。

V2 と V3 のリゾルバの違いは、一部の依存で選ばれるバージョンだけです。
Cargo.lock ファイルの生成を Cargo に任せていて、バージョンの解決を自分で行っていない限り、
この違いは問題になりません。

これがあなたのユースケースで問題になる場合は、[`cargo-config2`](https://crates.io/crates/cargo-config2)
を使って Cargo の設定を読み込み、解決できます。
{% endraw %}
