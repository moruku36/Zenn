---
title: "FDE（フォワード・デプロイド・エンジニア）とは？ 生まれた経緯から、各社のビジネスモデル、日本での広がりまでをやさしく整理する"
emoji: "🛰️"
type: "idea"
topics: ["fde", "openai", "anthropic", "palantir", "ai"]
published: false
---

## はじめに

2026年10月22日（日本時間では23日の朝）、OpenAI が「最前線での仕事：OpenAI のフォワード・デプロイド・エンジニアリング」というオンラインイベントを開き、日本語の紹介ページも公開しました。

https://openai.com/ja-JP/business/forward-deployed-engineering/

FDE（Forward Deployed Engineer、フォワード・デプロイド・エンジニア）という言葉は、ここ1年ほどで急に目にするようになりました。一方で、こんな疑問もよく聞かれます。

- お客さまのところに行って話を聞き、システムを作る仕事なら、これまでのシステムエンジニア（SE）や受託開発と何が違うのでしょうか。
- もともとは Palantir という会社が始めた働き方だそうですが、なぜ今になって AI 企業が次々と取り入れているのでしょうか。
- 結局、どうやって利益を出す仕組みなのでしょうか。

この記事では、これらの疑問に答えられるように、公開されている情報（各社の発表、決算資料、報道、求人情報）を時代の流れに沿って整理しました。情報は **2026年10月23日時点** のものです。FDE はまさに今かたちが作られている途中の働き方なので、この記事も**第1版**として、今後の動きに合わせて更新していく予定です。

:::message
**この記事のポイント**
- FDE は「お客さまのところに常駐する開発者」というだけではありません。**本番で使われて成果が出るところまで責任を持つこと**と、**現場で学んだことを自社の製品や AI モデルの改善に生かすこと**の2点が、従来の受託開発との大きな違いです。
- 始まりは Palantir です。現場で作った仕組みを製品開発チームが整え直して製品の機能にし、その製品の利用料で収益を上げてきました。
- 2025年には「企業の生成 AI の試験導入は、ほとんどが成果につながっていない」という調査結果が話題になりました。そして2026年には、OpenAI、Anthropic、Microsoft、AWS などが、合わせて1兆円を超える資金を FDE の組織に投じています。
- 収益の上げ方は、大きく3つの型に分けられます。**製品の利用料で回収する型**、**導入の支援料と AI の利用料で回収する型**、**作業時間ではなく成果や固定の金額で請求する型**です。日本の大手 IT 企業は、3つ目の型として FDE に取り組み始めています。
:::

### この記事で使う言葉

| 言葉 | 意味 |
|---|---|
| 本番導入・本番稼働 | 試しに使うだけでなく、毎日の業務で実際に使われる状態にすること |
| PoC（ピーオーシー） | 本格的に導入する前に、うまくいくかを小さく試すこと。「実証実験」とも言います |
| 人月（にんげつ） | 「1人が1か月働く量」を単位にして、かかった人数と期間で料金を決める方式。日本のシステム開発で広く使われてきました |
| 従量課金 | 使った量に応じて料金がかかる方式。AI の場合は、処理した文章の量（トークン）などで決まります |
| 粗利率 | 売上から、商品やサービスを作るのに直接かかった費用を引いた利益の割合。ソフトウェア企業は高く、人が作業するサービス企業は低くなりやすいです |
| PE ファンド | 企業に出資し、経営を改善して企業の価値を高めることで利益を得る投資会社 |

---

## 1. OpenAI はどんなことを発表したのか

まず、きっかけになった OpenAI のページの内容を確認します。

| 項目 | 内容 |
|---|---|
| どんな役割か | FDE がお客さまのチームに加わり、仕事の流れを観察しながら、AI を本番で使える形にする |
| 進め方 | 目指す成果を決める → 課題を探す → 試作する → 検証する → 本番稼働させる → 社内に広げる |
| 使う道具 | OpenAI Presence（AI エージェントの基盤）、Context Layer（社内の知識をまとめる基盤）、業務の流れの見える化、Codex による古いプログラムの刷新 |
| 事例 | T-Mobile（お客さまとのチャットの55%を、人に引き継がずに対応）、BBVA（信用リスク評価の精度80%）、15,000のシステムの刷新など |
| 体制 | OpenAI 本体の FDE と、子会社の **DeployCo（OpenAI Deployment Company）** が協力して進める |

![目指す成果を決めるところから始まり、課題探索、試作、検証、本番稼働、展開の5つの段階で進む。試作と検証で見つかった AI モデルの弱点は OpenAI の製品・研究チームに伝えられ、改善がプロジェクトに戻ってくる。展開の段階では DeployCo や IT パートナーが支援を引き継ぎ、社内に広げていく](/images/forward-deployed-engineering/fig5-openai-process.png)
*図1　OpenAI の FDE プロジェクトの流れ（OpenAI の紹介ページをもとに作成）*

このページの「よくある質問」には、FDE の性格がよく分かる答えが2つあります。

- 「FDE はプロフェッショナルサービス（専門家による受託サービス）を提供するのですか」という質問に、「**いいえ**」と答えています。サービスを提供する会社と協力しながら、AI の最先端の課題を解くエンジニアだという説明です。
- 「**今の AI の能力で実現できることであれば、通常 FDE は必要ありません**」とも書かれています。

つまり OpenAI の FDE は、どんな依頼でも引き受けるわけではありません。**今の AI ではまだ本番で使うのが難しい案件**に入り、そこで見つかった AI の弱点を製品・研究チームに伝える役割だと説明されています。そして、たくさんの会社に広く導入していく部分は、子会社の DeployCo や協力会社が担う、という分担になっています。

---

## 2. これまでの流れ：Palantir から AI 各社へ

![FDE の歴史を3つの時期に分けた年表。始まりの時期は、Palantir の創業と Shyam Sankar による FDE の考案、Airbus の案件と Foundry の誕生、2020年の上場と「サービス会社ではないか」という批判。見直しの時期は、2023年の AIP Bootcamp、2025年の FDE の求人の急増と MIT の調査、OpenAI の FDE チームの発足。大型投資の時期は、2026年5月の OpenAI DeployCo と Anthropic の合弁会社、6〜7月の AWS と Microsoft、8〜10月の日本の IT 大手の動き、Anthropic の育成プログラム、OpenAI のイベント](/images/forward-deployed-engineering/fig1-timeline.png)
*図2　FDE のこれまでの流れ（各種資料をもとに作成）*

### 2-1. 始まり：Palantir（2000年代後半〜2010年代）

FDE という職種を作ったのは、アメリカのデータ分析ソフトウェア企業 Palantir です。2006年に13人目の社員として入社した **Shyam Sankar**（現在の最高技術責任者）が考え出したとされています（[経歴紹介](https://goldhouse.org/people/shyam-sankar/)）。Palantir はもともと政府や情報機関向けにソフトウェアを作っていました。こうしたお客さまは、機密が多く、仕事の内容も特殊です。そのため「要望を聞いて持ち帰り、会社で作る」という進め方では、うまく課題を解決できませんでした。

Palantir では、お客さまの現場に出る人を2つの役割に分けていました（[Palantir 公式ブログ](https://blog.palantir.com/a-day-in-the-life-of-a-palantir-deployment-strategist-951cb59a5a96)）。

- **Delta（デルタ）**：Forward Deployed Software Engineer。現場で実際にプログラムを書いて作る人
- **Echo（エコー）**：Deployment Strategist。課題と成果に責任を持ち、お客さまの組織を動かしていく人

公式ブログでは「理屈の上では Echo は企画寄り、Delta は技術寄りだが、実際にはどちらも企画・開発・戦略のすべてにかかわる」と説明されています。

Palantir で約8年働いた Nabeel Qureshi さんのエッセイ「[Reflections on Palantir](https://nabeelqu.substack.com/p/reflections-on-palantir)」（2024年）は、この仕組みを内側から説明した貴重な資料です。主な内容は次のとおりです。

- FDE は週に3〜4日をお客さまのオフィスで過ごし、製造、医療、情報、航空宇宙といった専門性の高い業界の仕事を学んだ
- 最初の大きな民間の案件は、フランス・トゥールーズの **Airbus** だった。旅客機 A350 の生産が滞る原因を解消するソフトウェアを作り、生産のペースを大きく上げた
- FDE はスピードを優先し、多少粗い作りでもまず動かした。それを**製品開発チームが整え直して**、多くのお客さまで使える、安定したソフトウェアにした
- 主力製品 Foundry の多くの機能は、FDE が現場で手作業でしていたことを、製品開発チームが自動化して生まれた

![Palantir の循環。お客さまの現場に Echo と Delta が常駐し、まず素早く課題を解決する。製品開発チームがそれを整え直し、Foundry や AIP の標準機能にする。次のお客さまにはより速く安く導入でき、Bootcamp では数日で動くものができ、契約が広がってソフトウェアの利用料が入る。その利益が次の現場に再び投資される](/images/forward-deployed-engineering/fig4-palantir-loop.png)
*図3　Palantir の「現場 → 製品 → 次の現場」という循環（Qureshi さんのエッセイをもとに作成）*

### 2-2. 「Palantir はサービス会社では？」という批判と、その後

FDE をたくさん抱えるやり方は、長いあいだ「それはソフトウェア会社ではなく、コンサルティング会社ではないか」と批判されてきました。2020年の上場前後にも、こうした声は多くありました。Qureshi さんも「2016年ごろには、ある程度当たっていた」と書いています。

ところが、その後の数字はソフトウェア会社らしいものになりました。

- 2023年の粗利率は約80%でした（Qureshi さんのエッセイより。比較として挙げられている Accenture は約32%）。
- 2026年4〜6月期の決算では、売上高が前年の同じ時期より93%増えて19.35億ドルになり、**アメリカの民間企業向けの売上は149%増**の7.64億ドルでした（[Palantir の決算発表](https://www.sec.gov/Archives/edgar/data/0001321655/000132165526000039/a2026q2ex991pressrelease.htm)）。粗利率は約85%と報じられています（報道による数字です）。

現場での工夫が製品に取り込まれていくので、お客さまが増えても人手による作業の割合はあまり増えませんでした。これが、サービス会社にならずに済んだ理由だと説明されています。

### 2-3. AIP Bootcamp（2023年〜）

生成 AI が広まった2023年、Palantir は **AIP Bootcamp** を始めました。[2024年度の年次報告書](https://www.sec.gov/Archives/edgar/data/1321655/000132165525000022/pltr-20241231.htm)では、「お客さまの実際のデータを使って、実際の業務の仕組みを数日で届ける」取り組みだと説明されています。一般には5日間ほどの集中ワークショップとして紹介されることが多く、FDE がお客さまと一緒に、そのお客さまのデータで AI の使い道を形にします。そこから本格的な契約につなげていくので、FDE が**営業の入り口**の役割も担っていることになります。

### 2-4. 生成 AI の時代に改めて注目される（2025年）

2025年に FDE が急に注目されたのには、主に2つの理由があります。

1. **試験導入が成果につながらない、という問題がはっきりした**
   マサチューセッツ工科大学（MIT）の研究グループ NANDA は、報告書「The GenAI Divide: State of AI in Business 2025」で、企業の生成 AI の試験導入の約95%は、売上や利益に目に見える効果を出せていないと報告しました（[Fortune の記事](https://fortune.com/2025/08/18/mit-report-95-percent-generative-ai-pilots-at-companies-failing-cfo)）。原因は AI の性能ではなく、組織への取り入れ方にあるとされています。なお、この数字は初期段階の調査によるもので、調査の対象数は報道によって違いがあります。
2. **求人が大きく増えた**
   FDE の求人は、2025年1〜9月で800%以上増えたという報道（Financial Times、[Fast Company 経由](https://fastcompany.co.za/work-life/2025-11-08-forward-deployed-engineers-a-new-role-in-the-evolving-ai-job-market)）や、前の年と比べて1,165%増えたという分析（[Bloomberry](https://bloomberry.com/blog/i-analyzed-1000-forward-deployed-engineer-jobs/)）があります。集計の方法がそれぞれ違うので単純には比べられませんが、**1年でおよそ8〜12倍**という規模感はほぼ共通しています。

OpenAI の FDE チームも、この時期に**2人から始まり**、2026年8月には約140人になったと、同社の FDE 責任者 Colin Jarvis さんがインタビューで話しています（[Engineering Leadership](https://newsletter.eng-leadership.com/p/inside-openais-forward-deployed-engineer)）。

### 2-5. 大型投資の年（2026年）

2026年に入ると、各社が専用の組織や会社を作り、大きな金額を投じ始めました。

| 時期 | 企業 | 動き | 規模 |
|---|---|---|---|
| 2026年5月4日 | Anthropic | 投資会社の Blackstone、Hellman & Friedman、金融大手の Goldman Sachs と、企業向け AI サービスの合弁会社を作ると発表。7月に **Ode with Anthropic** として本格的に始動 | 15億ドル（Anthropic、Blackstone、H&F がそれぞれ3億ドル）、エンジニア約100人 |
| 2026年5月11日 | OpenAI | 子会社 **OpenAI Deployment Company（DeployCo）** を設立。イギリスの Tomoro を買収 | 最初の投資額40億ドル超、Tomoro から FDE など約150人 |
| 2026年6月30日 | AWS | FDE の組織を新しく作る。45日単位のチームをお客さまに送る | 10億ドル |
| 2026年7月 | Microsoft | **Frontier Company** を設立（社内の組織。もとのコンサルティング部門を作り直したものだという見方もあります） | 25億ドル、約6,000人 |
| 2026年9月8日 | Accenture と Google Cloud | Google の AI「Gemini Enterprise」専門の部隊を作る | FDE 1,000人を目標 |
| 2026年10月2日 | Anthropic | **Claude Frontier Academy** を開始。協力会社のエンジニアを、医師の研修のような方式で育てる | 1億ドル、2027年末までに1万人 |

出典：[OpenAI](https://openai.com/index/openai-launches-the-deployment-company/)、[CNBC](https://www.cnbc.com/2026/05/04/anthropic-goldman-blackstone-ai-venture.html)、[TechCrunch](https://techcrunch.com/2026/07/15/anthropic-blackstone-bet-the-next-trillion-dollar-ai-business-is-implementation-not-models/)、[The Next Web（AWS）](https://thenextweb.com/news/aws-1bn-forward-deployed-engineers-enterprise-ai/)、[GeekWire（Microsoft）](https://geekwire.com/2026/microsoft-announces-2-5b-frontier-company-to-embed-ai-engineers-inside-customers)、[Directions on Microsoft](https://www.directionsonmicrosoft.com/microsoft-launches-its-own-forward-deployed-engineering-unit-the-frontier-company/)、[Anthropic](https://anthropic.com/news/claude-frontier-academy)

投資家の Tomasz Tunguz さんは、この1年で FDE に投じられたお金を、5社の合計で約97.5億ドル（約1.5兆円）と集計しています（[The FDE Arms Race](https://tomtunguz.com/fde-arms-race/)）。10月22日の OpenAI のイベントも、こうした流れの中にあります。

---

## 3. FDE はどんな仕事なのか

### 3-1. どんなすき間を埋める仕事なのか

![フロンティアモデルの能力と、目に見える業務の成果のあいだには「導入のすき間」がある。その中身は、ばらばらのデータと古いシステム、仕事の中にある暗黙のルールや例外、品質の確認と人によるチェック、権限・監視・責任の分担、現場への定着。FDE はこのすき間を埋め、現場で学んだことを製品・研究チームに伝え、次のお客さまではそれが標準の機能になる](/images/forward-deployed-engineering/fig2-deployment-gap.png)
*図4　導入のすき間と FDE の役割（OpenAI の説明をもとに作成）*

OpenAI はこのすき間を「導入ギャップ」と呼んでいます。AI がどれだけ賢くなっても、**社内のあちこちにあるデータ、言葉にされていない仕事のルール、アクセス権限の管理、現場の習慣**とつながらなければ、成果にはなりません。FDE は、お客さまの中に入って、このすき間を埋めていきます。

### 3-2. これまでの職種と何が違うのか

「SE や受託開発と何が違うの？」という疑問に答えるために、似た職種と並べてみます。

| | 営業を支える SE | 経営コンサルタント | 受託開発（人月） | **FDE** |
|---|---|---|---|---|
| 主に作るもの | 製品の紹介や提案 | 提案書 | 決められた仕様どおりのシステム | **本番で使われ、成果が出た仕組み** |
| 課題は誰が決めるか | お客さま | 一緒に考える | お客さま（要件を決める） | **一緒に探し、取り組む価値があるかも見極める** |
| 自分でプログラムを書くか | 少し | 書かない | 書く（大人数で） | **書く（少人数で）** |
| どこまで責任を持つか | 受注まで | 報告まで | 納品まで | **本番稼働と定着まで** |
| 学んだことはどこへ行くか | 営業資料 | 自社のノウハウ | 案件ごとに終わりがち | **自社の製品や AI の改善** |
| 料金のもと | 製品の売上 | 作業時間 | 人数 × 期間 | **製品や AI の利用料、固定の金額、成果** |

![横軸が「製品の改善につながる度合い」、縦軸が「本番での実装と成果への責任」の4つの区画。右上の FDE の区画に Palantir の FDE、OpenAI の FDE、DeployCo や Ode。左上に人月の受託開発と、IT 企業の FDE 型の取り組み。左下に経営コンサルタント。右下に営業を支える SE とカスタマーサクセス](/images/forward-deployed-engineering/fig3-role-map.png)
*図5　似た職種との位置関係（各社の説明をもとにした整理で、位置はおおよそのものです）*

違いは、大きく次の2点にまとめられます。

1. **本番で成果が出るところまで責任を持つ**
   従来の受託開発は「頼まれたものを作って納める」ところで、コンサルタントは「提案する」ところで区切りになります。FDE は、**業務の数字が実際に良くなるところまで**を受け持ちます。
2. **学んだことを製品の改善につなげる道がある**
   ここが一番大きな違いです。受託開発では、案件で得た知見がその案件だけで終わってしまうことがよくあります。FDE は、Palantir なら製品開発チームに、OpenAI なら研究チームに、現場で分かったことを伝えます。たとえば OpenAI では、「古いプログラム言語の COBOL を Java に書き換える作業で、AI が単純な間違いを繰り返す」という現場からの報告が、学習データを増やしてほしいという要望として研究チームに伝えられたそうです（[Engineering Leadership](https://newsletter.eng-leadership.com/p/inside-openais-forward-deployed-engineer)）。

逆に言えば、**製品の改善につながる道がない FDE は、名前が変わっただけの常駐型の受託開発**になってしまいます。これは、あとで日本の動きを見るときにも大切な見方になります。

### 3-3. 実際の働き方

求人情報やインタビューからは、次のような働き方が見えてきます。

- **お客さまの現場にいる時間が長い**：OpenAI では、勤務時間のおよそ半分をお客さまのところで過ごします。地域による違いもあり、アラブ首長国連邦や日本では約8割、欧米では月に2〜3週間ほどだそうです（[Engineering Leadership](https://newsletter.eng-leadership.com/p/inside-openais-forward-deployed-engineer)）。Anthropic の求人では、出張は勤務時間の25〜50%とされています。
- **仕事の例**：OpenAI の日本での事例として、古いシステムをたくさん抱えるお客さまの、プログラムの刷新が紹介されています。決済などの担当チームとの打ち合わせ、テストの作成、移行のためのプログラムや進み具合を表示する画面の開発を、行ったり来たりする毎日だそうです。
- **求められる力**：国内外の求人をまとめた業界サイトによると、よく求められているのはお客さまとのやりとり、出張や現地での勤務、プログラミング言語の Python です。**大規模言語モデル（LLM）の専門知識を求める求人は32%にとどまる**とされています（[シンオンの FDE 業界デイリー](https://synon.co.jp/article/news/2026-10-15-fde-daily)）。AI に詳しいことよりも、「**お客さまの現場で、実際に動くものを作り上げる力**」が重視されていることが分かります。
- **報酬**：Anthropic のアメリカでの求人は年収28万〜32万ドル、AI 企業の上級 FDE は35万〜55万ドルという集計もあります（[Tunguz さんの分析](https://tomtunguz.com/fde-arms-race/)）。

### 3-4. 「開発は AI に任せて、エンジニアはお客さまと話す仕事になる」のか

FDE について、このように受け止める声もあります。これは、半分は当たっていて、半分は少し違います。

- **当たっているところ**：生成 AI やプログラムを書く AI のおかげで、「作る」手間は大きく減りました。そのため、**3〜5人の少人数**でも、1つの課題を本番まで持っていけるようになっています（富士通の例。くわしくは後で紹介します）。大人数で作る必要が減った分、**何を作るべきかを見極める仕事**の大切さが増しています。
- **少し違うところ**：FDE は「話すだけ」の人ではありません。Palantir の Delta も OpenAI の FDE も、自分でプログラムを書いて本番に載せるエンジニアです。企画や調整を中心に担う人は、Palantir の Echo や Salesforce の Deployment Strategist のように、**別の役割として分けている**会社が多くあります。

まとめると、「お客さまと話すこと」に専念するのではなく、**お客さまの現場で、お客さまと一緒に、AI を道具として使いながら、最後まで作り上げる**仕事だと言えます。

---

## 4. ビジネスモデル：どうやって収益を上げるのか

FDE を取り入れている各社の動きを見ると、収益の上げ方は大きく3つの型に分けられます（この分け方は、国内の FDE 支援会社による[整理](https://fde-consulting.jp/blog/fde-companies-japan)を参考にしています）。

![FDE の3つのビジネスモデル。A の製品定着型（Palantir、Salesforce）は、FDE や Bootcamp を短い期間・少ない金額で始め、業務に組み込まれたあとにソフトウェアの利用料で回収する。粗利率は約80%台。B の AI 企業・クラウド企業の導入会社型（OpenAI、Anthropic、Microsoft、AWS、Google）は、導入会社が導入の支援料を受け取り、本番で動き始めると AI やクラウドの利用料が積み上がる。C の IT 企業・コンサルの成果型（日立、富士通、NTT データ、Accenture）は、3〜5人の少人数で1つの課題を担当し、固定の金額や成果に応じて請求し、協力会社の製品の販売も行う](/images/forward-deployed-engineering/fig6-business-models.png)
*図6　FDE の3つのビジネスモデル*

### 型A：製品定着型（Palantir、Salesforce）

- FDE は**先行投資**という位置づけです。Bootcamp や試験導入は、短い期間・小さな規模で始め、FDE の人件費を最初の段階で回収しようとはしません。
- 業務に組み込まれたあとの**ソフトウェアの利用料**で回収します。報道でも「サービスは製品を使ってもらうための手段で、主な収益源ではない」と説明されています（[Yahoo Finance](https://finance.yahoo.com/markets/stocks/articles/ive-changed-mind-palantir-stock-193500694.html)）。
- この型が成り立つには、**現場での工夫が製品に取り込まれ続けること**が欠かせません。そうでないと、利益の出ない常駐サービスになってしまいます。

Salesforce も同じ型です。AI エージェントの製品「Agentforce」を使い続けてもらうために FDE を置いています。求人では FDE と Deployment Strategist を組み合わせていて、Palantir の Delta と Echo の分け方によく似ています。

### 型B：AI 企業・クラウド企業の導入会社型（OpenAI、Anthropic、Microsoft、AWS、Google）

![導入会社型のお金の流れ。AI 企業は導入会社に出資し、人材と AI モデルを提供して、経営の主導権を持つ。投資会社・金融機関・コンサルも導入会社に出資し、出資先の企業をお客さまとして紹介し、利益の分配を受ける。導入会社はお客さまの企業に FDE を常駐させて本番で使える形にし、導入の支援料を受け取る。お客さまの企業は、API・ChatGPT Enterprise・Claude の利用料を AI 企業に支払う。導入会社は現場で学んだことを AI 企業に伝える](/images/forward-deployed-engineering/fig7-deployco-money.png)
*図7　導入会社型のお金の流れ（各社の発表と報道をもとに作成）*

この型では、FDE の活動から2種類の収入が生まれます。

1. **導入の支援料**：DeployCo や Ode がお客さまから受け取ります。料金は公表されていません。
2. **AI やクラウドの利用料**：本番で動き始めた AI エージェントは、毎日使われ続けます。その利用料が、AI 企業やクラウド企業に継続して入ります。**本当のねらいはこちら**だと考えられます。試験導入の段階では利用量はわずかですが、本番で会社全体に広がれば、利用量は大きく増えるからです。

もう一つの特徴は、**PE ファンドと組んでいる**ことです。

- DeployCo には、投資会社の TPG を中心に19社が出資しています。出資者には、コンサルティング会社の Bain & Company や McKinsey、IT サービス会社の Capgemini も含まれます。OpenAI によると、出資者が投資・支援している企業は世界で**2,000社以上**あります（[OpenAI](https://openai.com/index/openai-launches-the-deployment-company/)）。
- Ode でも、Blackstone や H&F が出資先の企業をお客さまとして紹介します。ただし、それ以外の企業にもサービスを提供します（[CNBC](https://www.cnbc.com/2026/05/04/anthropic-goldman-blackstone-ai-venture.html)）。

PE ファンドにとっては、出資先の企業の業務を AI で改善して、企業の価値を高める手段になります。AI 企業にとっては、最初から2,000社以上のお客さま候補とつながれることになります。なお、DeployCo の出資者には**年17.5%の最低限の利益が保証されている**という報道もあります（[Tunguz さんの分析](https://tomtunguz.com/fde-arms-race/)）。この点は公式の発表では確認できていませんが、もし事実であれば、利益率の高いサービスになると見込んでいることになります。

Microsoft、AWS、Google は自社でクラウドサービスを持っているので、仕組みはもっと単純です。FDE が本番で動かした仕組みは、そのまま **Azure、AWS、Google Cloud の利用**につながります。

### 型C：IT 企業・コンサルの成果型（日立、富士通、NTT データ、Accenture）

自社で AI モデルを持たない IT 企業やコンサルティング会社にとって、FDE は「**人月からの脱却**」の手段になっています。

- **少人数・短期間**：富士通では、3〜5人の FDE が1つの課題に取り組みます（[週刊 BCN](https://www.weeklybcn.com/journal/feature/detail/20260907_217086.html)）。
- **固定の金額や成果にもとづく料金**：富士通は「金額は固定で、内訳は示さない。お客さまが『その金額なら価値がある』と判断すれば契約が成り立つ」と説明しています。NTT データは「人月の単価ではなく、お客さまの課題や得られる成果をもとに価格を決めるサービスのパッケージ」としています（同じ記事より）。
- **協力会社の製品の販売**：富士通は Palantir の製品と、Accenture は Anthropic、OpenAI、Google、Workday などの製品と組み合わせて提供しています。

![従来の人月の受託開発は、IT 部門が要件を決め、大人数で設計・開発し、納品・検収をして、人数 × 期間で請求する。FDE 型は、経営層や事業部門と課題そのものを探し、3〜5人が AI を使って短い周期で試作し、本番稼働と定着まで寄り添い、固定の金額や成果に応じた料金と、製品や AI の利用料で請求する。生成 AI で作る費用が下がり、作業時間で収益を上げる前提が成り立たなくなってきたことが、移り変わりの理由](/images/forward-deployed-engineering/fig8-ningetsu.png)
*図8　人月の受託開発と FDE 型の違い（週刊 BCN の記事などをもとに作成）*

生成 AI でプログラムを作る手間が減ると、「作業時間 × 単価」で料金を決める方式には、**AI を活用するほど売上が減ってしまう**という矛盾が生まれます。日本の大手 IT 企業が FDE に力を入れている大きな理由は、ここにあると考えられます。

### 3つの型の比べ方

| | A. 製品定着型 | B. 導入会社型 | C. 成果型 |
|---|---|---|---|
| 主な例 | Palantir、Salesforce | OpenAI DeployCo、Ode、Microsoft、AWS、Google | 日立、富士通、NTT データ、Accenture |
| FDE の費用 | 先行投資（ほぼ持ち出し） | 導入の支援料で一部を回収 | 固定の金額や成果に応じた料金で回収 |
| 本当の収益源 | ソフトウェアの利用料 | AI やクラウドの利用料 | サービス料と、製品販売の利益 |
| 学んだことの行き先 | 自社の製品 | 自社の AI モデルや製品 | お客さまの社内と、協力会社の製品 |
| 気をつけたい点 | 製品への取り込みが追いつかず、サービス会社のようになる | お客さまが特定の会社から離れにくくなる、投資会社からの利益の要求 | 「人月」の呼び方を変えただけになる |

---

## 5. 各社の取り組み

### 5-1. 海外

| 企業 | 取り組み | ポイント |
|---|---|---|
| **Palantir** | FDE（Delta）と Deployment Strategist（Echo）。AIP Bootcamp | FDE の元祖。FDE は約400〜500人で、社員全体の約12%という推計があります（[Tunguz さんの分析](https://tomtunguz.com/fde-arms-race/)） |
| **OpenAI** | 本体の FDE（約140人）は製品と研究への橋渡しに集中し、たくさんの会社への導入は DeployCo（約200人）で広げる | 人数は2026年8月時点（[Engineering Leadership](https://newsletter.eng-leadership.com/p/inside-openais-forward-deployed-engineer)）。買収した Tomoro のお客さまには Tesco、Virgin Atlantic、Supercell など |
| **Anthropic** | 社内の Applied AI チームの FDE、合弁会社の Ode（代表は Fractional AI の共同創業者 Chris Taylor さん）、Claude Frontier Academy | Ode は「まず Claude を使う」方針ですが、必要に応じてほかの会社の AI も使います。Academy は、Anthropic が自社で雇うのではなく、Accenture、Deloitte、McKinsey などの**協力会社のエンジニア**を育てる取り組みです |
| **Microsoft** | Frontier Company（25億ドル、約6,000人。責任者は前アジア担当プレジデントの Rodrigo Kede Lima さん） | 人員の多くは社内の別部門からの異動。コンサルティング部門を作り直したものだという見方もあります |
| **AWS** | FDE の組織（10億ドル）。45日単位のチームを派遣 | お客さまには Allen Institute、Cox Automotive、NBA、NFL、リコー、Southwest Airlines など |
| **Google Cloud** | 自社での FDE の採用に加え、Accenture と組んで Gemini Enterprise の FDE を1,000人体制に | 日立とも、FDE による「フィジカル AI」の分野で提携しています（2026年6月） |
| **Salesforce** | Agentforce の FDE と Deployment Strategist を組にして配置 | イギリスでは FDE を約95人に倍増させる計画です（[ITPro](https://itpro.com/security/salesforce-doubles-down-on-forward-deployed-engineers-in-uk-agentforce-push)） |
| **Accenture** | Anthropic 専門の部門（約3万人を Claude で研修、2025年12月）、Google との Gemini Enterprise 部隊、Workday との FDE プログラム（2026年10月） | 「FDE」を自社の職種名として強く打ち出すというより、**各 AI 企業の FDE の受け皿**として動いています |

Accenture は自社の AI モデルや主力のソフトウェア製品を持たないため、型A や型B ではなく、**AI 企業ごとに専門の部隊を作り、導入を引き受ける型C** として動いています。2026年に入ってからは、Google や Workday との取り組みの中で「FDE」という言葉も使い始めています。

### 5-2. 日本

| 企業 | 取り組み | ポイント |
|---|---|---|
| **日立製作所** | 2026年度中に、国内で FDE を1,000人規模にする計画。2026年9月3日から「Physical AI FDE Team」によるサービスを開始 | 熟練した人が持つ経験やコツを、AI との対話形式のインタビューで引き出し、AI が扱える形に整理します。Google Cloud と提携しています（[日本経済新聞](https://www.nikkei.com/article/DGXZQOUC183EE0Y6A810C2000000/)、[MONOist](https://monoist.itmedia.co.jp/mn/articles/2609/07/news035_3.html)、[Google Cloud ブログ](https://cloud.google.com/blog/ja/products/gcp/hitachi-and-google-cloud-collaborate-on-the-social-implementation-of-physical-ai-using-fde?hl=ja)） |
| **富士通** | 2026年4月に FDE 事業部を立ち上げ。9月10日に Palantir の **Global FDE Partner** に | Palantir との協業は2020年から続いています。Palantir の経験豊富な FDE が、一定期間富士通のチームに加わります。保守部品の管理で年間約5億円のコスト削減、トラスコ中山では人事異動案を作る手間を約98%削減した事例があります（[富士通](https://global.fujitsu/ja-JP/pr/news/2026/09/10-01)、[週刊 BCN](https://www.weeklybcn.com/journal/feature/detail/20260907_217086.html)） |
| **NTT データ** | 「DataEnablement Suite」で、課題の発見から実装、定着までを一貫して支援 | 人月ではなく、成果をもとにしたサービスのパッケージとして提供しています。地方の電力会社で約130の課題を洗い出した事例があります |
| **NEC など** | 日本経済新聞が報じた「IT 大手4社」として、FDE を増やす方針 | 2026年7〜8月の決算発表で、各社が「人月からの脱却」を掲げました（[日経クロステック](https://xtech.nikkei.com/atcl/nxt/column/18/00001/11956/)）。くわしい内容は有料記事のため確認できていません |
| **電通グループ** | 国内の4社で、約3,000人規模の FDE・FDM（Forward Deployed Manager）の育成を開始（2026年10月14日） | 新しく採用するのではなく、社内の人材を育てて FDE を増やす取り組みです |
| **AI・SaaS 企業** | LayerX、ログラス、AI Shift、JAPAN AI、SB OAI Japan などが FDE の職種を新設・募集 | 国内の FDE の求人は76件・47社（2026年10月10日時点、業界サイトによる非公式の集計） |

日本の特徴は、**大手 IT 企業が「人月からの脱却」（型C）として FDE に取り組んでいる**ことです。海外では AI 企業やソフトウェア会社が中心ですが、日本では大手 IT 企業が中心になりつつあります。また、富士通が Palantir の、日立が Google Cloud の FDE パートナーになっているように、**海外の企業の FDE の受け皿**という面もあります。

---

## 6. これからどうなるのか

### 6-1. 大きな流れになると考えられる理由

1. **課題の中心が「AI の性能」から「AI の取り入れ方」に移った**
   AI モデルどうしの性能の差が小さくなるほど、「どれだけ業務に深く入り込めたか」で差がつくようになります。Ode の技術責任者も「AI モデルの選び方は大事だが、労力の大半はそこにはかからない」と話しています（[TechCrunch](https://techcrunch.com/2026/07/15/anthropic-blackstone-bet-the-next-trillion-dollar-ai-business-is-implementation-not-models/)）。
2. **作る手間が減り、少人数でも本番まで届けられるようになった**
   以前なら大人数で何か月もかかったシステムの連携を、AI を使いこなす少人数のチームで、短い期間で進められるようになりました。これによって、Palantir のやり方をほかの会社でもまねできるようになりました。
3. **お金の流れと合っている**
   AI 企業は利用料、クラウド企業はクラウドの利用、IT 企業は人月からの脱却というように、立場はそれぞれ違っても、「本番で使われ続けること」が全員の収益につながります。

### 6-2. 気をつけたい点

1. **サービス会社のようになってしまうおそれ**
   ソフトウェア企業の粗利率は70〜80%台なのに対し、人が作業するサービス企業は30〜40%ほどだと言われます。FDE が現場で得た知見が製品に生かされなければ、利益の出にくいコンサルティング会社のようになってしまいます。「FDE は導入コンサルタントを言い換えただけ」という厳しい見方も根強くあります（[n47](https://www.n47.com/insights/the-forward-deployed-engineer-dilemma)）。
2. **特定の会社から離れにくくなること**
   FDE が特定の会社の AI の上に業務の仕組みを作り込むほど、ほかの会社に乗り換えるのは難しくなります。提供する側にとっては強みになりますが、お客さまにとっては負担になることもあります（[Tunguz さんの分析](https://tomtunguz.com/fde-arms-race/)）。お客さまの側では、作った仕組みや知見が**自社に残るのか、提供する会社の製品の改善に使われるだけなのか**を確かめておくことが大切です。
3. **人材の取り合い**
   高い開発力と、業務への理解と、人とのやりとりの力を兼ね備えた人は多くありません。Anthropic が自社で雇うのではなく、協力会社のエンジニア1万人を育てようとしているのも、この人材不足を反映していると考えられます。
4. **呼び方を変えただけにならないか**
   3章で見たように、FDE の大切な点は「本番の成果に責任を持つこと」と「学んだことを製品の改善につなげること」です。これまでの常駐型の受託開発を、そのまま FDE と呼び替えただけでは、どちらも満たせません。富士通の「金額は固定で、内訳は示さない」という例のように、**契約の形まで変わっているか**が見分けるポイントになります。

### 6-3. エンジニアにとっての意味

FDE の広がりから見えてくる、エンジニアに求められることを3つにまとめます。

- **作った量ではなく、成果で評価される働き方**
  AI がプログラムを書く部分を多く担うようになるほど、「何を、なぜ作るのか」を考えることと、「本番で動かし続けること」の価値が高まります。
- **業務の知識**
  OpenAI は、その業界にくわしい人と、あえて別の業界から来た人を組み合わせて FDE のチームを作っていると話しています。
- **現場で学んだことを、ほかでも使える形にまとめる力**
  一度きりの解決で終わらせず、次のお客さまでも使える形に整えられるかどうかが、従来の受託開発と FDE を分ける点です。

FDE が目指しているのは、**お客さまの現場に入り、AI を使いながら成果が出るまで作り上げ、その経験を次に生かせる形で残す**という働き方だと言えます。

---

## まとめ

- FDE は、Palantir が2000年代後半に生み出した職種です。現場で解決したことを製品に取り込み、ソフトウェアの利用料で収益を上げる仕組みでした。
- 2025年に、生成 AI の試験導入の多くが成果につながっていないことが分かり、2026年には OpenAI、Anthropic、Microsoft、AWS、Google などが専用の組織や会社を作って、大きな投資をしています。10月22日の OpenAI のイベントもその流れの中にあります。
- 従来の受託開発との違いは、**本番の成果に責任を持つこと**と、**学んだことを製品や AI の改善につなげる道があること**です。
- 収益の上げ方は、製品の利用料（型A）、導入の支援料と AI・クラウドの利用料（型B）、固定の金額や成果に応じた料金（型C）の3つに分かれます。日本の大手 IT 企業は、型C を「人月からの脱却」の手段にしようとしています。
- FDE を名乗る取り組みが本当に FDE と言えるかは、**契約の形**と**学んだことの行き先**を見ると判断しやすくなります。

FDE は、まさに今かたちが作られている途中の働き方です。この記事は第1版として、今後の各社の動きに合わせて更新していきます。

---

## 参考資料

### 企業の発表・公式資料
- OpenAI「企業向け AI のためのフォワード・デプロイド・エンジニアリング」 https://openai.com/ja-JP/business/forward-deployed-engineering/
- OpenAI「OpenAI launches DeployCo to help businesses build around intelligence」（2026年5月11日） https://openai.com/index/openai-launches-the-deployment-company/
- Anthropic「Claude Frontier Academy」 https://anthropic.com/news/claude-frontier-academy
- Anthropic「Anthropic and Accenture partnership」 https://anthropic.com/news/anthropic-accenture-partnership
- Blackstone プレスリリース https://www.blackstone.com/news/press/anthropic-partners-with-blackstone-hellman-friedman-and-goldman-sachs-to-launch-enterprise-ai-services-firm/
- Palantir 2026年4〜6月期 決算発表 https://www.sec.gov/Archives/edgar/data/0001321655/000132165526000039/a2026q2ex991pressrelease.htm
- Palantir 2024年度 年次報告書（10-K） https://www.sec.gov/Archives/edgar/data/1321655/000132165525000022/pltr-20241231.htm
- Palantir Blog「A Day in the Life of a Palantir Deployment Strategist」 https://blog.palantir.com/a-day-in-the-life-of-a-palantir-deployment-strategist-951cb59a5a96
- 富士通 プレスリリース（2026年9月10日） https://global.fujitsu/ja-JP/pr/news/2026/09/10-01
- Google Cloud ブログ「日立と Google Cloud、FDE によるフィジカル AI の社会実装」 https://cloud.google.com/blog/ja/products/gcp/hitachi-and-google-cloud-collaborate-on-the-social-implementation-of-physical-ai-using-fde?hl=ja

### 報道・分析
- Nabeel S. Qureshi「Reflections on Palantir」 https://nabeelqu.substack.com/p/reflections-on-palantir
- Engineering Leadership「Inside OpenAI's Forward Deployed Engineer」 https://newsletter.eng-leadership.com/p/inside-openais-forward-deployed-engineer
- Tomasz Tunguz「The FDE Arms Race」 https://tomtunguz.com/fde-arms-race/
- TechCrunch（Ode with Anthropic） https://techcrunch.com/2026/07/15/anthropic-blackstone-bet-the-next-trillion-dollar-ai-business-is-implementation-not-models/
- CNBC（Anthropic の合弁会社） https://www.cnbc.com/2026/05/04/anthropic-goldman-blackstone-ai-venture.html
- GeekWire（Microsoft Frontier Company） https://geekwire.com/2026/microsoft-announces-2-5b-frontier-company-to-embed-ai-engineers-inside-customers
- Directions on Microsoft https://www.directionsonmicrosoft.com/microsoft-launches-its-own-forward-deployed-engineering-unit-the-frontier-company/
- The Next Web（AWS） https://thenextweb.com/news/aws-1bn-forward-deployed-engineers-enterprise-ai/
- ITPro（Salesforce） https://itpro.com/security/salesforce-doubles-down-on-forward-deployed-engineers-in-uk-agentforce-push
- Fortune（MIT NANDA「The GenAI Divide」） https://fortune.com/2025/08/18/mit-report-95-percent-generative-ai-pilots-at-companies-failing-cfo
- Bloomberry「I analyzed 1,000 forward deployed engineer jobs」 https://bloomberry.com/blog/i-analyzed-1000-forward-deployed-engineer-jobs/
- n47「The Forward Deployed Engineer dilemma」 https://www.n47.com/insights/the-forward-deployed-engineer-dilemma
- 日本経済新聞「IT大手4社、『FDE』拡充急ぐ 日立は26年度内に国内1000人」 https://www.nikkei.com/article/DGXZQOUC183EE0Y6A810C2000000/
- 日経クロステック「IT大手4社、脱・人月へFDE拡充に意欲」 https://xtech.nikkei.com/atcl/nxt/column/18/00001/11956/
- 週刊 BCN「SIer における『FDE』の意義は 富士通と NTT データの実践から考える」 https://www.weeklybcn.com/journal/feature/detail/20260907_217086.html
- MONOist「日立が HMAX の新ソリューション群を発表」 https://monoist.itmedia.co.jp/mn/articles/2609/07/news035_3.html
- シンオン「FDE 業界デイリー（2026年10月15日）」 https://synon.co.jp/article/news/2026-10-15-fde-daily
- FDE Consulting「FDE を導入している企業一覧【2026年】」 https://fde-consulting.jp/blog/fde-companies-japan

:::message alert
この記事の数字のうち、DeployCo の企業価値と出資者への最低限の利益、Palantir の FDE の人数、各社の FDE の年収は、投資家のブログや求人の集計といった報道・分析によるもので、公式の発表では確認できていません。また、FDE の求人の増え方は、集計する会社によって期間や数え方が違います。
:::
