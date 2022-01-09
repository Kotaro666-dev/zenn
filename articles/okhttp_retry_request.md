---
title: "【Android/Kotlin】OkHttpでHTTPリクエスト失敗時にリトライする方法"
emoji: "📱"
type: "tech"
topics: ["Android", "Kotlin", "OkHttp"]
published: true
---

# 目的

OkHttp を使って、HTTP リクエスト（今回の例では GET メソッド）をして失敗したときに、複数回リトライしたい場合にはどうすればよいのか。

# 結論

OkHttpClient のインスタンス生成時に、こちら側で用意した Intercepters を登録し、HTTP リクエストとレスポンス間を監視する。
監視している中で、リクエストに失敗した場合に、リトライをするようにコントロールする。

# Interceptors とは？

OkHttp ライブラリを開発する Square の[公式ドキュメント](https://square.github.io/okhttp/interceptors/#interceptors)には、以下のようにまとめられています。

> Interceptors are a powerful mechanism that can monitor, rewrite, and retry calls.

今回の目的である HTTP リクエストの複数回リトライができそうなことが書いてありますね。

Interceptors でコントロールできる層は以下の 2 つがあります。

1. Application interceptors
2. Network Interceptors

![](/images/okhttp_retry_request/interceptors.png)

出典：https://square.github.io/okhttp/interceptors/

今回使用する層は、 **上記の画像の上部側 `Application interceptors` となります。**

その理由は、`Application interceptors` にはリトライして複数回の HTTP リクエストができることが書かれているからです。

> Permitted to retry and make multiple calls to Chain.proceed().

# 検証例

今回の検証例では、ランダムで猫の事実だけを教えてくれるフリーの API「[Cat Fact API](https://catfact.ninja/)」を使用します。

リクエストに成功すると、以下のような JSON 型の結果が返ってきます。

```JSON
{
  "fact": "Cat's urine glows under a black light.",
  "length": 38
}
```

こちらの API を Android プロジェクトに組み込んで、動作検証していきます。

HTTP リクエストに成功した場合には以下のような出力となります。

![](/images/okhttp_retry_request/request_success.png)

反対に、HTTP リクエストに失敗し、Interceptors を使って 3 回リトライしても失敗している場合には、以下のような出力となります。

![](/images/okhttp_retry_request/request_failure.png)

では、以下で実際のコード例をみていきましょう。

# コード例

## ApiClient クラス

ApiClient クラスでは、OkHttp でクライアントのインスタンスを作り、get メソッドでは同期リクエストを行います。

成功した場合には、レスポンスボディを返却し、失敗した場合には例外を投げます。
今回の例では、失敗して欲しいため、`url` に渡すエンドポイントを存在しないパス `/this_path_does_not_exist` に設定しています。

OkHttp のクライアント生成時に、`addIterceptor` メソッドを使用して、自作の Interceptor を追加します。

```Kotlin:ApiClient.kt
class ApiClient {
    companion object {
//        private const val URL = "https://catfact.ninja/fact"
        private const val URL = "https://catfact.ninja/this_path_does_not_exist"
    }

    private val client = OkHttpClient.Builder()
        .addInterceptor(RetryInterceptor()) // ここで自作の Interceptor を追加する
        .build()

    fun get(): String {
        val request = Request.Builder()
            .url(URL)
            .build()

        client.newCall(request).execute().use { response ->
            if (!response.isSuccessful) {
                throw IOException("Unexpected code $response")
            }

            return response.body?.string().orEmpty()
        }
    }
}
```

## Repository クラス

上記のコードで用意した HTTP リクエストの GET メソッドを、以下の Repository クラスで呼びます。
`try..catch..` 文を使って、リクエストに成功した場合には `Moshi` ライブラリで JSON 型をデコードし、必要な `fact` だけをリターン、リクエストに失敗して投げられた例外をキャッチしてコンソールにエラーを出力します。

```Kotlin:Repository.kt
class Repository {

    companion object {
        private const val REQUEST_FAILURE_MESSAGE = "Your request failed after 3 retries."
    }

    private val apiClient = ApiClient()

    suspend fun getText() = withContext(Dispatchers.IO) {
        try {
            val response = apiClient.get()
            return@withContext getCatFact(response)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return@withContext REQUEST_FAILURE_MESSAGE
    }

    private fun getCatFact(response: String) : String {
        val adapter = Moshi.Builder().build().adapter(CatFact::class.java)
        val data = adapter.fromJson(response)
        return data?.fact.orEmpty()
    }
}
```

## Interceptor クラス

こちらのクラスが今回の目的のキーとなる箇所です。
OkHttp のクライアント生成時に `addIterceptor` メソッドに渡していた RetryInterceptor を作っていきます。

`Interceptor` クラスを継承してクラスを作り、`intercept` メソッドをオーバーライドします。
あとは、この中に自分が行いたい処理をカスタマイズしていきます。

ロジックとしては非常にシンプルで、 **リクエストに失敗してリトライ回数が上限に達するまでを `while` 文で回し、`chain.proceed(request)` にてリクエストを実行する** だけです。

今回はテスト検証のため、リクエスト失敗時にしっかりとリトライが行われているのかを現在時刻とともにコンソールに出力するようにしています。

```Kotlin
class RetryInterceptor : Interceptor {
    companion object {
        private const val MAX_RETRY_COUNT = 3
        private const val FORMAT_PATTERN = "yyyy/MM/dd/HH:mm:ss.SSS"
    }

    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        // 最初のリクエストを行う
        var response = chain.proceed(request)

        var retryCount = 0
        // リクエストに失敗した場合には最大3回のリトライを行う
        while (!response.isSuccessful && retryCount < MAX_RETRY_COUNT) {
            println("リトライ回数: $retryCount（${getCurrentTime()}）")
            retryCount++

            // リトライ前に取得済みのレスポンスをクローズ
            response.close()
            // リクエストをリトライ
            response = chain.proceed(request)
        }
        return response
    }

    private fun getCurrentTime(): String {
        val date = Date(System.currentTimeMillis())
        val dateFormat = SimpleDateFormat(FORMAT_PATTERN, Locale.JAPANESE)
        return dateFormat.format(date)
    }
}
```

:::message alert
**【注意】**
リクエストをリトライする前に、取得済みのレスポンスをクローズしない場合、以下の例外が発生しリトライが行われません。

```
java.lang.IllegalStateException: cannot make a new request because the previous response is still open: please call response.close()
```

:::

# HTTP リクエスト失敗時のコンソール出力

以下のように、しっかりと 3 回リトライされていることが伺えます。
そして、リクエスト失敗して投げられた例外をキャッチして、レスポンスの中身が出力されています。

```
I/System.out: リトライ回数: 0（2022/01/08/14:29:48.898）
I/System.out: リトライ回数: 1（2022/01/08/14:29:49.142）
I/System.out: リトライ回数: 2（2022/01/08/14:29:49.353）
W/System.err: java.io.IOException: Unexpected code Response{protocol=h2, code=404, message=, url=https://catfact.ninja/this_path_does_not_exist}
W/System.err:     at com.example.okhttpretryrequestsample.repository.ApiClient.get(ApiClient.kt:28)
W/System.err:     at com.example.okhttpretryrequestsample.repository.Repository$getText$2.invokeSuspend(Repository.kt:18)
W/System.err:     at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
W/System.err:     at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)
W/System.err:     at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:571)
W/System.err:     at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:750)
W/System.err:     at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:678)
W/System.err:     at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:665)
```

# 環境

- Kotlin Version: 1.6.10
- compileSdk: 31

# 使用ライブラリ群のバージョン

```YAML:app/build.gradle
dependencies {

    // OkHttp3
    implementation("com.squareup.okhttp3:okhttp:4.9.1")

    // Kotlin Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.2")

    // Moshi
    implementation("com.squareup.moshi:moshi-kotlin:1.13.0")
    kapt 'com.squareup.moshi:moshi-kotlin-codegen:1.13.0'
}
```

# 参考資料

- [OkHttp Synchronous Get](https://square.github.io/okhttp/recipes/#synchronous-get-kt-java)
- [OkHttp Interceptors](https://square.github.io/okhttp/interceptors/#interceptors)
- [Parse a JSON Response With Moshi](https://square.github.io/okhttp/recipes/#parse-a-json-response-with-moshi-kt-java)
- [Android での Kotlin コルーチン](https://developer.android.com/kotlin/coroutines?hl=ja)

# サンプルコード

https://github.com/Kotaro666-dev/androidDevelopment/tree/main/mockups/OkHttpRetryRequestSample
