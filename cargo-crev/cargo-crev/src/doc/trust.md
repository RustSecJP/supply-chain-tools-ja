{% raw %}
# 信頼と信頼の網 (Web of Trust) {#trust-and-web-of-trust}

このドキュメントの目的は、`crev`（と `cargo-crev`）での信頼をユーザに
理解してもらうことです。

## 信頼の網 (Web of Trust) {#web-of-trust}

### 信頼プルーフ {#trust-proofs}

どのアイデンティティも、*信頼プルーフ* を生成して署名し、別のアイデンティティへの
直接の信頼を表明できます。

例。

``` text
-----BEGIN CREV TRUST -----
version: -1
date: "2019-04-28T22:05:05.147481998-07:00"
from:
  id-type: crev
  id: FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE
  url: "https://github.com/dpc/crev-proofs"
ids:
  - id-type: crev
    id: YWfa4SGgcW87fIT88uCkkrsRgIbWiGOOYmBbA1AtnKA
    url: "https://github.com/oherrala/crev-proofs"
trust: low
-----BEGIN CREV TRUST SIGNATURE-----
02BF0i1K0O7uR8T5UHzymqTo65P9R7JDuvfowZuHb3ubW8kd2-Fbl4jSv0n08ZdSU9P_E2YLWvEJrVQDYfjVCg
-----END CREV TRUST-----
```

アイデンティティ `FYlr8YoYGVvDwHQxqEIs89reKKDy-oWisoO0qXXEfHE` は、アイデンティティ
`YWfa4SGgcW87fIT88uCkkrsRgIbWiGOOYmBbA1AtnKA` を信頼しています。注目すべき点として、*信頼プルーフ* には
信頼レベルの情報（`trust` フィールド）が含まれます。

`cargo-crev` は、利用できるすべての *信頼プルーフ* から WoT を作り、
そこから個人の *信頼集合* (*trust set*) を計算します。

*信頼集合* を計算するとき、`crev` は与えられたルートアイデンティティから
グラフを再帰的にたどります。これにより *信頼* は推移的になります。

ルートアイデンティティはふつうユーザの今のアイデンティティですが、
`--for-id` 引数で任意に指定することもできます。

### *実効信頼レベル* {#effective-trust-level}

グラフをたどる間、`crev` は信頼された各アイデンティティの *実効信頼レベル*
(*effective trust level*) を記録しておきます。簡単に言うと、R がルートアイデンティティで、
R が X を低い *信頼レベル* で信頼し、X が Y を高い *信頼レベル* で信頼している場合、
R の Y に対する *実効信頼レベル* は低くなります。Y に対する *実効信頼レベル* は、
X に対する *実効信頼レベル* を超えられないからです。

より正確には、R の Y に対する *実効信頼レベル* は次に等しくなります:

- 次の最大値:
  - R の Y に対する直接の *信頼レベル*（あれば）、または
  - Y を信頼している、すでに信頼された任意のアイデンティティ Xi について、
    次のうち低いほうの値の最大値:
    - Xi の Y に対する直接の信頼レベル
    - R の Y に対する *実効信頼レベル*

言い換えると、R が Y に対してある *実効信頼* を持つには、R と Y の間に、
前のノードが次のノードを少なくともそのレベル以上で直接信頼している経路が、
少なくとも 1 つ存在しなければなりません。

それは、どの *実効信頼レベル* も、最も高い *実効信頼レベル* と同じ高さまでしか
なれないからです

### WoT の深さ {#depth-of-the-wot}

WoT を計算するために信頼グラフをたどる間、`cargo-crev` はルート ID からの距離を
記録しておきます。どこまで遠くまでたどるかの詳細は、次のコマンドラインオプションで
制御できます:

``` text
--depth <depth>
--high-cost <high_cost>
--medium-cost <medium_cost>
--low-cost <low_cost>
```

これにより、推移的な信頼を柔軟に制御できます。たとえば:

``` text
--high-cost 1 --medium-cost 1 --low-cost 1 --depth 1
```

とすると、実質的に `cargo-crev` は直接信頼しているアイデンティティだけを使うようになります。

### レビューの絞り込み {#filtering-reviews}

WoT の計算方法を制御できるだけでなく、使うパッケージレビューを
ほかの条件で絞り込むこともできます。

`--trust` オプションを使うと、指定した信頼レベル（以上）のアイデンティティが
作成したレビューだけでパッケージを検証できます。

次のオプションは:

``` text
--thoroughness <thoroughness>
--understanding <understanding>
```

レビューをその質で絞り込むのを制御します。

最後に:

``` text
--redundancy <redundancy>          Number of reviews required [default: 1]
```

は、各パッケージを信頼できるとみなすのに、信頼できるレビューがいくつ必要かを制御します。
{% endraw %}
