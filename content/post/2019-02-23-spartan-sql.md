---
date: "2019-02-23T00:00:00Z"
title: 'カンム社員全員でSQL勉強会をやっている話'
---

[株式会社カンム](https://kanmu.co.jp/) では、毎週火曜日17:00 - 18:00の時間をとって全員参加のSQL講座をやっている。2018/4から続けているのでそろそろ10ヶ月程経過するのでどのように継続してきたか、どのような効果があったか等を書く。

### 前提

カンムは [バンドルカード](https://vandle.jp/) というサービスを提供しており、2019/01末時点で正社員13名のまだ小さなチーム。このSQL講座開始当時、SQLを業務で書ける人員は社長を含め5名くらいだった。また、[redash](https://github.com/getredash/redash) は本番/検証ともに整備されており、以前は主要KPIのダッシュボードは社長の [8maki](https://twitter.com/8maki) と自分が作ってた。

バンドルカード自体を作ってる様子はこちら。

- [バンドルカードを作ってる by achiku](https://akirachiku.com/post/2017-05-12-building-vandle-card/)
- [バンドルカードができるまで by ideyuta](http://ideyuta.com/vandlecard/)

### 発端

自分がソフトウェアエンジニアからマーケティング側に移籍した話は [前回書いた](https://akirachiku.com/post/2019-01-27-i-am-software-engineer/)。長いのでまとめると、今まで5年程ソフトウェアエンジニアをやってきたけど社長に打診されて面白そうだったからマーケティング側に移籍し役員になったという流れ。

まず何やろうかなという感じになってたんだけど、ソフトウェアエンジニアリングにもある「推測するな計測しろ」という格言は、マーケティングであろうともある程度の範囲までは確実に役に立つはずなので、運用型広告から入ってくるユーザーがどのような推移になっているのか、各経路別MAUやリテンションレートはどうなっているのかを調査する為のダッシュボードをredashに作った。(以前からMAU、GMV、リテンションレート、等の基礎的なダッシュボードはあったけど運用型広告軸のはなかった)

この時点でこういう簡易な調査を全員が自発的にできたら強いなと思った。

精度の良い仮説を作る為にはある程度データを幅広く見て、その中から異常値/気になるポイントを見つけ、なぜその値が発生しているか調査していく流れがあると思う。もちろんデータを見るのではなく、何気ないユーザーのツイートやユーザーインタビューから仮説立案が始まることはあるとは思うんだけど、最も手軽に始めれるのがデータを見ること。これを自分ひとりだけしかできない状態は非常に効率が悪い。

また、こういう検証は、仮説立てては潰してを繰り返して確からしいものを見つけていかねばならない為、手数が重要になる。特に展開しているサービスがアプリだと運用型広告担当がGoogle Analyticsやadjustに閉じずに諸々のデータを見ながら訴求軸を考え続けていかなければ効率的な運用はできない。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">スパルタンSQLという講座を社内で開始した</p>&mdash; アチク (@_achiku) <a href="https://twitter.com/_achiku/status/986814816454041600?ref_src=twsrc%5Etfw">2018年4月19日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

スパルタンSQL(SQL講座)はこういった背景から生まれた。

### やり方

#### 誰がやるのか

発端は「マーケティング側が自発的にデータちゃんと見れるの良いな」だった。ただ、このタイミング以前に [xOpsと組織設計](https://y-matsuwitter.booth.pm/items/846384) という [y_matsuwitter](https://twitter.com/y_matsuwitter) さんが書いたマッハ新書を読んでおり、めちゃくちゃ感銘を受けていた為、一旦対象は総務、経理、財務、カスタマーサポート、マーケティング、デザイナーと制約は設けずに実施することにした。正直難しければデータ見るの必須のマーケティング側だけに絞ればいいやろという見切り発車だった。

xOpsに関しては書くともう一つ記事が書けてしまうので一旦冒頭の引用のみに留める。

- xOps + KPI経営で組織のアーキテクチャ設計をしよう
    * 様々なツールを組み合わせ、状態や結果の計測を自動化することで、組織の各メンバーが最も価値のある活 動に集中・貢献し、その活動に伴う課題も施策の結果も自動的に見える状態まで組織を設計することができる。
    * 人事や営業など全てのチームをプロダクトの一部とし て認識し、何をインプットとし、何をアウトプットと するか、それらをどのように組み合わせていけば最も円滑に価値を作れるか、エンジニアリングしていくことがこれからの組織のあり方かもしれない。


え、冒頭の引用だけで最高では？となった方は↑のリンクから買って読みましょう。実際最高です。

講師は開始時点では一旦achiku一人で、カンムのフロントエンドの守護神だけどバックエンドも余裕で書ける [moqada](https://github.com/moqada) にティーチングアシスタント(TA)的な動きをお願いし、演習中に自分と一緒にみんなの質問に答えるという形で参加してもらっていた。

#### なぜやるか

まずなぜやるのか、以下のような内容を説明した。当時の esa からそのまま持ってきたもので雑だなぁと思うけど大枠外してない。

- 全領域で事実に基づいた意思疎通をしたい
    * AとBのUIはどっちがどのように良いのか
    * この施策を打ったらこの値がx%くらい改善するはず
    * この値を見ると撤退ラインはxですね
    * 数値aを見ると他の人と違う動きをしてる人が一定数いるのでその人達にインタビューしたい

実際はこれ以上の効果があったと思ってるんだけどそれは後述する。

#### どうやってやるのか(環境準備編)

まずSQLを実行する為の安全な環境を用意した。本番の redash で参照しているDBの日時バックアップを利用してSQL勉強会用のDBをプロダクション環境に随時作成し、そのDBの読み取り専用ユーザ経由で redash からアクセスする。これにより本番のDBに影響を与えることなく、本番相当のテーブル定義とデータを使ったSQLの練習ができるようになる。
またセキュリティ的な観点からも、通常の社員が redash からアクセスがすることが許可されているデータなので問題ない。

#### どうやってやるのか(講義編)

スパルタンSQLは毎週1時間必ず実施。最初の方は前半講義、後半は演習という形式で進める事にした。また、一番最初の講義で「なぜやるのか」部分を説明した。理念は腹の足しにならないので30秒くらいさらっと。それよりも「コレ使えるの便利やんけ」や「これは面白いな」の方が先に来る動線設計の方が重要と思う。

第一回はSQLの基礎であるデータベース、テーブル、テーブルの中に入っているカラムとレコードという概念を説明する。ここでは関係代数が〜みたいな話はすっ飛ばし、「テーブルって要はExcelの表みたいなものです」から始まり、1回目の講義でとにかく `select` 、 `where` の概念を説明し「全件取ってくると時間かかるから最初は `limit` を付けてやってみてね」と伝えるに留める。

ある程度概念を説明したらとにかく下手でも良いから実行してみてもらう。これは自分がプログラミングを学んだ時に思ったことなんだけど、「高速にフィードバックを受ける」ことは理解の取っ掛かりを掴むのをとても助けてくれる。もちろん対象のDBは練習用のDBなので本番環境に影響を与える事は無いのでそこは安心して実行できるような仕組みが重要になってくる。

もう一点重要な事があって、それは本人が業務上知りたいと思う事を演習時間に1つ決めてデータを取得してもらうこと。これをエンジニア2名が全力でサポートしにいく。結局、仕事で使えて便利！なんか難しそうだと思ったけど自分もできた！という感覚自体がSQLのアハモーメントだと思うのでなるべくこれを早めに体験してもらい、アプリの演出の代わりにachikuがめちゃくちゃ騒ぐ、みたいな事をやっていた。

講義に関してはその後ざっくりな区切りではあるけど、JOIN、GROUP BY、サブクエリとCTE、よく使う関数1、よく使う関数2、HAVING、集合演算(except/intersect/union)、CASE基礎、CASE応用、Window関数基礎、Window関数応用、PostgreSQLをローカルで立てる、あたりまで存在する。また、redash自体細かいtipsは知ってるだけで生産性が爆上がりするものがある。なのでredashの使い方も講義する。各種グラフの作り方、パラメータ付きSQL、他のクエリ結果から別クエリへのリンクを生成する方法、QueryResultデータソースの使い方、Google Spreadsheetデータソースの使い方、等の講義までとはいかない小話も非常に良かったとのフィードバックをもらっている。

講義の後半は自分ひとりだと回らなくなってきた且つSQL書けるエンジニアが増えたのもあり、担当割り振って講義をお願いしていた。

#### どうやってやるのか(相談編)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">SQLに習熟する為の専用Channel作った <a href="https://t.co/JiVUXX0i7s">pic.twitter.com/JiVUXX0i7s</a></p>&mdash; アチク (@_achiku) <a href="https://twitter.com/_achiku/status/989715283039289344?ref_src=twsrc%5Etfw">2018年4月27日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

そうこうしているうちに業務で使うSQLの質問が業務時間中にポロポロあがってくるようになった。めちゃくちゃ良いことだと思ったのでそれ専用のチャネルを作った。このチャネルではSQL関連の悩み事を質問するとSQLわかるマン達が答えてくれる。特に個人に対してメンションが行って答える、とかではなくてゆるくこのチャネルにいる人達が回答していくのでエンジニアだけが張り付いて答え続ける、みたいなのではない。「あ、それはこのクエリが参考になるよ [redashのURL]」みたいなのを全員が自然に行う流れになっている。

#### どうやってやるのか(クラス分け編)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">弊社は社内SQL講座のことを「スパルタンSQL」と呼んでいます <a href="https://t.co/zPAjFcc0v1">pic.twitter.com/zPAjFcc0v1</a></p>&mdash; アチク (@_achiku) <a href="https://twitter.com/_achiku/status/1064710807454343168?ref_src=twsrc%5Etfw">2018年11月20日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

しばらく継続していると、昔プログラミングやっていた、業務で使う頻度が高い、自分で本買って読んでる、等の理由で他の人達よりも上達の速い人が現れ始めた。また新規に入社してきてくれた人たちはゼロからスタートとなってしまう為、いきなり集合演算とか言われても...という状態が発生。非常に嬉しい反面画一的な講義だけでは非効率な状況になった時期がある。対応策として、とりあえず上級コースを併設し、その人達には演習メインの講座に移行してもらった。上級コースの人たちに関して言えば、Window関数を理解し、redashのピボットテーブルを使って経理データ取得を効率化しはじめている時期で、「え？半年で？俺が今まで学習に費やした時間とは...」という戸惑いがあると同時にめちゃくちゃ誇らしかったのを覚えている。

現状は入社タイミング別にある程度クラスを分け、自分で上級者行くかとなったら上級者コースに乗り換える、上級者が初級者に教える、みたいな形にしていこうかなと考えている。ただこの辺はまだどうやっていくのが最適なのかわからない為、適宜実験しながら進めていきたい。

#### どうやってやるのか(演習編)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="und" dir="ltr"><a href="https://t.co/2YrXrn0Q0R">pic.twitter.com/2YrXrn0Q0R</a></p>&mdash; アチク (@_achiku) <a href="https://twitter.com/_achiku/status/1077490642924691456?ref_src=twsrc%5Etfw">2018年12月25日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

毎週講義をしていると大体半年くらいで一周してしまう。そこで講義編が一通り完了したらTAが付きそいながら1時間演習問題を解く、もしくは自分の知りたい事実をSQLで取得する時間にしている。

このタイミングの直前くらいで1人でやり続けるのがキツくなってきたので演習問題作成を他のSQL書けるエンジニアに依頼し始めた。現在は以下のようなフォーマットを整え、これに沿って演習問題を作ってもらっている。これが蓄積していくことで難易度別の練習問題一覧を作って行くのが最近の目標。

```text
## 問題

### 問題文

ユーザー名からその人のが入ってきた広告経路を取得する。

### 利用するテーブル/カラムの説明

- `xxxxxxxx.network_name` に広告経路が入っている
- `zzzzzzzz.display_name` ユーザー名が入っている

### ヒント

- `zzzzzzzz.user_id` と `xxxxxxxx.user_id` はjoinできる

### 回答SQL

https://redasdomain/queries/29/source
```

#### どうやってやるのか(継続編)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="und" dir="ltr"><a href="https://t.co/I48EDmImNs">pic.twitter.com/I48EDmImNs</a></p>&mdash; アチク (@_achiku) <a href="https://twitter.com/_achiku/status/1092706031430393856?ref_src=twsrc%5Etfw">2019年2月5日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

継続するの難しいんだけど自分がやる中で思ったコツが2つある。

1つめは「多少雑になっても実施すること」。スパルタンSQLは毎回100%の品質で提供できてるかと言われると、実際そうではない。ぶっちゃけ50%だな...みたいな週もあるし、普通に業務もガッツリあるので直前にあわてて資料や問題書いたりしてる事もある。それでもやる。受講してる人には少し申し訳ないな...と思いながらも、クラス分けによって複数回講義することは確定なので可能な限りの説明をして、次に同じ講義をやる時に活かす。

2つめのコツは「助けてもらうこと」。最初期から [moqada](https://github.com/moqada) さんにTAをお願いしてたし、今は [hiroakis](https://github.com/hiroakis) や [yy](https://github.com/yosukeyoshida) に問題作成をお願いしてたりする。[膝](https://github.com/mururu) はバスケット分析とかの講義してくれるし、 [iruka](https://twitter.com/iruka__u) は redash の tips めちゃくちゃ詳しいし、[ウチのCFO](https://kanmu.co.jp/interviews/daisuke-ito/) は既に並のエンジニアよりSQL書ける。彼はなんなら今Python書いてる。

こういう風に社内の人々にこういうことやりたいんや！！助けてくれ！！とお願いすると助けてくれる気のいいチームだからできる、という大きな前提はあるが、一人でここまで継続するのは無理だったんじゃないかなと思う。

ただし、最初の一歩である「雑でも絶対毎週やる」みたいなものがないと「一人で毎週やるの辛いから負荷分散できるようにしたい。どうやってお願いしたらいいか。」みたいな思考も出てこない為、やはり「どんだけ忙しくても雑になっても絶対毎週やる」という覚悟は大切だったなと思う。

#### どうやってやるのか(精神論編)

1点だけこれは精神論だなと思うことがある。それが「絶対に誰でも書けるようになると信じる」ということだ。根気強く続けていれば最初めちゃくちゃ苦手な人でも少しずつできるようになる。そしてそういう風にできるようになってきて嬉しそうにしている人を見ているとこっちも純粋に嬉しい。

Twitter見てるとプログラミングできる人たちはともすると「俺たち特殊なんで」感を出してしまうことがあると感じる。気持ちはわかる。一端のプログラマになる為には長い時間が必要だし、そのプロセス自体はともすると凄い孤独だ。自分にも時間をかけて上達したという自負はある。ただ、最初は自分も書けなかったですよね、という部分を忘れて初心者に接してしまえばこのような講座を開き、チーム全員で強くなろうぜ、みたいな雰囲気を醸成する事は難しかったんじゃないかなぁと思う。

### 効果

まずマーケティング側の施策を打つ際に必ずredashのurlが添付されるようになった。これは別のポストで詳細に語りたいんだけど、弊社では「仮説ノート」と「新規施策ノート」という esa のテンプレートが存在する。これらのテンプレートに必ずredashのurlを載せ、仮説や施策を事実を鑑みながらレビュー可能になった。また、施策を打った後に「あれ実はデータがなくて検証できなかった...」みたいなダサい失敗を撲滅する為に施策打つ前に検証用のSQLを書く習慣がついてきていると感じる。

また、ある程度のサイズになった会社では凄いデカイと思うんだけど、経理が帳簿をつける際に行うエンジニア勢とのコミュニケーションが非常に円滑になった。「[redash url]で取得できるデータに関して質問なんですけど」みたいなコミュニケーションが可能になったことで経理側が仕様を把握して月次の帳簿を付けていけるようになっている。

加えて、カスタマーサポート側でもリアルカードの本人確認の作業効率をモニタリングするダッシュボードが作成され、どのような仕様を追加することで本人確認書類のリジェクト率を下げれるか等の議論が発生している。また、小さいチームだとどうしても後手に回りがちな管理画面を、参照系のみならredashのパラメタ付きクエリとaタグで簡易的に構築できるようになっている。

副次的な要素ではあるが各人がサービスのデータ構造を把握することで仕様議論がめちゃくちゃスムーズになった。マーケティング側が、デザイナー側が、どの辺のテーブルを使えばこの変更を行えそうか、という感覚があるのはソフトウェアエンジニア側としては超絶ありがたい。

この他にもredashとSlackを連携し、値Aがとある閾値を超えるとアラートを飛ばしサポートが介入する仕組み等、上げたらキリがないほどの改善が社内で発生している。しかもソフトウェアエンジニアが介在しない場所で、だ。

みんなすごいなオイ。

### 今後の展望

データは見れるようになってきたので、去年後半くらいから「仮説会」を毎週30分実施している。結局プロダクトをよりよくしていく為に必要な「筋の良い仮説作る力」ってどうやって身につくんだろうなという思いから始めた会になる。まだその問に対する明確な答えは出ていないが、継続して整備いく事でこの技術に関しても多くの人が習得できるような仕組みができれば良いなと思っている。

現状はachikuがデータ/アンケートを見てて気づいた特徴的な動きから仮説を立てて、その仮説に対しての検証を更にデータを使って行う、という会になっている。一通り型みたいなものはできてきたので、今後は自分以外の人にも継続的に発表していって貰おうと思ってる。

### まとめ

「事実を見て意思決定をする」。めちゃくちゃ当たり前で、そんなもの太古の昔からそうだろという思いもある。インパクトのある仮設は何かを考え抜き、観測可能な事実は何かを根気強く定義し、今まで観測できなかった事実を観測可能にし、それらを用いて仮説の真偽を検定し続けるということは一定数の人類、特に科学と名のつく領域にいる人間はみなやってきている。なので特に新しいことをやっているつもりはない。むしろプロダクトの文脈では「発生した事実」から迫真の具体性を紡ぐ力の方が希少な技術じゃね、と思う部分もある。

ただし、「発生した事実」をまずは集計できなければどこにも行けず、そして何よりも「チームとしてまず事実に当たる文化を手に入れる」為に10ヶ月ほど行ってきた経緯をまとめた。今後もチームとしてまずは事実に当たり、集計し、比較し、それを解釈し、そこから迫真の具体性を紡げるようになれるよう、全員で研鑽を続けていく。

どのような職種で入ってもスパルタンSQLは常にオープンに参加できる環境にしていきたいと思ってる。興味ある！と思った方は全職種積極的に採用中なので是非！！！！！！！

- [株式会社カンムの会社情報 - Wantedly](https://www.wantedly.com/companies/kanmu)