---
title: "【Android/Kotlin】JUnit4 によるユニットテストを試してみた"
emoji: "🐡"
type: "tech"
topics: ["Android", "Kotlin", "ユニットテスト", "JUnit"]
published: true
---

# 目的

Android 開発を進めていく上で、ビジネスロジックのユニットテストを行いたい。
公式ドキュメントで紹介されていた JUnit4 を試してみたので、導入から実行までを紹介します。

# 使用ライブラリ

AndroidStudio を通じてプロジェクトを新規作成すると、ユニットテスト用のライブラリは最初から入っています。

```
dependencies {
    // ローカル単体テスト用 ※デフォルトで入っている
    testImplementation 'junit:junit:4.13.2'

    // インストルメンテーションテスト ※デフォルトで入っている
    // テストするアプリの Context などの情報を参照し、テスト対象のアプリをテストコードで制御することができる
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'

    // UIテスト用のライブラリ ※デフォルトで入っている
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'

    // Truth is a library for performing assertions in tests
    // https://truth.dev/
    testImplementation "com.google.truth:truth:1.1.3"
}
```

今回の例では、追加で `Truth` というライブラリを追加しています。
`Truth` は、テスト結果を確認する上で分かりやすいメソッドが豊富に用意されているとのことです。

# テストしたいメソッド

今回テストしたいメソッドは、ターゲットとなる文字列の日付が対象の期間内であるかを判定する `isDateInRange` になります。

この判定結果に応じて、View 側でユーザーにメッセージを表示/非表示することを行います。

今回の例では、ターゲットとなる日付が 10 日から 20 日の間であれば True を返すメソッドとなります。

```Kotlin:MainViewModel
class MainViewModel : ViewModel() {
    private val _isVisible = MutableLiveData<Boolean>()
    val isVisible: MutableLiveData<Boolean>
        get() = _isVisible

    init {
        val targetDate = getTargetDate()
        _isVisible.postValue(isDateInRange(targetDate))
    }

    /**
     * 通常であればリポジトリから引っ張る
     */
    private fun getTargetDate(): String {
        // ここでは仮の日付を返します
        return "20220115"
    }

    /**
     * 日付が 10 日から 20 日の間であるかを判定する
     */
    fun isDateInRange(date: String): Boolean {
        if (date.length != DATE_EXPECTED_LENGTH) {
            return false
        }
        val day = date.substring(START_INDEX)
        if (day in STARTING_DAY..ENDING_DAY) {
            return true
        }
        return false
    }

    companion object {
        private const val DATE_EXPECTED_LENGTH = 8
        private const val STARTING_DAY = "10"
        private const val ENDING_DAY = "20"
        private const val START_INDEX = 6
    }
}
```

# ユニットテストクラスの作成方法

## 1. ユニットテスト作成したいクラス名の上で右クリックし、`generate` を押下

![](/images/android-unit-test-sample/android-unit-test-sample01.png)

## 2. `Test...`を押下

![](/images/android-unit-test-sample/android-unit-test-sample02.png)

## 3. 項目の設定

以下のように設定して、`OK`を押下

`Testing Library`：JUnit4
`Class Name`：~~~~~~Test

![](/images/android-unit-test-sample/android-unit-test-sample03.png)

## 4. `../app/src/test/...` を選択して、`OK` を押下

![](/images/android-unit-test-sample/android-unit-test-sample04.png)

## 5. `../app/src/test/...`配下に、ユニットテストクラスが作成されていることを確認

![](/images/android-unit-test-sample/android-unit-test-sample05.png)

# ユニットテストコード

ユニットテストコードの書き方は非常にシンプルです。

ユニットテストとするメソッド名の前に `@Test` アノテーションを追加し、確認したい結果を `assertThat` に渡してあげるだけです。

以下の例では、`isDateInRange` に 15 日の日付を渡し、返却値が `result` に代入されます。
その `result` を `assertThat` に渡し、それが `isTrue` なのかを確認します。
もし `result` が False だった場合には、テストが落ちる形となります。

```Kotlin
@Test
fun `Return true when date is 15th`() {
    val result = viewModel.isDateInRange("20220115")
    assertThat(result).isTrue()
}
```

Kotlin のユニットテストコードの良いところは、メソッド名をバッククォートで囲むことで通常の文章として定義することができます。

今回の例では英文で記載していますが、日本語などの他の言語でも書いていくことができます。

このおかげで、テスト名が書きやすい、読みやすい、理解しやすい内容にできると思います。

以下は、今回の例で用意したユニットテストのパターンとなります。

```Kotlin:MainViewModelTest
class MainViewModelTest {

    private lateinit var viewModel: MainViewModel

    // Test アノテーションメソッドが実行される前に、Before アノテーションメソッドが呼ばれる
    @Before
    fun setUp() {
        // MainViewModel のインスタンス化
        viewModel = MainViewModel()
    }

    @Test
    fun `Return true when date is 15th`() {
        // 返却値として True が返る値を引数として渡す
        val result = viewModel.isDateInRange("20220115")
        // 返却値の結果が True かを確認
        assertThat(result).isTrue()
    }

    @Test
    fun `Return false when date is 5th`() {
        // 返却値として False が返る値を引数として渡す
        val result = viewModel.isDateInRange("20220105")
        // 返却値の結果が False かを確認
        assertThat(result).isFalse()
    }

    @Test
    fun `Return true when date is 10th`() {
        val result = viewModel.isDateInRange("20220110")
        assertThat(result).isTrue()
    }

    @Test
    fun `Return true when date is 20th`() {
        val result = viewModel.isDateInRange("20220120")
        assertThat(result).isTrue()
    }

    @Test
    fun `Return false when date is 9th`() {
        val result = viewModel.isDateInRange("20220109")
        assertThat(result).isFalse()
    }

    @Test
    fun `Return false when date is 21th`() {
        val result = viewModel.isDateInRange("20220121")
        assertThat(result).isFalse()
    }
}
```

# テスト実行する

[公式ドキュメント](https://developer.android.com/training/testing/unit-testing/local-unit-tests?hl=ja#run)に書いてある通り、以下のようにテスト実行します。

> 単一のテストを実行するには、[Project] ウィンドウを開き、対象のテストを右クリックして [Run] をクリックします。
> クラス内のすべてのメソッドをテストするには、テストファイル内の対象のクラスまたはメソッドを右クリックして [Run] をクリックします。

## テスト成功時

成功時には、左側の `Test Results` に緑色のチェックマークが付きます。

![](/images/android-unit-test-sample/android-unit-test-sample06.png)

## テスト失敗時

失敗時には、左側の `Test Results` に黄色のバツマークが付きます。

![](/images/android-unit-test-sample/android-unit-test-sample07.png)

テスト失敗したエラーコードは以下のようになります。

```
String index out of range: -6
java.lang.StringIndexOutOfBoundsException: String index out of range: -6
	at java.base/java.lang.String.substring(String.java:1841)
	at com.example.unittestsample.ui.main.MainViewModel.isDateInRange(MainViewModel.kt:31)
	at com.example.unittestsample.ui.main.MainViewModelTest.Return false when date is empty string(MainViewModelTest.kt:58)
```

今回の例では、テストに落ちるように想定外の入力値を２つ用意し、1 つを実行しました。

1. 渡される日付の文字列が空文字だったとき
2. 渡される日付の文字列が想定しているフォーマットではなかったとき

```
@Test
fun `Return false when date is empty string`() {
    val result = viewModel.isDateInRange("")
    assertThat(result).isFalse()
}

@Test
fun `Return false when date's format is unexpected`() {
    val result = viewModel.isDateInRange("2022/01/15")
    assertThat(result).isFalse()
}
```

こうした想定外の入力値を考慮して、`isDateInRange` メソッドに例外処理を追記します。

```Kotlin
fun isDateInRange(date: String): Boolean {
    // 例外処理
    if (date.length != DATE_EXPECTED_LENGTH) {
        return false
    }
    val day = date.substring(START_INDEX)
    if (day in STARTING_DAY..ENDING_DAY) {
        return true
    }
    return false
}

companion object {
    private const val DATE_EXPECTED_LENGTH = 8
    private const val STARTING_DAY = "10"
    private const val ENDING_DAY = "20"
    private const val START_INDEX = 6
}
```

この処理を追加して、再度テストを実行します。

![](/images/android-unit-test-sample/android-unit-test-sample08.png)

落ちたテストをクリアし、すべて OK となりました。

## 補足

今回のサンプルコードでテスト実行すると、以下のようなエラーが発生しました。

```
Method getMainLooper in android.os.Looper not mocked.
```

発生箇所は、`LiveData` の値を更新する `postValue` の箇所でした。

こちらの問題は、[こちらの公式ドキュメント](https://developer.android.com/training/testing/unit-testing/local-unit-tests?hl=ja#error-not-mocked)に載っている対処方法で解決しました。

# サンプルコード

https://github.com/Kotaro666-dev/androidDevelopment/tree/main/mockups/UnitTestSample

# 参考文献

- [アプリをテストする](https://developer.android.com/studio/test?hl=ja)
- [ローカル単体テストを作成する](https://developer.android.com/training/testing/unit-testing/local-unit-tests?hl=ja)
- [Truth - Fluent assertions for Java and Android](https://truth.dev/)
