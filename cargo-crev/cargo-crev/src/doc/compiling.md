{% raw %}
# ソースからビルドする {#building-from-source}

## 最新の Rust が必要 {#latest-rust-required}

`cargo-crev` には最新の安定版の Rust が必要です。更新の遅い Linux ディストリビューションの
Rust パッケージをインストールしている場合、おそらく古すぎて動きません。
コンパイルエラーや unstable なフィーチャについての警告が出る場合は、
*Rust のバージョンが古すぎる* ということです。次を実行してください:

``` bash
rustup update
rustup default stable
```

`rustup` がない場合は、入っている Rust や Cargo のパッケージをアンインストールして、
[`rustup` で Rust をインストールしてください](https://rustup.rs/)。

## OpenSSL への依存 {#openssl-dependency}

今のところ `cargo-crev` をコンパイルするには、Rust 以外の依存が 1 つ必要です。
TLS のサポートに OpenSSL が必要だからです。

OpenSSL はよく使われていてすぐに手に入りますが、利用できるすべての OS での
インストール方法を網羅するのは事実上不可能です。以下にいくつか例を挙げます。
お使いの Unix 系 OS にも、対応するコマンドや似た名前のパッケージがあるはずです。

問題があれば、遠慮なく助けを求めてください。

### Debian と Ubuntu {#debian-and-ubuntu}

Debian と、Ubuntu のような Debian ベースのディストリビューションでは、次で動くはずです:

``` bash
sudo apt-get install openssl libssl-dev
```

### Arch Linux {#arch-linux}

Arch と、Manjaro のような Arch ベースのディストリビューションでは、最新の
OpenSSL がインストールされていることを確認してください:

``` bash
sudo pacman -Syu openssl
```

### RedHat {#redhat}

RedHat と、その派生の Fedora や CentOS では、次で動くはずです:

``` bash
sudo yum install openssl openssl-devel
```

### SuSE {#suse}

SuSE Linux では、次で動くはずです:

``` bash
sudo zypper install openssl libopenssl-devel
```

## コンパイルする {#compiling}

最新の `cargo-crev` のリリースをコンパイルしてインストールするには、`cargo` を使います:

``` bash
cargo install cargo-crev
```

main ブランチの最新の機能を試したい場合は、次を試してください:

``` bash
cargo install --git https://github.com/crev-dev/cargo-crev/ cargo-crev
```

### ローカルの git チェックアウトからコンパイルする {#compiling-from-a-local-git-checkout}

``` bash
cargo build --release -p cargo-crev
```

これで実行可能形式 `target/release/cargo-crev` がビルドされます。

## サポート {#support}

コンパイルで困ったことがあれば、[gitter
チャネルで質問する](https://gitter.im/dpc/crev) か、[ビルド済みの
バイナリ](https://github.com/crev-dev/cargo-crev/releases) を使ってください。
{% endraw %}
