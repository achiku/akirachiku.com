---
date: "2018-11-30T00:00:00Z"
title: 'net/httpで作るGo APIサーバー #7'
---

2018/11/25、GoCon 2018 Autumnで "Beloved database/sql. How we go test with RDBMS" というタイトルで発表してきました。

<script async class="speakerdeck-embed" data-id="4d5d662a5bf843c6b6c2db22d19f39f9" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

> Many of API servers tend to interact with RDBMS to serve structured data for frontend. Unlike other languages with full-featureed web application frameworks, it seems, at first glance, a little difficult to write tests for Go applications using RDBMS. However, knowing `database/sql` with a bit of RDBMS knowledge is just enough to write clean tests for RDBMS backed Go applications. In this talk, I'll describe how to write efficient RDBMS backed Go application tests.

ということでRDBMSを使ったGoアプリケーションをどのようにテストするのが良さそうかということにフォーカスを絞った発表になります。DjangoやRuby on Railsのようなfull featuredなフレームワークから学び、それをGo+RDBMSのアプリケーションのテストに活かすにはという部分は調べていて自分も勉強になりました。
また、@deeeetさんが以前「GoConは今後グローバルにやるんで！！」と言っていたので今回は発表/QAを全て英語で実施しました。オーディエンスがほぼ日本人なのに日本人が英語で発表するという謎のスタイルになってはしまいましたが、友人が「やる」っつってんだからやるっしょというある種勢いだけで20分発表した形になります。
みなさんの発表もとても刺激的でとても勉強になる会となり、会場を提供してくれたGoogleさん、ボランティアで会場運営していただいたみなさんには本当に感謝です。とても楽しい会をありがとうございました！


また、この資料でDBの扱いとそのテスト方法までカバーできたので、一旦Go APIサーバーシリーズは終わりにします。自分はもうコードを書かなくなってしばらく経っているので、詳細な工夫や"リアル"な話を書けなくなってしまったなという思いがあります。今は違う業種にチャレンジしているのですが、そこでのリアルもまた機会があれば書けたら良いなと思います。

- [net/httpで作るGo APIサーバー #1](https://akirachiku.com/post/2017-04-01-go-net-http-api-server-1/)
- [net/httpで作るGo APIサーバー #2](https://akirachiku.com/post/2017-04-02-go-net-http-api-server-2/)
- [net/httpで作るGo APIサーバー #3](https://akirachiku.com/post/2017-04-03-go-net-http-api-server-3/)
- [net/httpで作るGo APIサーバー #4](https://akirachiku.com/post/2017-04-08-go-net-http-api-server-4/)
- [net/httpで作るGo APIサーバー #5](https://akirachiku.com/post/2018-06-23-go-net-http-api-server-5/)
- [net/httpで作るGo APIサーバー #6](https://akirachiku.com/post/2018-08-11-go-net-http-api-server-6/)
