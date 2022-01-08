---
title: "フラグメントがViewPager内で画面表示されていることを検知し、イベント処理を行う方法"
emoji: "🌟"
type: "tech"
topics: ["Android", "Kotlin"]
published: false
---

# 目的

`BottomNavigationView` と `ViewPager2` を使って、ボトムタブバーを表示すると同時に、タブ遷移ができる実装方法があります。

この時に、ボトムタブバーの各アイテムのフラグメントが表示された際に、何かしらのイベントを行いたい場合どうするべきか。

# 以前の方法

調べていくと、日本語でも英語でも方法が共有されています。

英語：[How to determine when Fragment becomes visible in ViewPager](https://stackoverflow.com/questions/10024739/how-to-determine-when-fragment-becomes-visible-in-viewpager)
日本語：[Fragment で自身の Pager 内での表示状態を監視する](https://qiita.com/sakuna63/items/653452eb48029d53d44f)

両方の記事で紹介されているのは、`setUserVisibleHint` を使った方法です。

```Java
public class MyFragment extends Fragment {
    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (isVisibleToUser) {
        }
        else {
        }
    }
}
```

ただし、 **こちらのメソッドは 2022 年 1 月現在では非推奨のものとなりました。**

> public void setUserVisibleHint(boolean isVisibleToUser)
> This method is deprecated.
> https://developer.android.com/reference/androidx/fragment/app/Fragment.html#setUserVisibleHint(boolean)

# 代替方法

**`setMenuVisibility` を使います。**

```Kotlin
override fun setMenuVisibility(menuVisible: Boolean) {
        super.setMenuVisibility(menuVisible)
        if (menuVisible) {
            // Do something
        }
    }
```

# 作ったサンプルアプリ

今回作ったサンプルアプリは、タブ遷移が行われ、各ボトムタブバーを生成するフラグメントが表示されるごとにダイアログを表示するものとなります。

![](/images/sample_gif.gif)

# コード例

## XML レイアウト

main の XML レイアウトファイルにて、ViewPager2 と BottomNavigationView を設置します。

```Xml: activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/bottom_navigation_view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="horizontal"
        app:layout_constraintBottom_toTopOf="@+id/bottom_navigation"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?android:attr/windowBackground"
        app:labelVisibilityMode="labeled"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:menu="@menu/bottom_navigation_menu" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## Acitivity

この Activity ファイル内で、viewPager2 と BottomNavigationView を連携させて、ボトムタブバーのタブ遷移を実現しています。
今回の例では、4 つのボトムタブバーアイテムがある形となります。

```Kotlin: MainActivity.kt
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.bottomNavigationViewPager.adapter = BottomNavigationPagerAdapter(this)
        binding.bottomNavigationViewPager.isUserInputEnabled = false

        binding.bottomNavigation.setOnItemSelectedListener {
            val currentItem = getCurrentItem(it.itemId)
            binding.bottomNavigationViewPager.setCurrentItem(currentItem, true)
            return@setOnItemSelectedListener true
        }
    }

    private fun getCurrentItem(itemId: Int): Int {
        return when (itemId) {
            R.id.home_tab -> 0
            R.id.register_tab -> 1
            R.id.view_tab -> 2
            R.id.my_page_tab -> 3
            else -> 0
        }
    }
}
```

## Fragment

例として、4 つのボトムタブバーのアイテムのフラグメントのうち、MyPageFragment を例に取ります。

`setMenuVisibility` メソッドをオーバーライドし、`menuVisible` が真となった場合に、ダイアログを表示する簡単な実装例となります。

```Kotlin:MyPageFragment.kt
class MyPageFragment : Fragment() {

    companion object {
        private const val TITLE_DIALOG = "マイページ画面"
        private const val MESSAGE_DIALOG = "マイページ画面が表示されています。"
        private const val TAG_DIALOG = TITLE_DIALOG
    }

    private lateinit var binding: FragmentMyPageBinding

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentMyPageBinding.inflate(layoutInflater, container, false)

        return binding.root
    }

    override fun setMenuVisibility(menuVisible: Boolean) {
        super.setMenuVisibility(menuVisible)
        if (menuVisible) {
            displayDialog()
        }
    }

    private fun displayDialog() {
        val fragmentManager = this.requireActivity().supportFragmentManager
        CustomDialog(
            title = TITLE_DIALOG,
            message = MESSAGE_DIALOG
        ).show(fragmentManager, TAG_DIALOG)
    }
}
```

## カスタムダイアログ

参考までに、表示しているカスタムダイアログとなります。

:::message
`setCanceledOnTouchOutside` を `false` に設定することで、ダイアログの外側をタップしてもダイアログが閉じないように設定することができます。
:::

```Kotlin: CustomDialog.kt
class CustomDialog(
    private val title: String,
    private val message: String,
) : DialogFragment() {
    companion object {
        const val TITLE_POSITIVE_BUTTON = "OK"
    }

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val dialog = AlertDialog.Builder(requireContext())
            .setTitle(title)
            .setMessage(message)
            .setPositiveButton(TITLE_POSITIVE_BUTTON) { _, _ -> }
            .create()
        dialog.setCanceledOnTouchOutside(false)
        return dialog
    }
}
```

# 参考資料

- [setMenuVisibility](<https://developer.android.com/reference/androidx/fragment/app/Fragment.html#setMenuVisibility(boolean)>)
- [How to know when fragment actually visible in viewpager](https://stackoverflow.com/questions/48893014/how-to-know-when-fragment-actually-visible-in-viewpager)
- [Fragment で自身の Pager 内での表示状態を監視する](https://qiita.com/sakuna63/items/653452eb48029d53d44f)

# サンプルコード

https://github.com/Kotaro666-dev/androidDevelopment/tree/main/mockups/DetectIfFragmentIsVisible
