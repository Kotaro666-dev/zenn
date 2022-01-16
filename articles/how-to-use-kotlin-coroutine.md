---
title: "コルーチンの基本的な使い方（並行処理/並列処理）"
emoji: "🎉"
type: "tech"
topics: ["Kotlin", "Android", "Coroutine", "コルーチン"]
published: false
---

# 目的

会社業務のプロジェクト内で、HTTP 通信による API リクエストをする実装作業がありました。

その際に、Kotlin のコルーチンを使って非同期処理の実装を行ったが、時間がないためにコルーチンの基本的な使い方をちゃんと理解しないまま実装を進めてました。

その後、コルーチンの基礎をしっかりと勉強しようとしていた際に、以下の記事に出会いました。

[Coroutines on Android (part I): Getting the background](https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb#)

恥ずかしながら、この記事を通じて自分の理解が本当に酷かったと反省し、サンプルプロジェクトを作って勉強してみました。

以下では、自己学習用に作成したサンプルプロジェクトを通じて学んだ、並行処理と並列処理によるコルーチンを使った API リクエストの具体的な方法をまとめていきます。

# これからコルーチンを学ぼうと思う方へ

これからコルーチンを学ぼうと思う方は、以下に紹介する記事と Youtube を見ることをオススメします。

- [Coroutines on Android (part I): Getting the background](https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb#)
- [[YouTube]Understand Kotlin Coroutines on Android (Google I/O'19)](https://www.youtube.com/watch?v=BOHK_w09pVA)

上に挙げた資料のおかげで、コルーチンの基本的な概念を学ぶことができました。

# 私の間違った解釈

まずはじめに私の間違った解釈を話します。

私はこれまでマルチスレッド対応言語で開発したことがなかったため、複数のスレッドを使った実行処理もなんとなくしか理解していませんでした。

私の解釈はこんな感じでした。

> 1. `CoroutineScope.launch` で囲った処理はメインスレッドではなく、バックグラウンドスレッドで実行される
> 2. `suspend` 修飾子をつけた自作関数では、`WithContext` を使ってバックグラウンドスレッドで使うことを命令する

# 正しい解釈

## 1. `CoroutineScope.launch` で囲った処理はメインスレッドではなく、バックグラウンドスレッドで実行される

上記で挙げた記事では以下のように書いてあります。

> Coroutines will run on the main thread, and suspend does not mean background.

コルーチンを作成する際に自ら明示しない場合、 **デフォルトではメインスレッドで処理が動いていきます。**

```Kotlin
fun example() {
    viewModelScope.launch(Dispatchers.IO) {
        // IOバックグラウンドスレッドで実行する処理
    }
}
```

## 2. `suspend` 修飾子をつけた自作関数では、`WithContext` を使ってバックグラウンドスレッドで使うことを命令する

コルーチン内の処理で **`WithContext` を使って処理するスレッドを変えます。**

例えば、API リクエストのような非同期処理には IO バックグラウンドスレッド、計算量のかかる重い処理には Default バックグラウンドスレッドといった形です。

参考資料：[CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/)

また、関数に `suspend` 修飾子をつけることで、その関数を一時的に中断させ、UI 側の仕事もしているメインスレッドを妨害しない。

そして、`WithContext` と組み合わせることで、一時的に中断した処理を別のスレッドで実行させることで非同期処理を実現している。

```Kotlin
suspend fun example() {
    doTaskOnMainThread()           // メインスレッドで実行
    withContext(Dispatchers.IO) {  // IOバックグラウンドスレッドで実行
        doTaskOnBackGroundThread() // IOバックグラウンドスレッドで実行
    }                              // IOバックグラウンドスレッドで実行
    doTaskOnMainThread()           // メインスレッドで実行
}
```

# サンプルプロジェクト

今回用意したサンプルプロジェクトは 3 パターンを用意しました。

1. 並行処理
2. async/await を使った並列処理
3. launch/join を使った並列処理

## 1. 並行処理

3 つの API リクエストを **並行処理で順番に実行していき**、結果を UI 画面に表示していくものです。

今回は実際に API リクエストをするわけではなく、各リクエスト実行時に 1 秒間の遅延を発生させます。そのため、ボタンを押下してから 3 秒後に処理が完了します。

![](/images/how-to-use-kotlin-coroutine/concurrent-sample-video.gif)

順番に見ていくと、まず ViewModel のライフサイクルと結びついたコルーチンスコープ [`viewModelScope`](https://developer.android.com/topic/libraries/architecture/coroutines#viewmodelscope)を作成します。

ライフサイクルと結びついているため、作成したコルーチンスコープは ViewModel が破棄されるタイミングで自動的にキャンセルされます。

:::message
Activity や Fragment のライフサイクルと結びついたコルーチンスコープ [`lifeCycleScope`](https://developer.android.com/topic/libraries/architecture/coroutines#lifecyclescope)があります。
:::

そして、このコルーチンスコープ内の処理は順番に逐次実行されていきます。

```Kotlin:MainVieweModel
fun requestApi() {
    // 1. コルーチンスコープを作成し、中身の処理がメインスレッドで逐次実行されていく
    viewModelScope.launch {
        _isLoading.value = true  // 2. ローディング中状態にする
        fetchApi1()              // 3. API1 へリクエスト
        fetchApi2()              // 4. API2 へリクエスト
        fetchApi3()              // 5. API3 へリクエスト
        _isLoading.value = false // 6. ローディング完了状態にする
    }
}
```

次に、3 つの API リクエストを行うメソッド `fetchApiX` の中を見ていきましょう。

このメソッド内では、推奨されている IO バックグラウンドスレッドに `WithContext` で切り替えて、1 秒時間のかかるリクエストを実行し、結果を LiveData に反映します。

```Kotlin:MainVieweModel
private suspend fun fetchApi1() {
    // 1. IOバックグラウンドスレッドに切り替える
    withContext(Dispatchers.IO) {
        delay(1_000)                      // 2. 1 秒間遅延させる（IOバックグラウンドスレッドで実行）
        _dataFromApi1.postValue("Kotlin") // 3. メインスレッドではないため、postValue で値を更新（IOバックグラウンドスレッドで実行）
    }
}
```

並行処理の実行処理フローイメージは以下のような形です。

![](/images/how-to-use-kotlin-coroutine/concurrent-process-image.png)

## 2. async/await を使った並列処理

次に、async と await を使った並列処理を見ていきましょう。

並行処理では順番に API リクエストをしていきましたが、 **並列処理ではすべての API 処理を同時に実行します。**

そのため、並列処理ではすべての API リクエスト完了まで 1 秒間となります。

![](/images/how-to-use-kotlin-coroutine/parallel-sample-video.gif)

```Kotlin:MainViewModel
fun requestApiWithAsyncAndAwait() {
    // 1. コルーチンスコープを作成し、中身の処理がメインスレッドで逐次実行されていく
    viewModelScope.launch {
        _isLoading.value = true     // 2. ローディング中状態にする
        val apiAsyncList = listOf(  // 3. async を使って新しいコルーチンを作成し、その中で各 API リクエストを実行する。
            async { fetchApi1() },  //    Deffered 型の返り値をリスト化する。
            async { fetchApi2() },
            async { fetchApi3() },
        )
        apiAsyncList.awaitAll()     // 4. 全ての API リクエストの実行完了を待つ
        _isLoading.value = false    // 5. ローディング完了状態にする
    }
}
```

並列処理の実行処理フローイメージは以下のような形です。

![](/images/how-to-use-kotlin-coroutine/parallel-process-image.png)

## 3. launch/join を使った並列処理

launch/join を使うと、async/await と同じ並列処理を実現できます。

`viewModelScope` のコルーチンスコープ内に、`launch` を使って新しいコルーチンスコープを作成し、その中で API リクエストを行います。

launch は Job 型の返り値を返し、Job 型のリスト `apiJobsList` の要素すべて完了するのを待ちます。

```Kotlin:MainViewModel
fun requestApiWithLaunchAndJoin() {
    // 1. コルーチンスコープを作成し、中身の処理がメインスレッドで逐次実行されていく
    viewModelScope.launch {
        _isLoading.value = true     // 2. ローディング中状態にする
        val apiJobsList = listOf(   // 3. lanuch を使って新しいコルーチンを作成し、その中で各 API リクエストを実行する。
            launch { fetchApi1() }, //    Job 型の返り値をリスト化する
            launch { fetchApi2() },
            launch { fetchApi3() },
        )
        apiJobsList.joinAll()       // 4. 全ての API リクエストの実行完了を待つ
        _isLoading.value = false    // 5. ローディング完了状態にする
    }
}
```

# サンプルコード

https://github.com/Kotaro666-dev/androidDevelopment/tree/main/mockups/KotlinCoroutineSample

# 参考資料

- [Coroutine context and dispatchers](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html)
- [Coroutines on Android (part I): Getting the background](https://medium.com/androiddevelopers/coroutines-on-android-part-i-getting-the-background-3e0e54d20bb#)
- [[YouTube]Understand Kotlin Coroutines on Android (Google I/O'19)](https://www.youtube.com/watch?v=BOHK_w09pVA)
- [Kotlin コルーチンでアプリのパフォーマンスを改善する](https://developer.android.com/kotlin/coroutines/coroutines-adv?hl=ja)
