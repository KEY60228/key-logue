+++
author = "KEY"
title = "ブログをWordPressからHugoに移行した話"
image = "/img/move-to-hugo/OGP.png"
date = 2022-03-13
description = "さくらVPSとWordPressからNetlifyとHugoに移行してみた背景とか詰まった点とか"
tags = ["Hugo", "Netlify", "GitHubActions", "textlint", "reviewdog"]
categories = ["Tech", "Daily"]
archives = ["2022/03"]
draft = false
+++

さくらVPSとWordPress使ってたこのブログをNetlifyとHugoに移行してみた背景とか詰まった点とかをまとめてみる。

---

### とりあえず技術スタック

- Netlify
- Hugo (Theme: [Blonde](https://github.com/opera7133/Blonde))
- GitHub Actions ([textlint](https://textlint.github.io/), [reviewdog](https://docs.google.com/document/d/1mGOX19SSqRowWGbXieBfGPtLnM0BdTkIc9JelTiu6wA/edit))
- Google Analytics
- DISQUS
- Figma (ロゴ作成、OGP画像作成)

---

### 移行の背景

WordPressやめたいな〜〜と思ったのが一番大きかったかな。

WordPressやめたくなった理由はざっくり以下2つ。

#### WordPressのエディタで書くのに慣れなかった。

ブラウザ経由でWordPressのエディタで書くのにどうしても慣れなかった。

手元のエディタで書きたいな〜〜あわよくばGitHubに草生やしたいな〜〜って考えたのが一つ目の理由。

#### セキュリティが気になった。

アクセスログ見てるとwp-adminとかにガンガンアクセス飛んできてるし…

あまりこのブログの運用にリソース割いていなかった(しこれからもあまり割くつもりはない笑)ので、PHPのバージョンアップ対応とかWordPressのバージョンアップ対応とかしたくないなあって気持ちが2つ目の理由。

#### 他の理由

VPSからNetlifyに移したらサーバー代かからなくなるのでは？とも考えた。
 
…けど今回の移行と前後して別のプロダクトをVPSにホストすることにしてたから決定的理由にはならず。

あとは技術トレンドに乗りたかった、とかかな笑

誰かに「何か作って」って言われたときにさくっと作れるようになりたいな、って気持ちで始めたけど…

そんな機会なかなかなかった&WordPressを本腰入れて勉強する時間あるなら他の勉強したいってのも一因笑


---

### 技術選定の理由

#### SSG系に触れてみたかった

正直ブログ書くだけだしあまりレイアウトとか考えたくなかったし何かブログサービス使えば良いのでは？とも思ったけど…

やっぱりエンジニアたるもの触れたことのある技術が多いに越したことはない！と考えて最近よく聞くSSG系にリプレイスすることに。

あとブログサービスだと独自ドメイン使えない(？)(あまり調べてない)ってのも一つあるかな〜〜

#### あまりデザインとかレイアウトについて考えたくなかった

~~それこそブログサービス使えって話では~~

候補になるのかな、と考えたのはHugo, Gatsby, Next, Nuxtあたり。

プライベートでReact触ったりはしてたからGatsbyとかNextはアリなのかなあ、最近よく聞くしなあ、って思ったけど…

デザインとかレイアウトに対する自由度が高(そう)すぎてちょっと心理的障壁を感じたというのが正直なところ。

一方でHugoはテンプレートも豊富そうだしマークダウンですぐに書き始められそう…って思ったのが結構大きかった。

「Go言語で書かれてるし何かあったら深くまで読み込んでいけそう」とも一瞬思ったけど、「WordPressでPHPほとんど読み書きしてねえや」ってなって我に返った。

---

### 詰まった点

#### テーマ固有の要因

(先にも書いたけど)このブログでは[Blonde](https://github.com/opera7133/Blonde)っていうテーマを使わせてもらってて、submodule cloneした後特に何も触ってないはずなのに何故かタイトル下の「日付」と「タグ情報」が出なくて詰まった…

ドキュメントを何度読み返してもおかしなことやってないはずだし、Blonde使ってる他の人のリポジトリを覗きに行っても特に追加作業は発生していない様子…何故…

ともう他のテーマにしようかな、と諦めかけてたところ…

[issue](https://github.com/opera7133/Blonde/issues/15)上がってた！！

Blondeが想定している投稿ディレクトリは「posts/」じゃなくて「post/」

HugoのQuick Startでは「posts/」だったからてっきりHugoのお作法が「posts/」なのかと思ってたけど、そういうわけではなかった、って話。

#### textlintのルールむずい問題

せっかくだからCIで文章校正出来たら面白いのでは？と考えてtextlintとreviewdogをGitHub Acitonsで回すことにしたけど…

textlintのルール色々ありすぎ問題。(詰まったってわけじゃないけど…メモ的に残しておく)

色々試して、最終的に今使っているルールは以下の通り。

- [textlint-rule-ja-no-orthographic-variants](https://github.com/textlint-ja/textlint-rule-ja-no-orthographic-variants) 表記揺らぎを校正してくれる。
- [textlint-rule-ja-no-redundant-expression](https://github.com/textlint-ja/textlint-rule-ja-no-redundant-expression) 冗長な表現を校正してくれる。
- [textlint-rule-no-dropping-the-ra](https://github.com/textlint-ja/textlint-rule-no-dropping-the-ra) ら抜き言葉を校正してくれる。
- [textlint-ja/textlint-rule-no-dropping-i](https://github.com/textlint-ja/textlint-rule-no-dropping-i) い抜き言葉を校正してくれる。

試したけど使わなかったルールは以下の通り。

- [textlint-rule-ja-joyo-or-jinmeiyo-kanji](https://github.com/textlint-ja/textlint-rule-ja-joyo-or-jinmeiyo-kanji)

汎用漢字以外の文字を使うと校正してくれる。

『脆弱性』(ぜい弱性)とか『叩く』(たたく)とか『繋がる』(つながる)とかダメだったのはちょっと…

- [textlint-rule-no-doubled-joshi](https://github.com/textlint-ja/textlint-rule-no-doubled-joshi)

1つの文章に2つ以上同じ助詞があると教えてくれる。

自分の文体には合わなかったな…

- [textlint-rule-preset-ja-spacing](https://github.com/textlint-ja/textlint-rule-preset-ja-spacing)

スペースの使い方を校正してくれる。

普段使いにはまあ耐えられるくらいだったけど、Hugoのfront matterもエラー出しちゃうの辛かったな。

- [textlint-rule-preset-ja-technical-writing](https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing)

語尾に「。」があるか等チェックしてくれる。

「…」終わりとか「？」が使えないのが辛かった。

- [textlint-rule-spellcheck-tech-word](https://github.com/azu/textlint-rule-spellcheck-tech-word)

技術的なワードのスペルチェックとか行ってくれるのと、基本的な文章構造のチェック。

ありっちゃありだったけど…？？とか！！とか使いまくってる既存記事の書き直しが辛いな、ということで不採用。

(あとよくみたら今はDeprecatedになってた…)

#### NetlifyのDNS遅すぎ問題

Netlifyで独自ドメイン使う方法は以下4パターン(たぶん)

- ドメイン取得したドメイン管理会社のDNS(あるいは他のDNS)のALIASレコードでNetlifyのアプリケーションドメインを見る
- (略)DNSのCNAMEレコードでNetlifyのアプリケーションドメインを見る
- (略)DNSのAレコードでNetlifyのロードバランサーを見る(非推奨)
- NetlifyのDNSを使う

このうち2つ目のCNAMEレコードを使うにはサブドメインの設定が必要なためapexドメインを使うことを考えると3パターン。

そんで1つ目のALIASレコードは自分がドメイン取得したお名前ドットコムでは設定できなかった。

3つ目は非推奨…ってことで必然的に4つ目のNetlifyのDNSを使うことに。

で、いざ設定してデプロイしてみると…とにかく遅い。

初回アクセスが全然ページ表示されない。

Devツールで見てみるとDNS Lookupに2sec.くらいかかってる。

調べてみるとNetlifyは(無料プランの場合)日本のエッジノードが使えない模様。

日本でCDNを活用するには有料プランにしないといけない…

そうなってくると対策としては

- 有料プランに移行する
- ALIAS使える日本のDNS噛ます
- 日本にエッジロケーションのあるCDN噛ます
- 諦めてwww.key-logue.comで運用する

のどれかか…

と思ったけど、よく見てみると先述の3つ目の非推奨の方法、「CDNの恩恵を受けられないから非推奨」とのこと。

そもそも日本のCDNの恩恵受けられないなら別にこの方法でも変わらないのでは？と思って試してみたら早くなった。

(ホントにあってるんか？)

---

### 今後の課題

#### SEO的な問題

WordPress時代のURLから変っちゃってるからSEO的によろしくなさそう。(というか絶対良くない)

実際「[【SPA】CORSと認証とCSRF対策と](https://key-logue.com/post/spa-cors-auth-csrf1/)」は結構良いポジションにいて継続的にアクセスされてたけど、移行してから検索順位ガツンと落ちてる。

ちゃんとリダイレクト処理噛ませてあげないと…だけどどうやってやるんだろ。

---

### 末筆

まとまりのない記事になっちゃったから今後タイミング見て加筆修正する(かもしれないししないかもしれない)

Figmaでロゴ作ったりOGP画像作ったりもしたけど盛り込むの忘れてた。

まあとにもかくにもまずは継続的に記事を書くことを意識していきたい次第。笑

<script async src="//iframely.net/embed.js" charset="utf-8"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
