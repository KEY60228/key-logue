+++
author = "KEY"
title = "【Docker】Dockerでよく分からんかったとこ"
image = "/img/docker-memo1/kujira.png"
date = 2020-07-31
description = "Dockerについてしっくりこなかったけど調べてみたら腹落ちしたこと"
tags = ["Docker"]
categories = ["Tech"]
archives = ["2020/07"]
draft = false
+++

ここ1.2ヶ月の間なかなかしっくり来なかったDockerについて、最近ようやく少し分かったような気がするのでまとめてみる。

---

### Dockerとは

省略。笑

下記Qiitaの一連の記事がとても分かりやすかったのでおすすめ。

<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://qiita.com/etaroid/items/b1024c7d200a75b992fc" data-iframely-url="//iframely.net/R3CEOku?card=small"></a>
    </div>
</div>
<br>
<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://qiita.com/etaroid/items/88ec3a0e2d80d7cdf87a" data-iframely-url="//iframely.net/tr8tEWe?card=small"></a>
    </div>
</div>
<br>
<div class="iframely-embed">
    <div class="iframely-responsive" style="height: 140px; padding-bottom: 0;">
        <a href="https://qiita.com/etaroid/items/40106f13d47bfcbc2572" data-iframely-url="//iframely.net/2rSQmUy?card=small"></a>
    </div>
</div>

これ見て「Docker完全に理解した」なんて一瞬思ったものの

実際に触ってみたらよく分からんポイントが結構あったので、(いるか分からんけど)同じようなポイントでモヤモヤしている方の参考になれば、とここに書き残します。

(あるいは過去を振り返りたい未来の自分へ)

---

### ホストOSとカーネルを共有する？

> 実際にDockerは、ホストOSのカーネルを共有する -- [wikipedia](https://ja.wikipedia.org/wiki/Docker)

まず引っかかったのが、色んなところで目にする「コンテナがホストOSとカーネルを共有する」ってこと。

ホストOSって(自分の場合)MacOS？？？

Windowsユーザーとどうコンテナを共有するの…？？

Docker分からん…

→ そもそもの構造の理解が間違ってた。

> MacOSはLinuxじゃないのでDockerは起動出来ません。
> なのでDocker for Macというアプリケーションをインストールした時に、準仮想化技術を使ってシンプル高速なLinuxを裏で仮想マシンとして動かして、その仮想マシン内でDockerを利用しています。
> -- [teratail](https://teratail.com/questions/142866)

DockerはLinuxのコンテナ技術を応用したものなので、そもそもLinux上じゃないと動かない！

なのでDocker for Macの裏側で仮想Linuxを立ち上げている！！

…つまり上で言っている「ホストOS」ってのはMacOSやらWindowsやらのことを言っているのではなく、Docker for Mac や Docker for Windows (厳密には hyperkit や hyper-v ) で立ち上げた仮想Linuxのことを言っていたのでした！！(スッキリ！！)

{{< tweet user="key60228" id="1273245457385316354" >}}

---

### あれ、このイメージのOSなんだっけ？？

次に引っかかったのはDocker Hubで共有されてる各種イメージを見た時。

仮想Linuxを立ち上げてその上に各種コンテナを載せていくのは分かった。

そんでDocker HubからPHPとかPostgreSQLとかApacheとかのイメージ引っ張って来て簡単に使えるのも分かったけど…

これコンテナの中のOSって何なの？

なんかそれぞれapt-getとかapk addとか、違うディストリビューションっぽいコマンド打ってるけど、大丈夫なの？？

Docker HubでDockerfileのコマンド見れば分かるかな、って思ったけど、これナニ？？？

{{<figure src="/img/docker-memo1/docker1.png" alt="謎の文字列">}}

→ まずコマンド、見るべき場所が違った。

Dockerfileはここ！

{{<figure src="/img/docker-memo1/docker2.png" alt="Description">}}

そんでGitHubに飛んで覗いてみたら…

{{<figure src="/img/docker-memo1/docker3.png" alt="GitHub">}}

バッチリDebianって書いてあった！

他のサービスは違うディストリビューションだけど大丈夫なんか…？？って疑問も…

> 実はLinuxは共通仕様としてマシン語で書かれている実行ファイルのフォーマットが同じ形式です。
> 違うディストリビューションでも動作し、適切な依存ライブラリを渡してやれば動作します。
> -- [teratail](https://teratail.com/questions/142866)

だそうです！！(まだフワッとしか分かってない)

---

### その他引っかかったとこ

#### コンテナ間でのファイル共有ってどうやんの？？

Dockerのことをよく知らずにLaradockから入ってしまったもんだから、workspaceに入ればWEBサーバーのログも見られるしDBもいじれるしファイルも触れるのが当たり前だと思っていた。

でもいざ自分でDockerfileとかdocker-compose.yml書いてみよう思ったらどうすればいいか全く分からん…

→ ローカルの1ディレクトリにそれぞれのコンテナからマウントする。

これだけ。

何故か1コンテナ1ボリューム、みたいな考え方をしていたため、この発想がなかなか出てこなかった…(お粗末)

#### 1コンテナ１プロセス？？

「コンテナ設計は1コンテナ1プロセスが理想」らしい。

WEB、アプリケーション、データベースサーバーをそれぞれ別コンテナで分けて、通信させるのはなんとなくイメージつくけど…

gitとかcomposerとかnode.jsとかどうしたら…

→ あくまで理想論。厳密に１コンテナ１プロセスは無謀。([TechRacho](https://techracho.bpsinc.jp/hachi8833/2014_06_16/17982)参照)

正直そんなに引っかかったわけじゃないけど、コンテナ設計するに当たってどこまでコンテナ分割したらいいんだろ…？としばらく悩んでたのでピックアップ。

---

### まとめ

初めてDockerに触れた時は、「なんだかお手軽で簡単そうじゃん！」なんて思ったけど、結局Apache + PHP-FPM + PostgreSQL + workspace (composer, node.js, git, postgresql-client...)の4つのコンテナ構成作るのに1ヶ月近くかかってしまったりと、なかなか自分のモノにできなかったなあ。

Dockerの教材漁ってたけど、結局Dockerって言うよりはネットワーク周りとかLinuxの知識が全然足りてなかった気がする。

この1.2ヶ月、Dockerにめちゃめちゃ時間吸われたけど、まあその辺の知識も多少はついたと思うから良しとしよう…笑

間違いとかあったらご連絡頂けると嬉しいです。

また何か思いついたら随時更新してみようと思います。

<script async src="//iframely.net/embed.js" charset="utf-8"></script>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
