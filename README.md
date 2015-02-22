# MOGUERA

最近、プロジェクトの度に同じ様な構成のシステムを量産している気がする。

それは、バックエンドAPI Server、フロントViewアプリケーション、ユーザライブラリ、管理用コマンド、そして認証が揃ったシステム。

そろそろ時間の無駄なので、車輪の再発明(自分の中で)はもう辞めたくなった。

だが、1人でのノロノロ休日プロジェクトのため、進捗は非常に遅い。

## テーマ

このプロジェクトで作るものは、フロントとバックエンドが分かれていて、認証はやり過ぎず、やらな過ぎずな感じで、APIを操作するためのユーザライブラリがあって、管理用のコマンドがあるCMS的な何か。

これをベースに、ブログやツイッターみたいなものがサクッと作れれば、他の新サービスのベースとして応用できないだろうか。また、将来的にスケールアウトが可能な構成だと、会社も上司も自己も満足なはず。

目的のために言語は問わないが、それなりに安定性があって、普段仕事でも使っていて運用実績もあるRubyでの実装が個人的に良い。

なんとなくこんな推奨構成案。


```
+--------------+ +-------------+
| DeskTop User | | Mobile User |
+-----+---+----+ +-+-----------+
      |   |        |
+-----+---+--------+--+
| nginx load balancer |
+-----+---+--------+--+
      |   |        | 
+-----++ ++-----+  |  +-----+
| web1 | | web2 |  |  | CLI |
+-----++ ++-----+  |  +-+---+
      |   |        |    |
 +----+---+----+   |    |
 | API Servers +---+----+
 +------+------+
        |
 +------+------+
 | DB Clusters |
 +-------------+
```

上から見ていくと、この構成のアクセス対象はブラウザアクセスのユーザとモバイルアクセスのユーザであろう。

ブラウザアクセスのユーザはnginxによりwebアプリケーションにリバースプロキシされる。
モバイルユーザはnginxにリバースプロキシされ、APIサーバのAPIを叩く。

管理の楽さを考えるとnginxでのリバースプロキシ+ロードバランスが良さそうだと考えている。
ロードバランスと聞くと、会社では、DNSラウンドロビンでのバランスが流行りなのだが、幾分難しいので...

webアプリケーションはMOGOKとかHerokuとかWebインスタンスをポコポコ足してスケールアウトできることを考えている。

APIサーバはAPI別にサーバを分けても良いし、分けなくても良い。

普段仕事では[MOGOK](https://mogok.jp)というPaaSを作っているので、フロントアプリケーションとバックエンドAPIサーバはMOGOKにデプロイ可能な構成だと尚良い。

DBはMHAとか準同期レプリケーションとかpgpool2とかRDBMSが好み。

## とりあえず

Mogueraはゴジラvsスペースゴジラに出てくる対ゴジラ作戦用飛行型機動ロボットである。

Mogueraは飛行用のスターファルコンと地上用のランドモゲラーに分離することが出来る。この分離するところが気に入った。

ゴジラと戦うために人類が作った、一見弱そうだが意外とやる奴がMogueraなのだ。

## コンポーネント

それぞれのコンポーネントをMogueraシリーズとでも呼ぼう。

それでは、Mogueraシリーズを解説していく。

シリーズの共通仕様として、メジャーバージョンv1を完成形とする。
そしてnode.jsに習って、マイナーバージョンが奇数系のものを開発版、偶数系を安定版としておく。

### [moguera-core](https://github.com/moguera/moguera-core)

RESTfulなAPIを持つAPIサーバ。

### [moguera-authentication](https://github.com/moguera/moguera-authentication)

認証ライブラリ。S3チックな認証機能を提供する。

### [moguera](https://github.com/moguera/moguera)

moguera-coreとmoguera-authenticationを合体。

### [moguera-driver](https://github.com/moguera/moguera-driver)

ユーザライブラリ。mogueraをより簡単に扱うことを目的にしている。

moguera-driver-rubyとかmoguera-driver-nodeとか言語毎にあっても良いかもしれない。

### [moguera-view](https://github.com/moguera/moguera-view)

Webアプリケーション。driverを込みこんでおり、Mogueraのリソース管理を行うことが出来る。そのままブログとしても利用可能。

moguera-view-railsとか、moguera-view-expressとか色々あっても良いかもしれない。

### [moguera-cli](https://github.com/moguera/moguera-cli)

単純にdriverを組み込んだコマンドラインインタフェース。管理・運用用にはやっぱりコマンドが求められる。

## まとめ

スタートアップ時の小中規模開発用にこんなシステムあったら良いなを実現したい。

Mogueraシリーズ手伝って頂ける仲間募集中 (mOgOk)!
