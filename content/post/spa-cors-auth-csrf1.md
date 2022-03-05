+++
author = "KEY"
title = "【SPA】CORSと認証とCSRF対策と"
image = "/img/spa-cors-auth-csrf1/SPA.jpeg"
date = 2020-11-26
description = "SPAにおけるCORS、認証、CSRF対策について調べてみた"
tags = ["SPA", "CORS", "CSRF"]
categories = ["Tech"]
archives = ["2020/11"]
draft = false
+++

今回のテーマはSPAにおけるCORSと認証とCSRF対策について。

---

### 前提条件

今回はあくまで技術について抽象的に書いていきたいと思うので、具体的な手法等についてはまた別の機会に。

とは言え現在個人で開発している条件をベースに書いているので、もしかしたらあまり汎用的ではないかも…？

とりあえずざっくりとした使用技術、構成は以下の通り。

- API: Laravel (6.18) / Nginx (1.19)
- クライアント: React (16.13) / webpack-dev-server (3.11)

ReactはLaravel上には乗せず独立したディレクトリにあり、開発環境上ではwebpack-dev-serverで確認するスタイル。

ポートは当然Nginxとwebpack-dev-serverで異なるので、クロスオリジンに該当する。

---

### 前提知識

#### CORS (Cross-Origin Resource Sharing) とは

「〜〜とは」系は公式とかMDN読むのが一番なのでまずは下記参照。笑

<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://developer.mozilla.org/ja/docs/Web/HTTP/CORS" data-iframely-url="//iframely.net/Z7fLZqT?card=small"></a>
    </div>
</div>

「オリジン」については以下参照。

<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://developer.mozilla.org/ja/docs/Glossary/Origin" data-iframely-url="//iframely.net/ENtld6q?card=small"></a>
    </div>
</div>
<br>
<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://web.dev/same-site-same-origin/" data-iframely-url="//iframely.net/0o04tM9"></a>
    </div>
</div>

ざっくり言ってしまえば、元々同一オリジンからしかリソースを読み込めなかったのを、ある条件下においては異なるオリジンからの読み込みも可能にした仕様のこと。

条件とは以下の通り。

- Simple Methodsである場合
- 該当するヘッダ以外を送信しようとしていない場合
- 該当するメディアタイプ以外をContent-typeに指定していない場合
- 上記3条件には当てはまらないが、プリフライトリクエストによってサーバーサイドに認められた場合

#### Cookieとは

(Cookieとはなんぞや、ってところは流石に省略)

<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://developer.mozilla.org/ja/docs/Web/HTTP/Cookies" data-iframely-url="//iframely.net/TbxNQPw?card=small"></a>
    </div>
</div>

CookieにはExpires、Max-Age、Domain etc.な属性があるわけだけど、今回関係してくるのは以下の3つ。

- Secure属性
- HttpOnly属性
- SameSite属性

簡単に言えば…

Secure属性の付与されたCookieはHTTPS通信でしか送受信されない。

HttpOnly属性の付与されたCookieはJSからアクセスが出来ない。

SameSite属性にはLax、Strict、Noneの3つの属性があり、リクエスト元によってそのCookieを送信するか否か決める。

…ってところかな…

ちなみに最近のブラウザでは「SameSite属性がNone」かつ「Secure属性が付与されていない」Cookieは自動的に受け付けないそう。

<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Set-Cookie/SameSite" data-iframely-url="//iframely.net/Uyf8YhV?card=small"></a>
    </div>
</div>
<br>
<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://web.dev/same-site-same-origin/" data-iframely-url="//iframely.net/0o04tM9"></a>
    </div>
</div>

#### Web Storageとは

Cookieとよく比較される、ブラウザにおける保存領域のこと。

<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://developer.mozilla.org/ja/docs/Web/API/Web_Storage_API" data-iframely-url="//iframely.net/rll6VAC?card=small"></a>
    </div>
</div>

Cookieと比べて、

- 保存できる容量が大きい
- 送受信のタイミングを実装者が選択できるため、通信時のパフォーマンスの向上を図れる

といったメリットがある一方、

- JSからのアクセスが容易なため、XSSに対する緩和策が成されていない[^1]

[^1]: HttpOnlyなCookieもXSS対策にはならない。あくまで被害を緩和するだけ。

というデメリットがある。

#### JWT (Json Web Token) とは

[JSON Web Token (JWT) -- OpenID Foundation Japan](https://openid-foundation-japan.github.io/draft-ietf-oauth-json-web-token-11.ja.html)

(正直勉強不足であまり理解できていない…)

要するに(認証)サーバーで発行するJSONベースのエンコードされたトークン、と認識しています…！

(下記Qiitaの記事が分かりやすかった)

<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://qiita.com/Naoto9282/items/8427918564400968bd2b" data-iframely-url="//iframely.net/mgMBvCk?card=small"></a>
    </div>
</div>

#### CSRF (Cross-Site Request Forgeries) とは

利用者の意図しないリクエストを偽造(forgeries)すること。

[クロスサイト・リクエスト・フォージェリ -- IPA](https://www.ipa.go.jp/security/vuln/websecurity-HTML-1_6.html)

CSRF脆弱性があると、爆破予告や犯行予告などの反社会的な投稿や、パスワードの変更など、ユーザーが意図しない投稿や編集を行われる可能性がある。

---

### 本題

#### SPAにおけるCORS

Laravel上でLaravel Mixを使う等の場合は考慮する必要はない。

一方、「ビルドしたJSをS3に置いてEC2でAPIサーバーを立てる」、「Nodeサーバーを立ててSSRをし、APIは別サーバーに立てる」等の場合には当然考慮する必要がある。

(イマドキのSPAは大体後者な気が…)

とは言えAPIサイドのCORS対策自体は大体のフレームワークでモジュールが準備されているはず。

(Laravelならfruitcake/laravel-cors、Djangoならdjango-cors-headers等)

クライアント側の実装は(後述するCSRF対策を意識しなければ)特に意識することはない(はず)

Laravelにおける具体的な設定方法等は下記記事を参考にさせて頂きました。

[CORSを許可する -- Larapet](https://larapet.hinaloe.net/2020/03/05/laravel-cors/)

(上記記事中に記載あるが、6.x系以前のLaravelでは自身でインストールする必要あり)

#### SPAにおける認証

SPAにおける主な認証パターンは大きく下記4パターンに分けられる。[^2]

[^2]: その他OAuthやSSOなど…？

1. サーバー: Sessionトークンを発行 -> クライアント: Cookieに保存
2. サーバー: Sessionトークンを発行 -> クライアント: Web Storageに保存
3. サーバー: JWTを発行 -> クライアント: Cookieに保存
4. サーバー: JWTを発行 -> クライアント: Web Storageに保存

1はこれまでのMPAで最も一般的な認証で、学習コスト、実装コストは軽いはず。

3も有効な手段のような気もするけど、1に勝るメリットは思いつかなかったな…

2と4は前述の通りJSからのアクセスが容易なため、悪意あるJSを埋め込まれると認証トークンが抜き出される可能性がある。

4の手法はRESTのステートレスの観点からか結構普及しているっぽい…？

XSS脆弱性対策はフレームワークで担保、あるいは認証の有効期限を短くして、リスク許容した上でWeb Storageを使う、みたいな意見もチラホラ見るような気がするけど…

正直今のところはCookieを使ってしまった方が安牌な気がした。

#### SPAにおけるCSRF対策

認証にCookieを使用している場合はCSRF対策を講じる必要がある。

SPAにおけるCSRF対策は以下のパターンが考えられる。

1. CSRFトークン
    1. 認証成功時に渡し、Redux/Vuex等で保持しておく
    2. 重要なリクエストの直前で渡し、直後の重要なリクエストで返す
2. プリフライトリクエストでオリジンチェック

1のCSRFトークンもこれまでのMPAでデファクトスタンダードだった手法。

ただSPAにおいてはトークン発行のタイミングが難しい気がする。

1-1は有効な気もするけど、リロードした時に毎回ユーザーに認証を求めることになるからUX的にはとてもイマイチ。

(Cookie使って自動ログインさせちゃうとCSRFトークンの意味がない…よね？)

1-2は2回API叩いたら突破できる気がするからそもそも意味ない気がする…

2は有効。

{{< tweet user="ockeghem" id="1254340420680560640" >}}

…しっかりとサーバー側で設定出来ている限り。笑

以前はオリジンを偽装出来てしまうFlash Playerの脆弱性があったみたいだけど、今は修正済みとのこと。

[mala/gist:8857629 -- GitHub](https://gist.github.com/mala/8857629)

現時点においてオリジンを偽装する手段はない(はずだ)から、カスタムリクエストヘッダをマストにして、プリフライトリクエストを送らせる仕様にすればCSRFは起き得ない(はず)

---

### 統括

だいぶ長くなってしまったけど…クロスオリジンなSPAにおいて、結論としては、

認証: サーバーでセッションIDを生成し、SameSite=None、Secure=true、HttpOnly=trueなCookieに保存させ、

CSRF対策: 重要なリクエストの際にはカスタムリクエストヘッダ等でプリフライトリクエストを送らせ、オリジンを確認する、

ことが学習コスト的にもセキュリティ的にもベストな気がする。

(けどあまりそう書いてあるページがないから正直自信がない)

デメリットとしてはSecureなCookie使ってるから開発環境もSSL化しないといけないってことかなあ。

---

### 最後に

JWT周りの理解が浅いため、もしかしたら認証周りはトンチンカンなこと言ってるかもしれない…

結構頑張って書いたので、間違ってるところとか参考になるサイトがあったらTwitterか何かでガンガン連絡頂けるとめちゃめちゃ嬉しいです。

#### 参考にした書籍、サイト、記事等

アフィリエイトとかではないのでご安心を…

- [徳丸 浩 『安全なWebアプリケーションの作り方』](https://www.sbcr.jp/product/4797393163/)
- [水野 貴明 『Web API The Good Parts』](https://www.oreilly.co.jp/books/9784873116860/)
- [MDN web docs](https://developer.mozilla.org/ja/)
- [Understanding "same-site" and "same-origin" -- Web.dev](https://web.dev/same-site-same-origin/)
- [HTTP クッキーをより安全にする SameSite 属性について (Same-site Cookies) -- ラボラジアン](https://laboradian.com/same-site-cookies/)
- [JSON Web Token (JWT) -- OpenID Foundation Japan](https://openid-foundation-japan.github.io/draft-ietf-oauth-json-web-token-11.ja.html)
- [【JWT】 入門 -- Qiita](https://qiita.com/Naoto9282/items/8427918564400968bd2b)
- [クロスサイト・リクエスト・フォージェリ -- IPA](https://www.ipa.go.jp/security/vuln/websecurity-HTML-1_6.html)
- [CORSを許可する -- Larapet](https://larapet.hinaloe.net/2020/03/05/laravel-cors/)
- [mala/gist:8857629 -- GitHub](https://gist.github.com/mala/8857629)
- [SPAのログイン認証のベストプラクティスがわからなかったのでわりと網羅的に研究してみた〜JWT or Session どっち？〜 -- Qiita](https://qiita.com/Hiro-mi/items/18e00060a0f8654f49d6)
- [JWTを使った今どきのSPAの認証について -- HiCustomer Developer's Blog](https://tech.hicustomer.jp/posts/modern-authentication-in-hosting-spa/)
- [徳丸浩 -- 質問箱](https://peing.net/ja/q/e23f2f51-d437-4b1f-bb36-61d8172babb6)
- [Web API の CSRF 対策まとめ【追記あり】 -- Qiita](https://qiita.com/okamoai/items/044c03680766f0609d41)

<script async src="//iframely.net/embed.js" charset="utf-8"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
