---
date: "2018-08-11T00:00:00Z"
title: 'net/httpで作るGo APIサーバー #6'
---


この一連の記事では `net/http` を主軸に据え、取替可能な部品となるライブラリを利用してAPIサーバーを作成する方法を紹介する。

- [net/httpで作るGo APIサーバー #1](https://akirachiku.com/2017/04/01/go-net-http-api-server-1.html)
- [net/httpで作るGo APIサーバー #2](https://akirachiku.com/2017/04/02/go-net-http-api-server-2.html)
- [net/httpで作るGo APIサーバー #3](https://akirachiku.com/2017/04/02/go-net-http-api-server-3.html)
- [net/httpで作るGo APIサーバー #4](https://akirachiku.com/2017/04/08/go-net-http-api-server-4.html)
- [net/httpで作るGo APIサーバー #5](https://akirachiku.com/2018/06/23/go-net-http-api-server-5.html)


上記5つの記事で `net/http` 単体でも小さいライブラリを利用しながらある程度の機能を満たせる形でAPIサーバーを構築できる事を示した。今回はエラーハンドリングについて書く。サーバーサイドのエラーハンドリングを中心に書くが、この領域はフロントエンドとも密に連携していく必要があるため、その辺りの仕組みをどうするかに関しても言及していく。


### "エラー"と呼ばれるものの種類と扱い方

Goで"エラー"という言葉が表すものを恣意的に二種類に分類した。それぞれの定義と簡易的な扱い方を以下に書く。

#### ハンドリングの余地の無いエラー

- 例: 存在しないスライスのインデックスにアクセスした、等
- アプリケーション側ではこれ以上どうしようもない、という状態になるエラー
- このタイプのエラーは `panic` を利用する

[Code Review Comment](https://github.com/golang/go/wiki/CodeReviewComments#dont-panic) にもあるようにアプリケーションコードの中で利用することはあまり無いかなという印象。たまにGoの `error` を扱うのがダルいので `error` を返さずに `panic` で終了！みたいなコードをライブラリ内で見ることがあるけど、そのような使い方をしてしまうとライブラリ利用側でそのエラーが発生した際にどのような挙動をするべきか決めるという自由を奪い強制的に終了することになるので、少なくとも利用には慎重になるのが大切かなと思う。(もちろん `error` を返すよりも `panic` で落とすケースの方が適切な場合もあると思うのであくまで `panic` 使う前に慎重になる、という意味)


#### ハンドリングの余地の有るエラー

- 例: 存在しないユーザーIDがリクエストされてきた、等
- アプリケーション側で発生したエラーによって挙動、メッセージを変更したいエラー
- このタイプのエラーは `error` を利用し適切なハンドリングをする


Goで `error` はビルトインのインターフェース型の一つ。以下の形を満たせばどのような構造体でも関数でも `error` として扱える。

```go
type error interface {
    Error() string
}
```

また標準ライブラリの `errors`　パッケージの中を覗いてみるとこういうコードになっている。[src/errors/errors.go](https://golang.org/src/errors/errors.go)。非常にシンプルだが拡張性のある形になっている。ただし、それが故に以下2点に関しては少し工夫が必要になる。

- `error` に付加情報を持たせる
    * 関数の深いところから `error` が返ってくる場合に `error` インターフェースを満たしながら各階層でコンテクストを付加して呼び出し元に渡したい場合がある
    * これは処理がGoの中で閉じるならよほどのことが無い限り [github.com/pkg/errors](https://github.com/pkg/errors) を利用すれば良いと思う
        * `errors.Wrap` もしくは `errors.Wrapf` で呼び出した関数から上がってきた `error` に何某かの付加情報を付与して返す形になる
        * これをやることで `no such file or directory` だけのエラーが返ってくる事がなくなり、どこで何の関数が失敗したのかわかりやすくなる
        * ただしフロントエンドに返すエラーとなると話が違うので後ほど詳細に書く
- `error` に種類を持たせる
    * `error` が呼び出した関数から返ってきた場合、具体的に何のエラーかによって呼び出し元で処理を分けたい場合がある
    * 実装方法は大きく分けて3つあり、それぞれにメリット/デメリットがある
        * この記事が最高に参考になる
        * [Don't just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully) 

「 `error` に付加情報を持たせる」に関してはあまり大きな論点が無いのでそのままにするとして、「 `error` に種類を持たせる」の部分はもう少し詳細に書く。

### どのようにerrorに種類を持たせるか

原則 [Don't just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully) を読めばいけるんだけどざっくりと要約すると以下になる。

- daveの結論
    * Opaque Errorを使うとパッケージ間の依存を極限まで減らせて便利
- エラーに種類を持たせ、種類によって呼び出し元の処理を変える方法は大きく3つあり、それぞれメリットとデメリットがある
- Sentinel Error
    * エラーを値としてパッケージ内に持つ (例: `io.EOF`)
    * 値なので `==` で比較して呼び出し元の処理を分ける
    * メリット
        * あんまりない
    * デメリット
        * エラーに追加で情報をもたせたりする事ができなくなる
            * `var EOF = errors.New("EOF")` で定義されているので
        * このエラー自体をパブリックなAPIとして他のパッケージが利用してしまうので変更が難しくなる


```go
if err == ErrSomething { … }
```


- Error Type
    * エラーを型を持つ構造体としてパッケージ内に持つ (例: `os.PathError`)
    * Type Assertionを使って呼び出し元の処理を分ける
    * メリット
        * 型なので属性を増やせば追加でエラーのメタデータを増やせる
            * `os.PathError` には `Op` , `Path` というメタデータが存在している
    * デメリット
        * このエラー自体をパブリックなAPIとして他のパッケージが利用してしまうので変更が難しくなる


```go
type MyError struct {
        Msg string
        File string
        Line int
}

func (e *MyError) Error() string { 
        return fmt.Sprintf("%s:%d: %s”, e.File, e.Line, e.Msg)
}

return &MyError{"Something happened", “server.go", 42}
```


```go
err := something()
switch err := err.(type) {
case nil:
        // call succeeded, nothing to do
case *MyError:
        fmt.Println(“error occurred on line:”, err.Line)
default:
// unknown error
}
```



- Opaque Error
    * 特定のインターフェースをパッケージ内に持つ (例: `net.Error`)
    * Type Assertionを使って呼び出し元の処理を分ける
    * メリット
        * インターフェースなのでパッケージ外に出る際付加情報を加える事も容易(インターフェースを満たす構造体をエラーとして利用)
        * 呼び出し側は特定の型を別パッケージからimportする必要無くエラー発生時に処理を分岐させれる
    * デメリット
        * あんまりない


```go
type temporary interface {
        Temporary() bool
}
 
// IsTemporary returns true if err is temporary.
func IsTemporary(err error) bool {
        te, ok := err.(temporary)
        return ok && te.Temporary()
}
```


こっからは自分の主観。Opaque Errorは確かに便利で標準ライブラリでも良く利用されているパターンだと思うんだけど、アプリケーション内部で使うにはそこまでパッケージの結合度合いを考えないで良いのではないかなぁという思いがある(特にアプリケーション構築初期には)。また、以下のようにエラーの扱いに困っている人とかもいたりして、確かに初見だと `XXXXError` みたいな型がパッケージのドキュメントにあった方が確かにわかりやすいよなぁという感想を持った。 [Go's net package doesn't have opaque errors, just undocumented ones](https://utcc.utoronto.ca/~cks/space/blog/programming/GoNetErrorsUndocumented)

なので自分はHTTP APIサーバーを作るというコンテクストでは Error Type 推し。ただ Opaque Error を使うと1関数から複数の Error Type が返ってくる想定で且つそれぞれの Error Type が `Tempolary()` と `Timeout()` を持っている場合、横断的なエラーハンドリングがしやすくはなるなとは思う。余談だけど、このタイプのメソッドは `Tempolary()` と `Timeout()` くらいしか見たことがなく自分の知見が狭いだけな可能性があるため、他にもこういう形のエラーハンドリングを使ってる場所があれば知りたい。


### HTTP APIサーバーという文脈でそれらをどのように扱うか

長かった。HTTP APIサーバーの中で `error` をどうやって使うかという話を書く。結論から書くと、カンムが提供するバンドルカードのAPIは「APIサーバーの中では原則 `error` をWrapしながら利用し、本当に必要な場合は Error Type を使う。フロントとのやり取りをするためにはそれ用の型( Error Type じゃない)を定義してやりとりする。エラーログの出力は `ServeHTTP` の中で実施する。」という形をとっている。一つづついきます。


#### APIサーバーの中では原則errorのみ利用し、本当に必要な場合は Error Type を使う

現状のパッケージ構成はいまだこの構成のまま。

<script async class="speakerdeck-embed" data-slide="10" data-id="9c6e284d996840b1a542b2beaf5ca30e" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

また、特にパッケージ毎にエラーを定義せずに原則 `error` を `errors.Wrap` して `model` -> `service` -> `handler` に伝播させていっている。Error Type を使わなかった理由は特に無くて、ただただアプリケーション作るだけならあんまし必要なくね、って思ったから。

例えば、`model.GetUserByID(tx sql.Tx, id int64) (*model.User, error)` という関数があったとして、その関数から `UserNotFoundError` や `UserSuspendedError` を返す事もできる。できるけど、`xxNotFound` 系のエラー型を作ってしまうとその他のエラー型もすべて作る事になってしまう(作らなくてもいいけど大きく一貫性を損なう)。それでも良いと言えば良いのだけど、メリットが少ないと感じている。それならば、`model.GetUserByID(tx sql.Tx, id int64) (*model.User, bool, error)` として戻り値2つ目を見つかったかどうかの真偽値にし、呼び出し側で処理を分岐するのが楽かなと思う。また、「ユーザーのステータスが `suspended` であるかどうか」というのは、取得されたユーザーデータを元に呼び出し元で判定し、適切なエラーをフロントエンドに返す、という形にすれば不要になる。この関数 `model.GetUserByID` の役割にもよるが、どうしても呼び出し元で複数のエラーハンドリングをしないといけない場合、関数を分割することを検討するなどし、現状はなるべく Error Type を作らずに分岐可能な情報を返すような方針で設計している。

ただし、特に外部APIにアクセスするGoのクライアントライブラリに関しては返ってくるエラーに様々なパターンがあり且つそれらのパターン別にフロントエンドに返すメッセージを分けたいことが多い為、独自の Error Type を定義して利用することがある。もちろん失敗した事だけわかれば良い外部APIであれば error をそのまま返す形になっている。


#### フロントとのやり取りをするためにはそれ用の型( Error Type じゃない)を定義してやりとりする

フロントエンド(アプリ/Web)とやりとりするデータはそれ用の型を作って利用している。これは `error` interface を満たす形ではなく、その他のレスポンスと同じように普通の struct になっている。これは完全にエンドユーザーとのコミュニケーションに利用する用途なので、エラーログ等には出すことは想定していない。多少変更しているが、現状以下。

```go
// Error struct for error resource
type Error struct {
	Status     int64  `json:"status"`
	Type       string `json:"type"`
	Code   string `json:"code,omitempty"`
	Errors []struct {
		Code       string `json:"code,omitempty"`
		Detail     string `json:"detail,omitempty"`
		Field      string `json:"field,omitempty"`
		UserDetail string `json:"user_detail,omitempty"`
	} `json:"errors,omitempty"`
	UserDetail string `json:"user_detail,omitempty"`
}
```

- `Status` : HTTP Status Codeを入れる
- `Type` : どの領域のエラーなのかを入れる (e.g. `card_activation`)
- `Code` : 該当領域の中でどのようなエラーなのかを入れる (e.g. `invalid_activation_code`)
- `Errors` : ユーザーインプットにエラーがあった場合に利用する
- `UserDetail` : 緊急脱出用ユーザー向けエラーメッセージ

フロント側は `Type` と `Code` を見て、なぜ処理が成功しなかったのか、どういう行動を取ればエラーを回避できるのか、等をエンドユーザーに伝えていく。この部分に関してはバックエンドとフロントエンドががっつり時間を取って話し合い、どういうケースが存在するのか、どのケースならユーザーに別のアクションをサジェストできるのか、等の認識を合わせていく必要がある。

正直バンドルカードリリース時に自分は不要だと思っていたが圧倒的に役に立っているのは `UserDetail` というエラーメッセージだ。フロントではこのフィールドに値が入っている場合、その文言をそのまま表示するようになっている(moqadaさん本当にありがとうございます.....)。アプリを開発しているとその仕組み上、修正して即時デプロイという手を取れない。どうしてもこのケースはユーザーに伝えたい、またはバグが発生している、となった場合この `UserDetail` を利用しバックエンドを新規デプロイする事によりユーザーに適切なメッセージを提供できることになる。

あくまでも `UserDetail` は緊急脱出用、その場しのぎ用であり本来的には各ケースをしっかり議論することが肝要ではあるが、度々このフィールドに命を救われている。


#### エラーログの出力はServeHTTPの中で実施する

`ServeHTTP` と handler の関連性についての詳細は下記を参照。


> 気軽に使いたいだけならhttp.HandlerFuncのシグネチャに合う関数func(http.ResponseWriter, *http.Request)を書いてサクッと使える。もし実際の処理の前後に何か処理を挟みたい、処理を共通化したい、そもそもfunc(http.ResponseWriter, *http.Request)の空returnがいまいち好きになれない、等、ということになればServerHTTP(http.ResponseWriter, *http.Request)をメソッドに持つ型を作り、それに処理をする独自シグネチャを持つ関数を渡してURLに紐付けるのが良いのではないか。
> [net/httpで作るGo APIサーバー #2](https://akirachiku.com/post/2017-04-02-go-net-http-api-server-2/)

loggerに関しては以下の記事を参照。

- [net/httpで作るGo APIサーバー #5](https://akirachiku.com/post/2018-06-23-go-net-http-api-server-5/)


エラーログ出力に関しては1箇所にまとめ、しかもアプリケーションコードを書いている際は `error` の取り回しさえ気をつけていれば問題無いようにしたかった。handlerのシグネチャを `func(http.ResponseWriter, *http.Request) (int, interface{}, error)` にして戻り値の `int` をHTTP Status Codeとして利用、`interface{}` をユーザーに返すレスポンス、`error` を開発者が見るエラーログに出力する情報と定義している。


```go

// ApiHandler handler for api
type ApiHandler struct {
	handler func(http.ResponseWriter, *http.Request) (int, interface{}, error)
}

// ServeHTTPC for api
func (h ApiHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	encoder := json.NewEncoder(w)
	logger := xlog.FromRequest(r)

	status, res, err := h.handler(w, r)
	switch {
	// 204 でbodyにWriteすると以下のエラーが発生するので 204 の場合はwriteしない
	// http: request method or response status code does not allow body
	case status == http.StatusNoContent:
		return
	// 300系はリダイレクト。そのさいは interface{} は redirect先URL で返してもらうようにする。
	case status == http.StatusFound || status == http.StatusMovedPermanently:
		redirectURL, ok := res.(string)
		if !ok {
			logger.Error("failed to convert res to redirectURL")
			w.WriteHeader(http.StatusInternalServerError)
			encoder.Encode(res)
			return
		}
		http.Redirect(w, r, redirectURL, status)
		return
	case status >= http.StatusBadRequest && status < http.StatusInternalServerError:
		logger.Warn(err)
	case status >= http.StatusInternalServerError:
		logger.Error(err)
	}

	w.WriteHeader(status)
	if err := encoder.Encode(res); err != nil {
		logger.Error(err)
		w.WriteHeader(http.StatusInternalServerError)
		encoder.Encode(errorResponseUnknown)
		return
	}
	return
}
```

400系は全てwarning log、500系はerror logとして扱っている。この辺りのエラーを運用でどうやって整備しているのかの話はまた別記事で書きたいのでここでは深入りはしない。


### まとめ

エラーハンドリングは本当にむずい。


### 参考資料

- [Failure is your Domain](https://middlemost.com/failure-is-your-domain/)
- [Don't just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
- [Error handling and Go](https://blog.golang.org/error-handling-and-go)
- [Golangのエラー処理とpkg/errors](https://deeeet.com/writing/2016/04/25/go-pkg-errors/)
- [Error handling in Upspin](https://commandcenter.blogspot.com/2017/12/error-handling-in-upspin.html)
