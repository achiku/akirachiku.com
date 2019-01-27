---
date: "2019-01-27T00:00:00Z"
title: 'ソフトウェアと'
---

### 2013: はじめに

約5年前にソフトウェアエンジニアになりたくて[前の会社を辞めた](http://localhost:1313/post/2013-11-18-job/)。当時3人の会社の4人目として入社。Web系のソフトウェアエンジニアの親しい友人はいない。その時からソフトウェアエンジニアコミュニティというものが存在していることは知ってたんだけど、どうしても好きになれくてその中に積極的に入っていこうという思いもあまりなかった。いわゆるスタートアップと呼ばれる会社だったけど、当時スタートアップ野郎には全く良い印象がなく、身内ノリがキモすぎてあまり関わりたくなかったので距離を取っていた。

会社で一日中設計してコードを書いて家に帰ってDjangoやfluent-agent-hydraやpaho-mqtt、気になったソフトウェアを写経して土日は自分が感じる不便を解決するOSSを書いてた。写経は脳を大きく動かさなくてもとにかく開始できるという一点において便利な練習で、その頃はよくやっていた。とにかく沢山読んで書いて、風呂に入りながら本を読んで寝て。それを繰り返す日々をおくっていた。

深夜、渋谷から誰もいない六本木通りを歩いて家に帰る道すがら「俺が一番強い」と声に出して唱え、エレカシの今宵の月のようにを小声で歌うのが好きだった。インターネットではしゃいでいちゃつきながら社畜自慢してるクソフェイクじゃなくて、本物のソフトウェアエンジニアになりたい。めちゃくちゃ傲慢なのはわかってるけど、ずっとそう思ってた。

> "I'm sticking to the script, you niggas skipping scenes"
> Lil Wayne

Lil Wayneも言っている。`be good, or be good at it` というリリックが大好きで、今も心の中で唱えてる。

### 2014-2015: OSSやろう

自分がOSSに出したPRで最初に受け入れられたのは[tagomoris/shibに対するPR](https://github.com/tagomoris/shib/pull/30)だ。何のことはない小さな修正だけど、会社で使ってたソフトウェアだし、tagomorisさんは自分がソフトウェアエンジニアになると覚悟したタイミングで以下の文章を書いていて、めちゃくちゃ励まされた覚えがある。今でも読み返す、何かが溢れてしまう文章だと思う。

- [4年前、おれがSIerの片隅で、何者でもなかった頃](https://tagomoris.hatenablog.com/entry/2014/02/25/091607)

このタイミングでソフトウェアエンジニアになろうと覚悟して大体1年くらい経過している。ここに至るまでに自分がリーチできる範囲内で、自分がリアルだと思う人に「良いソフトウェアエンジニアになるにはどうしたらいいですか」と聞きに行くということをしていた。[チクメキメモリアル](https://akirachiku.com/post/2015-08-23-chiku-meki-memorial) にまとめて書いたんだけど、seizansさん、Vさん、muddydixonさんには色んな助言を貰い、かつそれが活きている部分も多くとにかく感謝しかない。

転機になったのは、AWSのCLIである [jungle](https://github.com/achiku/jungle) を書いた時。initial commitは2015/07を記録している。ある時日曜日に仕事してて、何気なく[HackerNewsに投稿](https://news.ycombinator.com/item?id=10109339)したらすごい勢いでスターが増えていきテンションぶち上がってそのまま行きつけの飲み屋に行き、酒をかっくらいながらソフトウェアのことなんか何も知らないんだけどめちゃくちゃ良くしてもらってた店長に「俺やったんすよ！！これ見てくださいよ！！」って言いながらGitHubの画面を見せてたのを思い出す。店長は困惑しながらも祝ってくれた。

日本国内にソフトウェアエンジニアの知り合いが沢山いなくても、そのコミュニティーとやらに属さなくても、その道の人たちから少なからず認められたことは大きな自信になった。もちろんGitHubのスターはソフトウェアの品質を測る上では大して役にも立たないことは認識済みだけど、それでも、それでも本当に嬉しかった。

### 2016: Goとバンドルカード

2015年末頃から会社で新規のプロダクトであるバンドルカードを作り始めるぞ！という事になり、バックエンドはGoで行こうといういう決断をしてから必死で資料を読み込んでGoでHTTP APIサーバーを作るなら誰にも負けないくらいの知識をつけようと全部の時間をここに投資した。この時期に限定すればGoで書くRDBMS backedなHTTP APIサーバーのことを、品質はさておき日本で5本の指に入るくらい考えた自信がある。寝ても覚めても、どうやったら just enough なテストが書ける構造になるのか、RDBMSと相性の良い構造とは、パッケージ構成はどうすれば、各種外部サービスとのデータの整合性のとり方、フロントと役割分担を効率的に行える仕組みとは、海外の文献を読み漁って、書いて、潰してを繰り返していた。

2016年はまさに激動で、上記調査+設計+開発に加えて、プロダクトのプロトタイプ(Pythonで書いた)、ユーザーインタビュー実施、インタビューをもとにした仕様議論、AWSインフラ業、外部決済APIサービス提供者との仕様調整(英語)、をひたすらやっていた。あの当時からずっとチームには恵まれており、本当に殴り合いながら背中を任せられる良い状態で、だから自分もがんばれたんだと思う。とにかくチームでハチャメチャに良いものが作りたかった。

- [バンドルカードを作ってる by achiku](https://akirachiku.com/post/2017-05-12-building-vandle-card/)
- [バンドルカードができるまで by ideyuta](http://ideyuta.com/vandlecard/)

### 2017: GoCon

無事バンドルカードをリリースし、明けて2017/03、意を決してGoCon 2017に申し込んだ発表した内容は "How We Built Go Testable HTTP API Server"。これのCfPに通ったときは小躍りして喜んでたのを覚えてる。

- [How We Built Go Testable HTTP API Server](https://speakerdeck.com/achiku/how-we-built-testable-http-api-server)

発表の練習も結構ガッツリしたんだけど、実際の発表は時間が足りなくなって全然上手く出来ずに悔しい思いをした。この時も発表会場に知り合いはおらず、完全にアウェーなので閉会とともにすごすごと帰ってしまった。帰りの電車に揺られながら、やっぱりコミュニティーみたいなのに参加するのは難しいなぁと考えていた。

この辺りから「net/httpで作るGo APIサーバー」という一連の考えてることをブログに書き始めた。とにかく忘れないうちに学んだことを書いておこうというのもあるんだけど、やっぱり自分が色んな記事を読んで学んだ事を他の誰かに渡さなくては、という思いが強かった。


2017/11に再度GoConに応募して発表。このときは前回の失敗を活かして時間どおり終えることができた。このGoConはかなり印象深くて、mattnさんの本体が会場に来ており、database/sql関連の質問できたり、「アレがリアルなソフトウェアエンジニアだ」という実態を伴った思いと、直前にmattnさんに自分の出してるOSSにPRをもらってたり、めちゃくちゃ感慨深かった。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">PostgreSQL のテーブルから PlantUML の ER 図を生成するツール。これ待ってた。 <a href="https://twitter.com/hashtag/golang?src=hash&amp;ref_src=twsrc%5Etfw">#golang</a> / “GitHub - achiku/planter: Generate PlantUML ER diagra…” <a href="https://t.co/alSZEJvvvG">https://t.co/alSZEJvvvG</a></p>&mdash; mattn (@mattn_jp) <a href="https://twitter.com/mattn_jp/status/912135399174135808?ref_src=twsrc%5Etfw">2017年9月25日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

発表資料は[Async, Persistent, Fast, and Sable "Enought" Queue/Worker Using Go and PostgreSQL](https://speakerdeck.com/achiku/worker-using-go-and-postgresql)で、やっぱり自分はどうやったら Just Enough になるのかを考えるのが好きなんだなぁと思いながら資料を作っていた。(Just Enough 過ぎて今みんなに助けられまくっているので本当に申し訳ないという思いが最近強まってる)

もう一点、このGoConで転機になったことがある。清水の舞台から飛び降りるつもりでアフターパーティー的なものに参加したのだ。その中に [ymotongpoo](https://twitter.com/ymotongpoo) という男がいた。[すごいErlangゆかいに学ぼう！](https://www.amazon.co.jp/dp/B00MLUGZIS/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1)の翻訳者でその本にはお世話になっており、以前から知っていたんだけど、当時はツイッターで過激な発言を繰り返すマジで怖そうな人という印象だった。ただ、話がめちゃくちゃ盛り上がってしまい、途中から参加してきた当時飲み友達今は同僚の [膝](https://twitter.com/mururururu) や、Far Yeast Tokyo Craft Beer & Baoでビールを一緒に飲んで盛り上がった [Songmu](https://twitter.com/songmu) さん、ずっとブログを読んでてこの人達はマジでリアルだと思っていた [deeeet](https://twitter.com/deeeet) さんや [tenntenn](https://twitter.com/tenntenn)さん、GoConで [ebiten](https://github.com/hajimehoshi/ebiten) を発表していた [hajimehoshi](https://twitter.com/hajimehoshi) さんと、六本木小松のうっすい酒をガブガブ飲みながら、くだらない話からソフトウェアの話まで、なんというかすごい楽しかった。

なんか、こう、会社の外ででもリアルにソフトウェアと向き合ってる人たちとガッツリ話してめちゃくちゃ楽しい、というのは自分の中ですごい新鮮であり驚きでもあった。

### 2018: オライリー本

2018年、ymotongpooから一通のDMがTwitterに入る。その中には "Concurrency in Go"の翻訳をしているということ、そしてその翻訳レビューを手伝って欲しいということが書かれていた。食い気味に是非！！！と返し、とはいえ翻訳のレビューとかどうやってやるんや...しかもコレ、自分がずっとお世話になってるオライリー本やんけ...という不安もあった。

が、とにかく今まで死ぬほど(それこそ死ぬほど)お世話になっているGoという言語に対して、友人が信頼して出してくれた依頼に対して、全力でやらねばという気持ちでレビューをしまくった。正直自分はGoの並行/並列完全に使いこなせているわけではないが、英語と時代背景自体は読めるのでその辺りのニュアンスを中心にレビューをし、晴れて[Go言語による並行処理](https://www.amazon.co.jp/Go%E8%A8%80%E8%AA%9E%E3%81%AB%E3%82%88%E3%82%8B%E4%B8%A6%E8%A1%8C%E5%87%A6%E7%90%86-Katherine-Cox-Buday/dp/4873118468/ref=sr_1_1?ie=UTF8&qid=1548568988&sr=8-1&keywords=go%E8%A8%80%E8%AA%9E%E3%81%AB%E3%82%88%E3%82%8B%E4%B8%A6%E8%A1%8C%E5%87%A6%E7%90%86)が2018/10に出版された

- [「Go言語による並行処理」という本が出版されました #cingo](https://ymotongpoo.hatenablog.com/entry/2018/10/26/163000)

上のブログの謝辞部分を読んでいただければわかると思うんだけど、リアルを突き詰めてるソフトウェアエンジニア達ばかりの名前一覧に自分の名前が載ってるのが今でもむず痒いというか、だいぶおかしいというか、なんというか、違うという気持ちもあると言えばある。ただやっぱりすごい嬉しかった。これはマジなんだけど、この本に少しでも貢献できたことを誇りに思う。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">聞いてくれよ…オライリーの本にレビュアーとして名前載ったんや…マジでクソ嬉しい… <a href="https://t.co/fuUdFbu3Re">pic.twitter.com/fuUdFbu3Re</a></p>&mdash; アチク (@_achiku) <a href="https://twitter.com/_achiku/status/1053232333153742853?ref_src=twsrc%5Etfw">2018年10月19日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

この本のレビューの打ち上げの後、以下のようなツイートをしているんだけど、コレはこの文章の最初に戻ってもらえれば意味がわかると思う。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">昨日の翻訳レビュー打ち上げ飲み、尊敬するソフトウェアエンジニアの人たちと飲んでめちゃくちゃエモくなってしまって「俺は！！！一人でずっとコード書いてる時！！！！こういうコミュニティーを！！！！憎んでいた！！！！」って新宿駅で叫んでしまったんだけど完全に変質者だったな</p>&mdash; アチク (@_achiku) <a href="https://twitter.com/_achiku/status/1060094415102824450?ref_src=twsrc%5Etfw">2018年11月7日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

自分はこういうコミュニティーを憎んでいた。自分には無いものだったし、望んでも手に入らない場所にいたから。前世がイタリアのギャングなので仲よしクラブは間に合ってると思ってた。当時28-30歳とかなのを鑑みると相当ヤバイ程こじらせてるとも思う(今33歳)。

もし今こじらせてる人がいたとしたら、それはそれで良いと思う。歯を食いしばりながら、アイツラとは違うと思いながらやるのが良いと思うのであれば、その時間を大切にして欲しいなとも思う。先人の研究を参照しながら、真摯に課題を定義しそれに対する解決策を作る。それを人前で発表する/検索可能な場所に置いておく。楽観的すぎると馬鹿にされるかもしれないけど、その2つさえできてればきっと同じような情熱とリアルさで問題に取り組んでいる人たちと、いつか一緒に笑いながら語り合えると思ってる。

### 2019: ソフトウェア以外

去年の後半くらいから会社でソフトウェアを書く機会を減らしている。2018末に正式に株式会社カンムの役員になり、COOというポジションでマーケティングをやることになったから、というのもあるが、ドチャクソ優秀な [yy](https://github.com/yosukeyoshida) 、 [hiroakis](https://twitter.com/la_luna_azul) 、 [膝](https://twitter.com/mururururu) が会社に入ってくれて、自分がやっていた仕事をより上手くより効率的にやってくれているから、という側面もある。

あんなに「ソフトウェア書くぞ！！！！」という気合でやってきたのに結局自分もフェイクなのかな、と思うタイミングもあった。けど、社長の [8maki](https://twitter.com/8maki) にやってくれないかと言われ、自分自身も、

- 「ソフトウェアエンジニアリングは今後10年くらいで各職種の中に溶けていくだろうし、それの先鋒を切るのは面白そう」
- 「カンムがデカイ勝負をしようとしており、その中枢が元ソフトウェアエンジニアである、という状態でどういう力が発揮されるのかやってみたい」

という思いがあり、意外にすんなりと受け入れられた気がする。

また、ゼロからやる。

もう一回学んだ事をぶん投げて、更地の脳に新しい回路を構築していく。

慣れない場所はやっぱり怖いけど、ソフトウェアへの向き合い方と同じように、マーケティングに対して/プロダクトに対して、真摯に向き合って仕事を成していきたいと思う。