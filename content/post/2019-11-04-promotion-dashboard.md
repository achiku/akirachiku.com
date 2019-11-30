---
date: "2019-11-04T00:00:00Z"
title: 'カンム流redashを使ったプロモーションDashboardの作り方 その1'
image: '/images/20191130-promotion-dashboar/redash.png'
---

カンムではredashを使い倒している。とにかくお世話になっているredashではあるけど、特にマーケティングチームはかなりお世話になっている。最近、3テーブル+1 Google Spreadsheetあればとりあえずオンライン広告を利用したプロモーションのDashboardの大本になるようなものはできることに気がついたので以下に簡単にまとめる。かなり簡素化して書く予定なので身構えず読んで欲しいし、みんなもどうやってるのか教えて欲しい。

## 前提

まずは現在の前提から整理する。

- プロダクト
    * [バンドルカード](https://vandle.jp/)
    * 180万DL以上
- マーケティングチーム
    * CRM: 1名
    * 広報: 1名
    * 広告: 1名
    * 責任者: 1名(achiku)
- オンライン広告代理店
    * 1社とお仕事させていただいている
- 単語の定義
    * Paid広告 (CPM/CPI/CPA/CPV/CPE等の指標で配信もしくはアクションに費用が発生するオンライン広告)
    * Paidユーザー (Paidオンライン広告から新規登録したユーザー)
    * Organicユーザー (Paidオンライン広告以外の経路から新規登録したユーザー)
    * Paid CPA (広告配信費用[代理店フィー含む] / Paidユーザー数)
    * Blended CPA (広告配信費用[代理店フィー含む] + 広告クリエイティブ作成費/ Paid+Organicユーザー数)
- redash
    * 7.0.0以上
- データベース
    * PostgreSQL 9.5以上
    * 時間は全てtimestamp with timezoneで保存
    * DBのデフォルトタイムゾーンはUTC
- SQLの知識
    * CTE、Window関数の初歩は必要
- 広告計測SDK
    * [adjust](https://www.adjust.com/ja/)
    * アドフラウド防止機能追加済み
    * リアルタイムデータ連携でインストール/登録データはポストバックされている

また、今回redashのグラフ等も利用して説明したかったので以下にテーブル定義、ダミーデータ作成のスクリプトもまとめておく。時間帯別に登録者件数を変化させるために `generate_seriese` で5秒刻みの時間帯データを作り、そのテーブルから `tablesample` を使って時間帯別に傾斜が付いたデータを取得したんだけど、SQLだけで大量のデータ生成するのは結構面白かった。

- [achiku/redash-tips](https://github.com/achiku/redash-tips)


## テーブル定義

今回使う3テーブルは以下。install/registrationのイベントに連携先SDK名称を入れている。これは広告計測SDKを切り替える際に便利なので付けている(ある程度雌雄は決したといえ何があるかわからないので...)。また、カラムは今回説明の為に必要な最小限にしている。SDKによってはかなり細かい粒度まで取得できるので、自分たちのデータ含めて深ぼって分析したいチームは是非入れておきましょう。

- `adjust_install_event`
    * `id`
    * `network_name`
    * `registered_at`
- `adjust_registration_event`
    * `id`
    * `network_name`
    * `user_id`
    * `installed_at`
- `your_service_user`
    * `id`
    * `gender`
    * `birthday`
    * `registered_at`

また、今回はGoogle Spreadsheetにも登場してもらう。主にネットワーク別の費用を記録していく。項目は以下で、これも必要最低限に絞っているが `imp` や `click` など取れる媒体も存在する為、必要に応じてそのあたりも記録しておくのが良いと思う。

- `date`
- `network_name`
- `cost`

`date` は日付、`network_name` は adjust SDK から返される `network_name` と揃える。`cost` はそのまま、その日にそのネットワークに掛けた費用だ。「これはテーブルでも良いのでは？」という話もあるのだけど、後追いコンバージョンでコストが追加で乗ってきたり、代理店チームも参照する為、一括更新が楽+権限管理コスト低いGoogle Spreadsheetにしている。請求が確定したタイミングで、振り返りの為にもテーブルに入れるのが良いと思う。

注意点としては、「表示形式」をいじってシート上の数値にカンマや円マークが入るとredashのquery resultからクエリする際にうまいこと数値に変換できない。この辺もいつか上手いことできるようにしたい。

## 何を目指すか

プロモーションDashboardで何を目指すかだけど、大きく分けて4つ。

- 月次目標管理
- 異常値検出/対応
- 流入経路別ユーザーアクション/属性分析
- 過去の振り返り

気軽な気持ちで書こうと思ったけど長くなりそうな気がしてきた。

## 月次目標管理

### 日別累積新規登録

![](/images/20191130-promotion-dashboard/monthly-target.jpg)
※ダミーデータです

まずは月次で見た時の現在の日別累積新規獲得を、過去2ヶ月分の日別累積新規獲得と目標値を添えて出す。今回のケースは2019/11のタイミングで、10万件の新規登録を目標とする。Window関数を利用した累積獲得数の説明は省くが、コツは当月の日にち(30日ある月なら30)を `select generate_series(1, 30)` で出し、この数値に目標件数を日割りした数値を掛けることで月中n日における累積目標獲得を出す、という部分(`100000 / 30 * n as cumsum`)。SQLは以下。今見返してたら別にCTEにしなくても単純なサブクエリで出せますね。お好みでどうぞ。

```sql
with nums as (
  select generate_series(1, 30) n
), registration as (
  select
    to_char(u.registered_at at time zone 'JST', 'YYYY/MM') mt
    , to_char(u.registered_at at time zone 'JST', 'DD') dt
    , count(*) cnt
  from your_service_user u
  where u.registered_at >= '2019-09-01 00:00:00' at time zone 'JST'
  and u.registered_at < '2019-12-01 00:00:00' at time zone 'JST'
  group by mt, dt
)

select
  r.mt mt
  , r.dt dt
  , sum(r.cnt) over (partition by r.mt order by r.dt) cumsum
from registration r
union
select
  'target' as mt
  , lpad(n::text, 2, '0') dt
  , 100000 / 30 * n as cumsum
from nums
```

このグラフを見れば基本的に新規登録数のアヘッド/ビハインドは一発でわかるようになる。また先月、先々月の実績とも比較できるので、この伸びで行くと今月の着地はどの辺りなのかを見れるようになり、問題がありそうならそのタイミングで手を考え始めれる。

### 日別累積コスト

![](/images/20191130-promotion-dashboard/monthly-target-cost.jpg)
※ダミーデータです

日別累積のコストを出す。予算に対してどれくらいの進捗で予算が使われているのかは常に見ておきたい。先程も書いたが、コストはGoogle Spreadsheetで管理している。redashにはSpreadsheet用のデータソースもあるのでこれを作成し、Spreadsheetに対してクエリを書く形になる。若干ややこしいのは [query results](https://redash.io/help/user-guide/querying/query-results-data-source) という仕組みを利用する為、書けるSQLはsqlite用のものになる点だ。sqliteは3.28.0 (2019-04-16)くらいからいい感じにWindow関数が導入されており、redash最新版イメージではquery resultsに対してWindow関数が利用できるようになっている。それ以前のバージョンだと累積を出すためにcross joinしないと出せず、まぁまぁめんどくさい。以下は古いバージョン用のSQL。早めに最新イメージにアップグレードしたい。

```sql
with recursive generate_series(x) as (
 select 1
 union all
 select x+1 from generate_series limit 30
), t as (
  select
    date
    , sum(cost) cost
  from query_xxx
  group by date
)
select
  '2019/11' as mt
  , strftime('%d', a.date) date
  , sum(b.cost) total_cost
from t as a
cross join t as b 
where b.date <= a.date
group by a.date
union
select
  'target' as mt
  , substr('00' || x, -2, 2) date
  , (20000000 / 30) * x total_cost
from generate_series
order by date
```

運用型広告の場合、獲得ボリュームが伸びるかどうかはかなりクリエイティブに依存する。もちろん安定的な運用を目指すのだけど、どうしても上振れ下振れが発生してしまう為、このグラフを見ながらCFOと密に会話し、今月の予算着地をより精緻に見積もっていけるようにしている。

### 過去3ヶ月organic/paid別獲得推移 with organic率

![](/images/20191130-promotion-dashboard/paid-organic.jpg)
※ダミーデータです

次に過去3ヶ月のpaid/organic分類した上での日別登録件数を、organic率を添えて出す。redashのグラフはかなり便利になってて、Yの値の軸をleft axis、right axisで切り替えれる為、実数はleft axis、割合はleft axis等にすると変化を可視化しやすくなる。

もう一つグラフ作成時のポイントとしては、paid/organicで分けたいけどtotalも見たい為、paid/organicは棒グラフをスタックさせ、totalを線グラフにして、マウスオーバーすると両方数値が見れるようにしている部分。これは他の数種類の実数+合計数値+率系のグラフにも応用できる考え方だと思う。

```sql
with t as (
  select 
    date_trunc('day', are.registered_at at time zone 'JST') dt
    , case
         when are.network_name in ('Organic', 'other-organic-network') then 'organic'
         else 'paid'
       end as category
    , count(*) cnt
    , sum(count(*)) over (partition by date_trunc('day', are.registered_at at time zone 'JST')) total
  from adjust_registration_event are
  where are.registered_at >= date_trunc('day', current_timestamp + '-3 months')
  group by dt, category
)
select
  paid.dt
  , coalesce(paid.cnt, 0) paid_cnt
  , coalesce(organic.cnt, 0) organic_cnt
  , coalesce(organic.cnt, 0) + coalesce(paid.cnt, 0) total_cnt
  , organic.organic_rate
from (
  select
    t.dt
    , t.cnt cnt
  from t
  where t.category = 'paid'
) paid
left join (
  select
    t.dt
    , t.cnt cnt
    , cast(t.cnt as float) / cast(t.total as float) organic_rate
  from t
  where t.category = 'organic'
) organic
on organic.dt = paid.dt
order by paid.dt
;
```

日次の獲得数推移はどうなっているのか、Paid獲得だけに寄ってしまっていないか、CPIメニューの動画広告が大量に配信された際にOrganicにどのように影響があるのか等、ざっくり把握するのに便利なグラフとなっている。また再度になるけどこのleft axis→実数、right axis→割合、グループは積み上げ棒グラフして合計はline chartという形式はかなり汎用的に利用できるのでお勧めです(日別登録数推移と当日にアクションAをした割合推移とか)。

### 当月ネットワーク別Paid CPA、Blended CPA

これはPaid Cost/Paid獲得出せばいけるし、Blendedは分母にOrganic獲得を入れれば出せる。ネットワーク別日別Paid CPAもサクッと出せるので一旦ここではSQL省く。

## まとめ

全然最後まで到達できなかった。ちょっとデータ準備に時間使いすぎました。。一旦こういうのってみんなどうやってんのかずっと知りたかったので、まずはこっちから開示するスタイルで出します！続編は反響あれば書きます！

## 募集

株式会社カンムでは上記のように広告の運用を媒体側の管理画面に閉じず、自社内のデータと突合しながら最適な配信を目指して日々改善を繰り返しております。もしご興味がある方がいらっしゃれば是非お声掛けください！

- [オンライン広告担当](https://kanmu.co.jp/jobs/ad-manager/)
- [マーケティングマネージャー](https://kanmu.co.jp/jobs/ad-manager/)

また、マーケティング以外の領域でも積極的に採用中です！

- [その他職種](https://kanmu.co.jp/jobs/)

<script async class="speakerdeck-embed" data-id="78911418a1df4b1a9a60f938a57e9951" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>
