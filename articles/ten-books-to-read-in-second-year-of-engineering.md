---
title: "エンジニア 2 年目に読んで良かった本 10 選"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["本", "技術書", "2年目"]
published: true
published_at: 2022-12-14 00:00
---

:::message
こんにちは、[エヌ次元株式会社 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/nzigen) の 14 日目を担当する [Kotaro666-dev](https://twitter.com/Kotaro666_dev) です。
:::

私はエヌ次元でモバイルアプリ開発エンジニアをやっています。

# まえがき

本を紹介する前に、簡単な自己紹介をさせてもらいます。
なぜならば、以下で紹介する本は私の興味・関心や業務上のポジションに関連しているからです。

- 略歴
  - 2020 年初頭から エンジニア養成機関 [42Tokyo](https://42tokyo.jp/) でプログラミングを勉強し始めました
    - なぜプログラミングを勉強しようと思ったのかは、[こちらの Note ](https://note.com/kkamashi/n/nba8de97c3bf6)でまとめています
  - 2020 年 11 月からエヌ次元でモバイルアプリ開発者として働き始めました
- 業務内容
  - 主にモバイルアプリ開発のプロジェクトを担当しています
  - WEB アプリケーションも必要に応じて担当しています
  - プロジェクトによっては、チームリーダーをやっています
  - 開発以外では、メンター、ファシリテーター、エンジニア採用活動もやっています
- どんな人？
  - 低レイヤーから基礎的な知識を知りたいという興味があります
  - どんな技術でも、内部構造を理解したいという関心が強いです

# アジェンダ

- 技術書
  - 1. ソフトウェアアーキテクチャの基礎
  - 2. データ指向アプリケーションデザイン
  - 3. A Philosophy of Software Design
  - 4. System Design Interview – An insider's guide
  - 5. Go言語による並行処理
- ビジネス書
  - 1. サーバント・リーダー 「権力」ではない。「権威」を求めよ
  - 2. エンジニアリングマネージャーのしごと
  - 3. HIGH OUTPUT MANAGEMENT－人を育て、成果を最大にするマネジメント
  - 4. 解像度を上げる――曖昧な思考を明晰にする「深さ・広さ・構造・時間」の４視点と行動法
  - 5. LISTEN――知性豊かで創造力がある人になれる

※読んだ若い日の順に並べています。

# 技術書

## 1. ソフトウェアアーキテクチャの基礎

@[card](https://www.oreilly.co.jp/books/9784873119823/)

私がアーキテクチャに対する関心が出てきた時に出版されて、Twitter でも話題になっていたために買って読みました。

2022 年 6 月に Web アプリケーション開発のプロジェクトが始動する際に、アーキテクチャと技術選定を決める機会がありました。
本書籍は、この機会に大いに役立ちました。また、私がアーキテクチャに対する強い関心を持つきっかけともなりました。

冒頭から印象的な名言を残しています。

> ソフトウェアアーキテクチャはトレードオフがすべてだ。

> アーキテクトが、トレードオフではない何かを見出したと考えているなら、まだトレードオフを特定していないだけの可能性が高い。

> 「どうやって」よりも「なぜ」の方がずっと重要だ。

「トレードオフ」という言葉は、書籍内で度々言及されています。
「トレードオフ」は、私がソフトウェア開発の至るところで考えるようになった、2022 年私の流行語大賞となりました。

ちなみに、本書籍の著者たちが続編「[ソフトウェアアーキテクチャ・ハードパーツ](https://www.oreilly.co.jp/books/9784814400065/)」を出しました。

この続編は、架空の会社の物語とともにトレードオフ分析をしながらアーキテクチャの選択を実践的に学べる書籍です。

オライリージャパンの書籍プレゼントキャンペーンで運よく当たったことは今年一番嬉しい出来事でした。

@[tweet](https://twitter.com/Kotaro666_dev/status/1590218476232536065)

-----

## 2. データ指向アプリケーションデザイン

@[card](https://www.oreilly.co.jp/books/9784873118703/)

本年度に社内で開催した輪読会の課題図書に選出した書籍です。

私がこの書籍に興味を持ったきっかけは、「[SQL実践入門──高速でわかりやすいクエリの書き方](https://gihyo.jp/book/2015/978-4-7741-7301-6)」を通じてデータベース物理層に関心を持ったことが始まりで、データベースを学ぶ上で多くの人が推薦していたことです。

ものすごい分厚い書籍（600ページ強）で、かつデータベースや分散システムに知見のない私が一人だけで完読できる気がしませんでした。
そのため、輪読会の課題図書として読み進めることができ、とても良かったです。

結論として、バックエンド側の開発経験に乏しい私のような者が読んでも、すべてを簡単に理解できる代物ではなかったです。
しかしながら、たくさんの新しい概念に触れることができたり、分散システムの運用がいかに複雑で大変なことなのかを知る機会となりました。

第 2 部 分散データは、全ての章が面白かったです。

本書籍は概念を中心に説明されているため、なかなかイメージしづらいものもありました。
補助教材として、以下に記載する 2 冊の書籍がとても参考になりました。

###  補助教材 1. マイクロサービスパターン[実践的システムデザインのためのコード解説]

@[card](https://www.amazon.co.jp/%E3%83%9E%E3%82%A4%E3%82%AF%E3%83%AD%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3-%E5%AE%9F%E8%B7%B5%E7%9A%84%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%89%E8%A7%A3%E8%AA%AC-impress-top-gear/dp/4295008583)

Amazon のワンコインセールキャンペーンで購入して夏休み中に読みました。

具体的な図と一緒にたくさんのパターンを紹介してくれています。

### 補助教材 2. Go言語による分散サービス

@[card](https://www.oreilly.co.jp/books/9784873119977/)

Go 言語で分散サービスを実際にコードで書きながら、触れることができました。

「データ指向アプリケーションデザイン」第 9 章 一貫性と合意 で紹介されている分散合意アルゴリズムのライブラリを使った実装を取り組むことができます。

-----

## 3. A Philosophy of Software Design

@[card](https://www.amazon.co.jp/Philosophy-Software-Design-2nd/dp/173210221X)

Amazon の洋書 Programming 部門ランキングで 1 位となっていて気になり、購入して読みました。

本書籍の主題は「複雑性」です。
いかに開発コードから複雑性をなくすかは私自身も意識しているところなので、読む前からワクワクしました。

この書籍を読んで良かった点は、数多くのことを新しく学べることです。
著者独自の目線からの問題を提起したり、時には定説となっているプログラミング哲学に反対する意見を言及しています。

読んでいた中で印象に残っているものは、以下になります。

- 自分からは一見シンプルだが、誰かが複雑だと思ったコードは、複雑なコードである
- 大きなクラスを複数の小さなクラスに分解することは、浅いクラスとメソッドを生み出し、結果的にシステム全体に複雑性をもたらす
- 必ずしも行数の多い関数を分割することで、システム全体の複雑性が減少されるわけではない
  - 関数分割したことで関数を行ったり来たりしないと処理を理解できない場合には赤フラグです
- たくさんの例外処理をすることが正ではなく、システムに複雑性をもたらすだけである
  - 例外処理をするべき箇所に限定して、複雑性を減少させる
- 2 回デザインしよう
  - 1 回目に思いついた設計からすぐに動くのではなく、時間を使って 2 回目の設計をして、より良い設計を選択しよう
- 「どのようにコードが動いているのか」ではなく、「何をするコードなのか」、「なぜ導入されたのか」の良いコメントを書く
- コードが誰かにとって明白ではなかった場合、口論するのではなく、なぜ明白ではないのかを理解し、より良いコメントやコードを書くマインドセットを持とう
- 既存の慣習を変更しない。既存の慣習を「改善しよう」という衝動を抑えよう。「より良いアイディア」を持つことは、一貫性を壊す十分な理由ではない

-----

## 4. System Design Interview – An insider's guide

@[card](https://www.amazon.co.jp/System-Design-Interview-insiders-Second/dp/B08CMF2CQF/)

上述した「A Philosophy of Software Design」に次いで上位にランクされていて気になり、購入して読みました。

本書籍は、エンジニア面接で受けるシステムデザインやシステムアーキテクチャに関する質問に対して、準備するための教本です。

しかしながら、様々な要件に応じたアーキテクチャパターンの例に触れることができて楽しかったです。

例えば、以下のような要件の例題があります。

- 数百万人のユーザーをサポートするシステム
- 通信回数にリミットを導入するシステム
- 一貫性のあるハッシュ化システム
- ユニークIDを生成する分散システム
- オートコンプリートシステム
- Youtube のシステム
- Google Drive のシステム

-----

## 5. Go言語による並行処理

@[card](https://www.oreilly.co.jp/books/9784873118468/)

補助教材として挙げた「Go言語による分散サービス」を通じて Go 言語を写経する中で、Go 言語をよく知りたいと思い、購入して読みました。

2021 年冬に私の中で Go 言語ブームがあり、何冊か書籍を読んだり、実際にコードを少し書いてみたりしていました。
しかしながら、当時は携わる案件で Kotlin を学習する必要性が出ていたため、Go 言語を途中で投げ出してしまいました。

投げやりな状態だった Go 言語、本書籍を読んだことで一気に熱が戻ってきました。
本書籍は Go 言語がいかに魅力的な言語なのかが詰まっている本です。

本書籍では、並行処理だけでなく、Go 言語のベースとなる哲学にも触れることができます。

# ビジネス書

## 1. サーバント・リーダー 「権力」ではない。「権威」を求めよ

@[card](https://www.amazon.co.jp/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%B3%E3%83%88%E3%83%BB%E3%83%AA%E3%83%BC%E3%83%80%E3%83%BC-%E3%80%8C%E6%A8%A9%E5%8A%9B%E3%80%8D%E3%81%A7%E3%81%AF%E3%81%AA%E3%81%84%E3%80%82%E3%80%8C%E6%A8%A9%E5%A8%81%E3%80%8D%E3%82%92%E6%B1%82%E3%82%81%E3%82%88-%E3%82%B8%E3%82%A7%E3%83%BC%E3%83%A0%E3%82%BA%E3%83%BB%E3%83%8F%E3%83%B3%E3%82%BF%E3%83%BC/dp/4903212351)

担当するプロジェクトで、今年 3 月からチームリーダーのポジションになる際に、リーダー論を学びたいと思い、この書籍を読みました。

結論として、私が目指したいリーダー像を言語化してくれていると感じました。

同時に、リーダー論に関して私に最も影響を与えた本です。

「時には自身を犠牲にしてでも他者に対して奉仕し、そこから信頼を得て影響力を持つようになり、その影響力で他者を導いていくリーダーシップ」に関心のある方にぜひオススメしたいです。

この本に関連して、最近日本語版が出版された以下の本が気になっています。

- [アジャイルリーダーシップ―変化に適応するアジャイルな組織をつくる―](https://www.kyoritsu-pub.co.jp/book/b10023139.html)

-----

## 2. エンジニアリングマネージャーのしごと

@[card](https://www.oreilly.co.jp/books/9784873119946/)

最近よく目にする機会が増えてきたエンジニアリングマネージャーという役職。

私はチームリーダーですが、担当する業務や自身が行っている取り組みに関連することが本書籍で多く扱われているために、購入して読みました。

以下の点で参考になりました。

- 委譲
- 1on1
- 各メンバーに合う仕事を考える
- 採用活動

とりわけ委譲に関しては、この本をきっかけに強く意識して業務を取り組むきっかけになりました。

-----

## 3. HIGH OUTPUT MANAGEMENT－人を育て、成果を最大にするマネジメント

@[card](https://www.amazon.co.jp/OUTPUT-MANAGEMENT-%E3%83%8F%E3%82%A4%E3%82%A2%E3%82%A6%E3%83%88%E3%83%97%E3%83%83%E3%83%88-%E3%83%9E%E3%83%8D%E3%82%B8%E3%83%A1%E3%83%B3%E3%83%88-%E4%BA%BA%E3%82%92%E8%82%B2%E3%81%A6%E3%80%81%E6%88%90%E6%9E%9C%E3%82%92%E6%9C%80%E5%A4%A7%E3%81%AB%E3%81%99%E3%82%8B%E3%83%9E%E3%83%8D%E3%82%B8%E3%83%A1%E3%83%B3%E3%83%88/dp/4822255018)

Twitter タイムライン上でよく薦めている光景を目にしていたために気になっていて、地元の図書館で借りて読みました。

私自身を「マネージャー」として捉えていませんが、日々の業務にあたる中で以下のような悩みがありました。

「日々の業務で設計やコードを書く時間や機会が減りつつある中で、私はどんな価値を出して、会社に貢献できているのか？」

本書籍で書かれていた以下の計算式で、その悩みはいくばくか解消されました。

> マネージャーのアウトプット = 自分の組織のアウトプット + 自分の影響が及ぶ隣接諸組織のアウトプット

また、この計算式と本書籍のおかげで、自身に任されたポジションの役割を認識し、どうやってチームメンバーがより多くのアウトプットをできるか、を意識して業務にあたるようになりました。

-----

## 4. 解像度を上げる――曖昧な思考を明晰にする「深さ・広さ・構造・時間」の４視点と行動法

@[card](https://www.amazon.co.jp/%E8%A7%A3%E5%83%8F%E5%BA%A6%E3%82%92%E4%B8%8A%E3%81%92%E3%82%8B%E2%80%95%E2%80%95%E6%9B%96%E6%98%A7%E3%81%AA%E6%80%9D%E8%80%83%E3%82%92%E6%98%8E%E6%99%B0%E3%81%AB%E3%81%99%E3%82%8B%E3%80%8C%E6%B7%B1%E3%81%95%E3%83%BB%E5%BA%83%E3%81%95%E3%83%BB%E6%A7%8B%E9%80%A0%E3%83%BB%E6%99%82%E9%96%93%E3%80%8D%E3%81%AE%EF%BC%94%E8%A6%96%E7%82%B9%E3%81%A8%E8%A1%8C%E5%8B%95%E6%B3%95-%E9%A6%AC%E7%94%B0%E9%9A%86%E6%98%8E/dp/4862763189)

発売前に予約をして、今すぐにでも読みたいと思っていた書籍です。

現在任されているチームリーダーというポジションになってから、自身の解像度の低さに課題を感じていました。

とりわけ、コミュニケーションの中で、解像度の低さに問題を感じていました。
相手が求めているものの本質をしっかりと捉えられず、コミュニケーションの齟齬（そご）が発生して、無駄な時間を作ってしまっているように感じていたからです。

本書籍は、私のように解像度の低さを課題として感じている人に、解像度を上げるための工夫やテクニックをたくさん紹介してくれています。

以下のようなアドバイスは、日々の業務でもすぐに意識して取り組めるものなので、ぜひ皆さん実践してみてください。

> 社内で何か仕事を依頼されているのであれば、なぜこの仕事が重要なのか、なぜ自分にこの仕事が割り当てられたのかを考え、場合によっては仕事の意味を捉えなおすのです。

> 良い質問をするのは、とても難しいからこそ、良い質問をしようとすることで、必然的に解像度を上げることになりますし、質問の答えとして受け取れる情報は、解像度をさらに上げるために大いに役立つでしょう。

-----

## 5. LISTEN――知性豊かで創造力がある人になれる

@[card](https://www.amazon.co.jp/LISTEN%E2%80%95%E2%80%95%E7%9F%A5%E6%80%A7%E8%B1%8A%E3%81%8B%E3%81%A7%E5%89%B5%E9%80%A0%E5%8A%9B%E3%81%8C%E3%81%82%E3%82%8B%E4%BA%BA%E3%81%AB%E3%81%AA%E3%82%8C%E3%82%8B-%E3%82%B1%E3%82%A4%E3%83%88%E3%83%BB%E3%83%9E%E3%83%BC%E3%83%95%E3%82%A3/dp/4822289001)

いかに私が悪い聞き手なんだと痛感させられると同時に、良い気づきとなった一冊でした。

> 優れた聞き手は、それを承知の上で質問を投げかけ、もう少し詳しく話すよう働きかけることで、話し手が答えを自分で気づくように手助けします。

> しっかりと聞くとは、話し手の言わんとしている内容は妥当か、その話を聞かせてくれる動機は何かを、自問し続けること

より良いメンター、より良いチームのメンバーの一人になるために、聞き手としてのスキルを日々少しづつ改善していきたいと思わせてくれました。

-----

# 終わりに

エンジニア 2 年目は意識してたくさんの本を読むようにしましたが、エンジニア 3 年目はより多くの本を読んでいきたいです。

最後に今年出会った良い言葉を書き残して、終わります。

> 読む量は学力の上限を規定し、書く量は学力の下限を規定する
> 岸本裕史先生


:::message
アドベントカレンダーの次回メンバーの記事お楽しみに！ぜひみなさんチェックしてくださいね！
:::