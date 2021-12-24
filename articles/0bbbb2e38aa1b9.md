---
title: "CupertinoTabBarによるBottomNavigationBar機能搭載時に、pushNamedで出力されるエラーの解決方法"
emoji: "📑"
type: "tech"
topics: ["flutter", "dart", "メモ"]
published: true
---

# 出力されるエラー文

```
Could not find a generator for route RouteSettings("/settings_page", null) in the _CupertinoTabViewState. Generators for routes are searched for in the following order: 1. For the "/" route, the "builder" property, if non-null, is used. 2. Otherwise, the "routes" table is used, if it has an entry for the route. 3. Otherwise, onGenerateRoute is called. It should return a non-null value for any valid route not handled by "builder" and "routes". 4. Finally if all else fails onUnknownRoute is called. Unfortunately, onUnknownRoute was not set.
```

# 経緯 Flutter プロジェクトに、BottomNavigationBar が維持された状態で画面遷移が行われる実装をした。

![](https://storage.googleapis.com/zenn-user-upload/isswx35g5ik52uglohcjf8jnbmt2 =250x) 参考資料：[Flutter Case Study: Multiple Navigators with BottomNavigationBar](https://medium.com/coding-with-flutter/flutter-case-study-multiple-navigators-with-bottomnavigationbar-90eb6caa6dbf)

````dart
dart:main.dart

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(primarySwatch: Colors.blue,),
      home: TabBarPage(),
      onGenerateRoute: MyRouter.generateRoute,);
  }
}

class TabBarPage extends StatelessWidget {
  @override Widget build(BuildContext context) {
    return CupertinoTabScaffold(
      tabBar: CupertinoTabBar(items: const [
        BottomNavigationBarItem(
          icon: Icon(Icons.home), label: 'home',),
        BottomNavigationBarItem(
          icon: Icon(Icons.shopping_cart), label: 'shopping',),
        BottomNavigationBarItem(
          icon: Icon(Icons.payment), label: 'payment',),
        BottomNavigationBarItem(
          icon: Icon(Icons.settings), label: 'settings',),
      ],), tabBuilder: (BuildContext context, int index) {
      switch (index) {
        case 0:
          return CupertinoTabView(builder: (context) {
            return CupertinoPageScaffold(child: HomePage());
          },);
        case 1:
          return CupertinoTabView(builder: (context) {
            return CupertinoPageScaffold(child: ShoppingPage());
          },);
        case 2:
          return CupertinoTabView(builder: (context) {
            return CupertinoPageScaffold(child: PaymentPage());
          },);
        case 3:
          return CupertinoTabView(builder: (context) {
            return CupertinoPageScaffold(child: SettingsPage());
          },);
        default:
          return CupertinoTabView(builder: (context) {
            return CupertinoPageScaffold(child: HomePage());
          },);
      }
    },);
  }
}
```

CupertinoTabViewで作られるページ内で、下記のようにpusnNamedを使った画面遷移をしようとすると、上記のエラーが出力される。

```dart
class HomePage extends StatelessWidget {
  static const routeName = '/home_page';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('HomePage'),
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, SettingsPage.routeName);
              },
              child: Text('Go To SettingsPage'),
            ),
          ],
        ),
      ),
    );
  }
}

```

# 解決方法 CupertinoTabView の routes プロパティに、遷移する画面のルートを渡してあげる

```dart
final appRoutes = {
  HomePage.routeName: (_) => HomePage(),
  ShoppingPage.routeName: (_) => ShoppingPage(),
  PaymentPage.routeName: (_) => PaymentPage(),
  SettingsPage.routeName: (_) => SettingsPage(),
};

class TabBarPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return CupertinoTabScaffold(
      tabBar: CupertinoTabBar(
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.shopping_cart),
            label: 'shopping',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.payment),
            label: 'payment',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.settings),
            label: 'settings',
          ),
        ],
      ),
      tabBuilder: (BuildContext context, int index) {
        switch (index) {
          case 0:
            return CupertinoTabView(
              routes: appRoutes, <----------------------- ここ
              builder: (context) {
                return CupertinoPageScaffold(child: HomePage());
              },
            );
          case 1:
            return CupertinoTabView(
              routes: appRoutes, <----------------------- ここ
              builder: (context) {
                return CupertinoPageScaffold(child: ShoppingPage());
              },
            );
          case 2:
            return CupertinoTabView(
              routes: appRoutes, <----------------------- ここ
              builder: (context) {
                return CupertinoPageScaffold(child: PaymentPage());
              },
            );
          case 3:
            return CupertinoTabView(
              routes: appRoutes, <----------------------- ここ
              builder: (context) {
                return CupertinoPageScaffold(child: SettingsPage());
              },
            );
          default:
            return CupertinoTabView(
              routes: appRoutes, <----------------------- ここ
              builder: (context) {
                return CupertinoPageScaffold(child: HomePage());
              },
            );
        }
      },
    );
  }
}

```

Flutter公式ドキュメントの CupertinoTabView/routes プロパティには以下のように記載されている。

> When a named route is pushed with Navigator.pushNamed inside this tab view, the route name is looked up in this map. If the name is present, the associated WidgetBuilder is used to construct a CupertinoPageRoute that performs an appropriate transition to the new route. [routes property](https://api.flutter.dev/flutter/cupertino/CupertinoTabView/routes.html)


参考資料：[Trying to add router with bottom navigation bar in CupertinoApp](https://stackoverflow.com/questions/62554769/trying-to-add-router-with-bottom-navigation-bar-in-cupertinoapp)
````
