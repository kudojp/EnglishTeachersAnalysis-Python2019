# DMM英会話の講師情報のスクレイピングと基礎統計集計



## 概要
- このプロジェクトでは、[DMM英会話](https://eikaiwa.dmm.com)の[1]講師約5000人のプロフィール [2]30万件の受講者からの評価コメント、をwebスクレイピングして抽出し、その基礎統計を可視化した。
- 当初は、各生徒にとっておすすめの講師を抽出するレコメンデーションシステムを構築しようと考えたが、**そもそも受講レッスンの
をしている受講者側の情報が全くない**という致命的な欠陥に気付き、基礎統計の集計にとどまった。

## 関連URL

- 解析結果をまとめたMediumのブログ記事、[Are older teachers more favored in DMM Eikaiwa?](https://medium.com/@daikikudo/are-older-teachers-more-favored-in-dmm-eikaiwa-1fd65ce99fce)
- このプロジェクトで使用された[Jupyter Notebook3本](https://kudojp.github.io/EnglishTeachersAnalysis/)


## このRepoのファイル構成 

```
.
├── README.md
├── FinalReport.pdf                              (統計情報をまとめたブログ形式のレポート。Mediumのブログ記事と中身は同一。)
├── ipynb                                        (スクレイピングのために使用したJupyterNotebooks)
│   ├── 190111_reviews_scraping.ipynb               (レッスン受講者のコメントをスクレイピングする際に使用したJupyterNotebook)
│   ├── 190117_teachers_scraping.ipynb              (講師のプロフィール情報をスクレイピングする際に使用したJupyterNotebook)
│   └── 190306_wrangling_for_writing_blog.ipynb     (スクレイピング終了後、基礎統計を集計する際に使用したJupyterNotebook)
└── html_of_ipynb                                (上のJupyterNotebookをhtmlエクスポートしたファイル)
    ├── 190111_reviews_scraping.html
    ├── 190117_teachers_scraping.html
    └── 190306_wrangling_for_writing_blog.html
    
以下は、スクレイピングしたデータをエクスポートした[テキスト|sqlite]ファイルであり、GitHubにはプッシュしていない。

- teacher_urls.txt                               (全ての講師のプロフィールのURLを記録したテキストファイル)
- raw_teachers_data2.db                          (スクレイピングした講師情報の生データを収納したsqlite3ファイル)  
- teachers_data.db                               (クリーニングされた講師情報を収納したsqlite3ファイル) 
- reviews.db                                     (スクレイピングした評価コメントのデータを収納したsqlite3ファイル[重複行を含んでしまっている])  
- reviews2.db                                    (クリーニングされた評価コメントのデータを収納したsqlite3ファイル)  
```

## 技術選定

### 使用言語とミドルウェア
- **Python3**: JupyterNotebook上で使用
- **sqlite3**: スクレイピングしたデータの永続化のために使用


### 使用ライブラリ(いずれもPython)

|ライブラリ名|使用用途|
|----|----|
|**requests**|HTTPリクエストにより該当エンドポイントURLからHTMLを取得するために使用|
|**time**| HTTPリクエストを連続して行わないよう、リクエスト毎に1秒ずつ間隔を開けるために使用|
|**bs4[BeautifulSoup]**| 取得したHTMLをツリー構造にパースするために使用|
|**pandas**| 抽出したデータをJupyterNotebook上で二次元配列として保持するために使用|
|**sqlite3**| pandasデータフレームとSQLiteファイルとの間のI/Oのために使用|
|**pickle**| 変数(文字列の配列)とテキストファイルとの間ののI/Oのために使用|
|**matplotlib**| 抽出したデータをグラフで可視化するために使用|


## 作業手順

### [190117_teachers_scraping.ipynb](https://kudojp.github.io/EnglishTeachersAnalysis/190117_teachers_scraping.html)

講師のプロフィールをスクレイピングする。

1. https://eikaiwa.dmm.com/teacher/index/[:id] で表される講師のプロファイルURLを叩いた時に返されるHTMLを、全ての講師に関して取得する。なお、取得したデータはpickleを用いてテキストファイル`teacher_urls.txt`に書き出しすことで永続化する。
    - 講師のIDは連番にはなっておらず、飛び飛びの番号が振り当てられている(やめた講師だろうか)。このため、まず講師のプロファイルのURLの一覧を「予約・講師検索ページ > 全ての講師タブ」から取得する必要があった。
    - 一つの講師一覧ページに表示されている講師の数は12人である。このため、まず1ページ目では講師12人分のプロフィールのURLを取得し`teacher_url`というリストにappendした。尚且つ「次へ」ボタンのリンク先のURLを抽出して(1秒間間隔を開けた上で)このページをスクレイピングすることで、同様に13~24番目の講師を取得する、、、ということをプロフィール一覧の最終ページに達するまで繰り返す。
    - このような周りくどいやり方をとっているのは、例えば講師一覧の2ページ目(13人目~24人目の講師が表示されているページ)のURLは https://eikaiwa.dmm.com/list/?per_page=12&page=2 のようにページネーションの番号が指定されているわけではなく、以下のスニペットのようにparamという暗号化された文字列によってページネーションを指定しているからである。このため、講師一覧の1ページ目の「次へ」ボタンのリンク先のURLを取得する他なかった。
    ```
    https://eikaiwa.dmm.com/list/?param=BFsBAVxNEggJVQsaQEIHRRU6Ql9dXEYCEVgDXBALWF4AQwtKXA5bEF0LVWdHXwtSQ15FDAQDRlcNDFJECRYNCF9DUlYJXQBQVAATA1oMVgwSXwIMEk0dSQdADAcIVQ1LGBIKDlwUAl1NC0VKShRdXltVDUUKD14bBQdZAlcXFQsMWwACFQxSCBoEVl0RDRUNV18U04mNjYTAQAwVCFENEgsAXVxEDS8JS18FAhFSB0MERw1FCghUA0BQB1QCSAcHSFEFG11FWwYCR0JXQUJEDBJfBwwSDUYCEVgOXBADRVUAPkdWFFJDCUtfAQIRFF1EW10MFFZYElYQC0MDEF55CxZbBgNEWABGURNUGgh4XURbVAcMElcFTQsUUjlBDVhHR1p*AhUMUggaC1RPEQ0PDVFeRQwBCF4bCBJoElcEVFgAE0MbXXhaQQJRCxpDVwFSQ15fDAICFwNRWBUSUwcVCwxbAQIb
    ```
    <img width="558" alt="dmm-all-tutors" src="https://user-images.githubusercontent.com/44487754/86532633-8e84d380-bf06-11ea-8422-5f0daa2daddb.png">



2. 1で取得した各講師のプロフィールページの各URLにGETリクエストを投げてHTMLを取得し、各ページに含まれるデータを抽出することを繰り返す。
    - 講師600人ごとに、`teachers`というpandasのデータフレームを毎回初期化して、(1秒の間隔を開けながら)講師のデータを取得してはこのデータフレームに追加してを繰り返す。講師600人分のデータの収納が完了したら、このデータフレームをsqliteファイル`raw_teachers_data2.db`に書き込む。
    - この操作はおおよそ600人分一回ごとに3200秒程度の時間を要した。
    <img width="668" alt="Untitled 3" src="https://user-images.githubusercontent.com/44487754/86534846-c8f66c80-bf16-11ea-924a-0188089c5850.png">

3. 2で取得したデータが収納されたsqliteファイル`raw_teachers_data2.db`を再びpandasのデータフレーム`teachers2`にロードし、以下のようにデータの整形かつクリーニングをする。終了後、再びこのデータフレームをsqliteファイル`teachers_data.db`に書き出す。
    - カラム`id`, `age`. `fav(お気に入りの数)`がstr型として収納されてしまっているため、これをint形に変える。
    - 講師の各レコードのに付けられたタグ(`ビジネス英会話`、`キッズ向け`、`TOEFL対応`など)が、`tag`という一つのカラムに文字列の配列として入っているので、存在する全てのタグを新しいカラムとして追加し、講師が該当するカラムには1を、該当しないカラムには0を収納する。(なぜbooleanにしなかったかというと、将来的に機械学習モデルを構築する可能性が想定したからである)
    - 日本人講師の全員に関して、カラム`staff_comment(日本語での紹介文)`には以下の二つの文章のうちのいずれかが収納されているため、これをnullに置き換える。
    
    ```
    (1)プラスネイティブプランのユーザー様に限り、日本人講師のレッスンをご予約いただけます（試験運用中）。「学習カウンセリング」を受けたり、英語に関する疑問を日本語で質問をすることが可能となります。「海外生活」、「旅行」、「試験対策」、など様々なトピックについてご相談いただけます。また、無料体験の際にも一度だけ受講可能です。「いきなり外国人の先生と話せない。」「レッスンの進め方が不安。」という方でも安心です。
    (2)この講師は無料体験レッスン専用講師です。無料体験の際に一度だけ受講可能です。 「いきなり外国人の先生と話せない。」「レッスンの進め方が不安。」という方のためのプロの日本人講師です。\u3000日本人講師と一緒にレッスンの流れを掴んで素敵なオンライン英会話デビューを飾りましょう！※現在、試験的にネイティブプランのユーザーも受講可能としています。そちらは予告なく提供を終了する場合がございます。
    ```

### [190111_reviews_scraping.ipynb](https://kudojp.github.io/EnglishTeachersAnalysis/190111_reviews_scraping)

講師が提供したレッスンの受講者からの感想コメントを全てスクレイピングする。

1. それぞれの講師に関して、その講師を受講したユーザーからの全評価コメントを「講師のプロファイルページ > 評価・コメントタブ」からスクレイピングする。
    - 評価コメントは一つのページに5つずつ表示されている。特定のページ番号のボタンや「次へ」ボタンを押した際にはAjaxで`POST https://eikaiwa.dmm.com/teacher/tab_ajax_ratecomment/`に対して、FormDataとして`teacher_id=9999&page=2&rating=0`といった形でページネーションを指定して、リクエストを投げている。
    - したがって、各講師に関して、FormDataのpage変数を1から最終ページ番号までそれぞれ指定してリクエストを投げてHTMLを取得することを繰り返すことで、その講師に対して投稿された全ての評価コメントのデータを取得することができる。
    - 講師100人ごとに、`df_reviews`というpandasのデータフレームを初期化して、(1秒の間隔を開けながら)評価コメント5件ずつを取得してはこのデータフレームに追加してを繰り返す。講師100人に対応する全評価コメントののデータの収納が完了したら、このデータフレームをsqliteファイル`reviews.db`に書き込む。
    - この操作はおおよそ講師100人ごとに2000-5000秒程度の時間を要した(講師によってコメント数にばらつきがあるため、時間の幅が生じる)。
    <img width="668" alt="dmm_reviews" src="https://user-images.githubusercontent.com/44487754/86534859-e4617780-bf16-11ea-9685-62ef3567c999.png">

2. 1で取得したデータが収納されたsqliteファイル`reviews.db`を再びpandasのデータフレーム`reviews2`にロードし、以下のようにクリーニングをする。終了後、再びこのデータフレームをsqliteファイル`reviews2.db`に書き出す。
    - 2016-06-15日以前に投稿された全ての評価コメントに関して、(1-5の選択肢外の)0というスコアがratingカラムに収納されている。これは評価コメントを投稿する際にスコアを入力するシステムがこの日以降導入されたからだと推測される。したがって、ratingカラムの0に関しては全てnullに置き換える。

    
### [190306_wrangling_for_writing_blog](https://kudojp.github.io/EnglishTeachersAnalysis/190306_wrangling_for_writing_blog)

基礎統計の可視化や、ブログ記事になるような知見の発見を目的に以下のデータラングリングを行った。

1. Preliminary Wrangling
2. Univariate Exploration
    - Distribution of countries teachers are from
    - Distribution of region teachers are from
    - Distribution of age of teachers
    - Distribution of gender of teachers
    - Distribution of number of students who liked that teacher as "favorite"
    - Distribution of each average evaluation score from students for each of teachers
    - Distribution of evaluation score for each lesson.
    - Distribution of experience years of teachers.
3. Bivariate Exploration
    - Relationship between age and favorite count
    - Relationship between age and favorite count
    - What can be said about teachers who get more "fav"s?

## 抽出したデータを保存したデータベース

[弁明]このテーブルを作った2018年時点の自分はDB設計の知見のかけらもなかった。

### teachers_data.db

**teachersテーブル**

|カラム名|型|備考|
|----|----|----|
|id| INTEGER ||
|name| STRING ||
|country|STRING||
|age|INTEGER||
|male|INTEGER|該当する行には1, 該当しない行には0|
|school|STRING||
|hobby|STRING||
|movie|STRING||
|fav|INTEGER||
|evalu|INTEGER||
|n_evalu|INTEGER||
|rate5|INTEGER|該当する行には1, 該当しない行には0|
|rate4|INTEGER|同上|
|rate3|INTEGER|同上|
|rate2|INTEGER|同上|
|rate1|INTEGER|同上|
|message|STRING||
|staff_comment|STRING||
|日本人講師|INTEGER|該当する行には1, 該当しない行には0|
|ビジネス英会話|INTEGER|同上|
|キッズ向け|INTEGER|同上|
|TOEFL対応|INTEGER|同上|
|初心者向け|INTEGER|同上|
|上級者向け|INTEGER|同上|
|スピーキングテスト対応|INTEGER|同上|
|Let'sGo対応|INTEGER|同上|
|英検®対応|INTEGER|同上|
|IELTS|INTEGER|同上|
|ネイティブ講師|INTEGER|同上|
|1年未満|INTEGER|同上|
|1年|INTEGER|同上|
|2年|INTEGER|同上|
|3年以上|INTEGER|同上|

### reviews2.db

**reviewsテーブル**

|カラム名|型|備考|
|----|----|----|
|teacher_id|INTEGER|
|name|TEXT||
|date|TEXT||
|rating|REAL||
|review|TEXT||




