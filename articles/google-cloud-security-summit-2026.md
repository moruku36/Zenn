---
title: "AI時代におけるサイバーセキュリティ変革：Google Cloud Security Summit 2026 基調講演 レポート"
emoji: "🛡️"
type: "tech"
topics: ["googlecloud", "security", "ai", "cybersecurity", "secops"]
published: true
---

## 1. オープニング：サイバー空間の「平時」から「有事」への質的転換

### 1.1 導入：現代の脅威環境と日本企業が直面する構造的危機

現代のサイバー脅威環境は、従来の「平時」を前提とした防御モデルが完全に形骸化し、常に実効的な攻撃に晒される「有事」の状態へと質的に不可逆な転換を遂げました。連日のように報じられる大規模な情報漏洩やランサムウェア被害、社会インフラの停止は、サイバー攻撃が単なる一部の技術的脅威ではなく、あらゆる産業領域における最優先の経営危機であることを突きつけています。経営陣およびセキュリティリーダーにとって、セキュリティ投資の再定義と抜本的な運用改革は、企業の事業継続性と市場信頼を維持するための最重要命題です。

現在、日本企業が直面している地政学的・構造的リスクは、以下の3つの要素に整理できます。

* **AIを用いた攻撃の本格化と高度化**: 生成AIの悪用により、フィッシングメール等における従来の不自然な文脈や文法エラーは消失し、極めて高度なソーシャルエンジニアリングが自動展開されています。未知の脆弱性が発見されてから実際の攻撃行動へ移るまでの時間（Time to Exploit）は劇的に短縮され、日本の上場企業における情報漏洩事故は過去最悪レベルに達しました。ダークウェブ上のリークサイトにおける日本企業の被害件数も急増しているほか、金銭目的のサイバー犯罪のみならず、国家背景を持つ高度持続的脅威（APT）グループの暗躍や、リモートワーカーになりすまして企業内部ネットワークへ直接侵入を図る精巧な工作が現実化しています。

* **日本企業固有の構造的弱点（サプライチェーンリスク）**: 日本の産業構造は、広範な外部委託、複雑な多層サプライチェーン、およびグローバル展開に伴う海外拠点との密接なネットワーク接続に強く依存しています。この構造的特性ゆえに、どれほど本社の中核システムが堅牢であっても、セキュリティ対策の手薄な委託先や海外子会社が「最初の足がかり（Initial Access）」として侵害され、最終的に本社の中核をなす物流システムや工場の生産ライン全体が停止に追い込まれる「横断的破綻」のパターンが多発しています。

* **「能動的サイバー防御」関連法の施行と法的・制度的要請**: 日本国内において[「能動的サイバー防御」（サイバー対処能力強化法）](https://www.cas.go.jp/jp/seisaku/cyber_anzen_hosyo_torikumi/index.html)に関連する法制度が施行され、[国家サイバー統括室（NCO）](https://www.cyber.go.jp/)を中心とした体制が整備されたことで、特定社会基盤事業者やサプライチェーン全体に対し、脅威の迅速な「検知」と「報告」が法的な義務として課されるようになりました。これにより、従来の「侵入を水際で未然に防ぐ（境界防御）」という幻想を捨て、「侵入されることを前提（Assume Breach / 侵入前提）」としたうえで、数分・数秒単位で異常を検知・特定・報告・対処できる実効的なアクティブ・ディフェンス体制への転換が制度的にも不可欠となっています。

#### 従来型運用の限界：なぜ「人海戦術」と「事後対応」は破綻するのか

これまで日本企業の防御を支えてきた「人海戦術によるアラート確認」や、インシデント発生後に調査を開始する「事後対応（リアクティブ運用）」は、AI時代のスピード感の前では完全に破綻しています。人間がログを目視で解析する速度や従来の多層承認フローでは、マシンスピードで自律展開される現代の攻撃を食い止めることは不可能です。

「日本語の壁による防衛」という地理・言語的アドバンテージが消滅し、境界防御が崩壊した今、運用モデルの刷新を怠る企業は、事業停止による直接的損害のみならず、法的制約に伴う制裁やサプライチェーンからの排除という壊滅的な経営リスクに晒されることになります。

攻撃の手法そのものがAIによって根本から書き換えられている実態を正しく解き明かすため、次に攻撃側にもたらされた「3つのS」の変革モデルを検証します。

### 1.2 攻撃の次元を変える「3つのS」（Scale, Speed, Sophistication）の変革モデル

[Google Cloud Security](https://cloud.google.com/security) 責任者の村山正氏、および [Google Threat Intelligence Group（GTIG）](https://cloud.google.com/security/products/threat-intelligence)のジョン・ハルトキスト（John Hultquist）氏が提示した通り、生成AIの台頭は攻撃側に「3つのS」と呼ばれる構造的なイノベーションをもたらしました。その定量的・定性的影響と、防御側に与える構造的インパクトを以下のように対比・構造化します。

| 変化の軸 (3つのS) | 従来の脅威モデル | AI時代の脅威モデル | 防御側に与える構造的インパクト | 
| ----- | ----- | ----- | ----- | 
| **Scale（規模）** | 手動・限定的ターゲットに対する攻撃 | 自律型エージェントによる持続的・連続的無差別攻撃 | アラート過多（Alert Fatigue）によるSOCの機能不全と、動的生成によるシグネチャ回避 | 
| **Speed（スピード）** | 人間の作業時間を前提とした対応 | マシンスピードによる侵入・横断・目的遂行 | パッチ検証・適用までの「猶予時間（Grace Period）」の完全消滅 | 
| **Sophistication（高度化）** | 一部トップ層の専有技術（脆弱性探査・分析） | AIによるスキルの民主化（スクリプトキディの高度化） | 未熟な攻撃者によるゼロデイ級侵入手法および精巧な偽装工作の行使 | 

## 2. 攻撃者のリアルと脅威インテリジェンス最前線

### 2.1 AI悪用のリアルと攻撃者優位の経済構造

[Google Threat Intelligence Group](https://cloud.google.com/security/products/threat-intelligence) のチーフアナリストであるジョン・ハルトキスト氏の分析によると、現代のサイバー攻撃者はAIを単なる概念実証や実験段階としてではなく、自身の攻撃プロセス全体を効率化・高速化するための「日々の生産性向上プラットフォーム」として全面的に実務へ組み込んでいます。

#### 自律型攻撃とスケール（規模）の拡大

日本国内の技術系企業を対象として実際に発生した侵害事例では、「X-Strike AI」をはじめとするAIエージェント対応のレッドチームツール（攻撃検証ツール）が不法に悪用されました。この攻撃フレームワークの核となっているのが、AIモデルが外部ツールやデータソース、記憶領域と安全にやり取りを行うための標準プロトコル [**MCP（Model Context Protocol）**](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro) です。

```text
[攻撃用AIエージェント] ───── (MCP経由でツール連携) ─────> [標的ネットワークの自動探査]
       │                                                      │
       │                                                      │
       └─── <── (標的の応答・拒否反応から自律的に試行錯誤/学習) ───┘
```

攻撃用AIエージェントは [MCP](https://modelcontextprotocol.io/specification/2026-07-28) を通じてメモリ（記憶層）を保持し、標的システムの応答や拒否ログからリアルタイムに反復学習・実験を行います。攻撃者がキーボードの前に座って直接操作しなくても、AIエージェントが自律的にプログラミングの修正、侵入パスの探索、認証回避の試行錯誤を繰り返し、最終的な侵入成功までを自律完遂するプロセスが確立されています。

#### AIコストの非対称性とアンダーグラウンド市場の構造

AIの活用において、防御側（企業）と攻撃側（犯罪組織）の間には深刻な「コスト構造の非対称性」が存在します。企業側は正規のAIライセンス費用、コンプライアンス管理費、インフラ構築費を正規に負担してAIを導入しますが、攻撃側は侵入先のサーバー計算資源、改ざんしたインフラ、あるいは不正窃取したSaaSアカウントやAPIトークンを無断悪用（タダ乗り）することで、インフラコストを実質ゼロに抑えています。

さらに、ダークウェブをはじめとするアンダーグラウンド市場では、盗み取られたクレデンシャル（認証情報）や侵害済みアカウントへのアクセス権が、正規価格の「最大99%引き」という著しいディスカウント価格で大量取引されています。自前での技術開発費やインフラ費を支払うことなく最先端のAI計算リソースを無限に行使できるため、攻撃側には防御側を圧倒する構造的な経済的アドバンテージ（経済優位性）が生じています。

#### シグネチャ回避と北朝鮮工作員によるプロセス自動化

従来のセキュリティ対策（AVやEDR等）は、Known（既知）のマルウェアのパターン（シグネチャ）を特定・抽出して遮断することに依存していました。しかし、AI時代の攻撃者はソースコードの動的書き換えや新しい通信インフラの確保を即座に行えるため、同じマルウェアを再利用する必要がありません。コストゼロで毎回異なる未知の変種（ポリモーフィック・コード）を大量生成できるため、シグネチャ依存型の防衛網は完全に機能不全へ追い込まれています。

また、北朝鮮の工作員によるAI悪用事例では、単なる偽造身分証明書（ID）の作成にとどまらず、企業への潜入工作プロセス全体の完全自動化が達成されています。

1. **偽造ID・履歴書の自動生成**: AIを用いて精巧な人物プロフィールを作成。

2. **管理シート（Excel等）の自動構築**: ターゲット企業や求人状況を追跡する管理ファイルを生成。

3. **求人探査・応募の自律実行**: AIエージェントが人間を介さずWeb上の求人情報をスクレイピングし、自律的に応募手続きを完遂。

4. **組織内部への潜入**: 採用されることで内部関係者の権限を取得し、組織内部からのデータ窃取や不正操作を自動化。

攻撃「規模」の爆発的拡大に続き、防御体制の前提を根底から揺るがしているのが、マシンスピードによる攻撃展開の「加速」です。

### 2.2 マシンスピードの到来とサイバー犯罪者の高速展開

#### 人間のスピード vs マシンスピードのギャップ

サイバーインシデント対応における従来の標準的枠組みとして、検知1分・分析10分・封じ込め60分を目指す「1-10-60モデル（1060モデル）」や、重篤な脆弱性公表後「48〜72時間以内」にパッチを適用する運用サイクルが存在しました。しかし、これらはすべて「人間の手作業の速度」を前提としたものであり、マシンスピードで展開される現代の攻撃に対しては猶予時間として機能しません。

今日、ベンダーから修正パッチが公開された後、攻撃者がそのパッチをリバースエンジニアリングしてPoC（概念実証コード）を作成し、実際の攻撃に投入するまでの時間は「数時間」にまで短縮されています。従来与えられていた48〜72時間の検証期間は消失しており、人間主導の変更管理プロセスでは物理的に追いつけない現実が突きつけられています。

#### サイバー犯罪者の行動モデルの変化：スピードの兵器化

国家背景を持つスパイクラスの攻撃者は、長期間検知されずに「静かに潜伏（Espionage）」することを最優先するため、あえて目立つ高速動作を避ける傾向にあります。これに対し、金銭奪取を目的とするサイバー犯罪グループ（ランサムウェア・アフィリエイト等）は、「スピード」そのものを防御側を無力化するための最大の兵器として活用しています。

犯罪者は侵入に成功した瞬間、セキュリティチームに検知・遮断される前に、AIを用いてネットワーク内をマシンスピードで横断（ラテラルムーブメント）し、数時間で重要データを窃取、あるいはランサムウェアを一斉展開します。実際に起きた大規模侵害事件のデータでは、AIモデルを介した攻撃の開始から、全工程の遂行、被害確定までがわずか **「6時間」** で完了したケースが確認されています。

### 2.3 スキルの高度化と日本市場における「言語バリア」の崩壊

#### ハッキングスキルの民主化と「Big Sleep」

かつて複雑な脆弱性の発見やリバースエンジニアリング、ゼロデイ攻撃コードの開発は、世界でも限られた高度なセキュリティ研究者にしか不可能な領域でした。しかし、フロンティアAIモデルの知能向上により、これらの高度な分析ノウハウが「民主化」され、深い技術知識を持たないスクリプトキディであっても高度な攻撃を行える環境が整ってしまいました。

Google自身も、[Google DeepMind](https://deepmind.google/) と [Google Project Zero](https://projectzero.google/) の共同によるゼロデイ脆弱性の先回り発見枠組みとして「[Big Sleep](https://projectzero.google/2024/10/from-naptime-to-big-sleep.html)」プロジェクト（[Google Cloud Blog 解説記事](https://cloud.google.com/blog/products/identity-security/cloud-ciso-perspectives-our-big-sleep-agent-makes-big-leap)）を運用しており、AIを用いてオープンソースソフトウェア等の非常に複雑な脆弱性を自動検知・パッチ作成まで完了させる技術を実証しています。しかし、この卓越した脆弱性検出能力は攻撃者側にとっても表裏一体のツールであり、ゼロデイ探索の自動化が急展開しています。

#### 日本語の「言語バリア」消失とソーシャルエンジニアリングの精緻化

長年、日本企業や国内の組織は「日本語という特殊かつ複雑な言語の壁」によって、海外からのサイバー犯罪被害から一定程度保護されてきました。不自然な文法や単語の誤用を含むフィッシングメールは、ユーザー側で容易に見分けることが可能であったためです。

しかし、生成AIの高度な自然言語処理・翻訳能力により、この「言語バリア」は完全に消失しました。現在では、文脈に即した極めて自然な日本語による多段階スピアフィッシングが自動生成されるだけでなく、ディープフェイク技術を用いたリアルタイム音声・動画合成を組み合わせ、企業の経営幹部や取引先になりすます精巧なソーシャルエンジニアリングが容易に実行されています。

## 3. Google Cloud Securityの統合防御アプローチ

### 3.1 導入：フロンティアAIとセキュリティ専門性の融合

攻撃側がAIを駆使して「Scale, Speed, Sophistication」を進化させる中、防御側が対抗するための唯一の道は、単なるセキュリティツールの継ぎ足しではなく、「最先端のフロンティアAI技術」と「世界最高峰のサイバーセキュリティ専門性」を極限まで融合させることです。[Google Cloud Security](https://cloud.google.com/security) / [Mandiant](https://cloud.google.com/security/mandiant) CTOのスティーブ・レザン氏が強調するように、Googleはこの双方の強みを統合した独自のアプローチを提示しています。

#### フロンティアAIとセキュリティの融合構造

Googleは、世界屈指のAI研究機関である「[Google DeepMind](https://deepmind.google/)」が開発するセキュリティ特化型フロンティアモデル（Gemini 4 Argon等）と、年間数十万時間に及ぶ最前線のインシデント対応で培われた「[Mandiant](https://cloud.google.com/security/mandiant)」や「[VirusTotal](https://www.virustotal.com/)」の脅威インテリジェンス（[Google Threat Intelligence](https://cloud.google.com/security/products/threat-intelligence)）を単一の基盤に統合しています。

```text
┌────────────────────────────────────────────────────────┐
│                   Google Cloud Security                │
├──────────────────────────┬─────────────────────────────┤
│   Google DeepMind AI     │   セキュリティインテリジェンス│
│ (Gemini 4 Argon 等)      │ (Mandiant / VirusTotal 等)  │
└─────────────┬────────────┴──────────────┬──────────────┘
              └──────────────┬────────────┘
                             ▼
     ┌──────────────────────────────────────────────┐
     │  ・AI Threat Defense / CodeMender            │
     │  ・Google SecOps (自律型SOC)                 │
     │  ・Wiz (横断的クラウドセキュリティ)          │
     └──────────────────────────────────────────────┘
```

この融合から生まれた「[Google AI Threat Defense](https://cloud.google.com/security/ai-threat-defense)」（[発表ブログ](https://cloud.google.com/blog/products/identity-security/introducing-google-ai-threat-defense)）および「[CodeMender](https://cloud.google.com/security/codemender)」（[プレビュー公開記事](https://cloud.google.com/blog/products/identity-security/find-and-fix-software-vulnerabilities-with-codemender)）は、ソースコードレポジトリを自動検査し、攻撃者に脆弱性を悪用（エクスプロイト）される前に、AIが自動的に修正パッチ（フィックス）を生成・検証・提示・適用する環境を実現しています。

基盤モデルの進化を運用現場に直結させ、SOCにおける対応スピードを異次元の領域へ引き上げる具体的なソリューションが、次に解説する [Google SecOps](https://cloud.google.com/security/products/security-operations) の自律型エージェント（[Agentic SOC](https://cloud.google.com/solutions/security/agentic-soc)）です。

### 3.2 自律型SOCの実現：Google SecOpsと革新的AIエージェント

[Google SecOps（Google Security Operations）](https://cloud.google.com/security/products/security-operations)（[公式ドキュメント](https://docs.cloud.google.com/chronicle/docs/secops/secops-overview)）は、自然言語（日本語）による直感的な検索・分析、自動化された検知ルールの生成、およびペタバイト級ログの即時相関分析を可能にする次世代自律型SOCプラットフォームです。

Google SecOps内に実装された2つの自律型AIエージェントは、従来のSOC運用を劇的に変革します。

#### 1. Triage & Investigation Agent（調査エージェント）

SOC運用において最もリソースを消費するのは、発生した大量のアラートに対して「何が起きたのか」を調査・確認（Investigation）するプロセスです。従来、熟練のアナリストがログを追い、相関関係を特定するのに平均「30分」を要していたこの工程を、[Triage & Investigation Agent](https://docs.cloud.google.com/chronicle/docs/secops/triage-investigation-agent) はわずか **「60秒（1分）」** で完了させます。

高度な文脈理解をAIが担うことで、アナリストを「アラート疲れ（Alert Fatigue）」から解放し、SOCの人手不足に対する劇的な処方箋を提供します（10人体制のSOCが、実質300人規模の処理能力に相当する対応力を獲得します）。

#### 2. Detection Engineering Agent（検知エンジニアリングエージェント）

新しい攻撃手法や脅威情報が世に出た際、自社環境での検知可否を評価し、即座にルール化するプロセスを、[Detection Engineering Agent](https://docs.cloud.google.com/chronicle/docs/secops/agentic-detection-engineering)（[Public Preview 発表ブログ](https://security.googlecloudcommunity.com/community-blog-42/announcing-public-preview-of-the-google-security-operations-detection-engineering-agent-8178)）はわずか **「90秒」** で完結させます。そのワークフローは以下の5つのステップで構成されます。

```text
【Step 1: 脅威情報の入力】
  新しい攻撃手法の解説記事やURL、レポートを自然言語で指示入力。
          │
          ▼
【Step 2: MITRE ATT&CKマッピング】
  AIが内容を分解・構造化し、MITRE ATT&CKフレームワークの各攻撃ステージにマッピング。
          │
          ▼
【Step 3: 合成ログ（Synthetic Logs）の自動生成】
  対象の攻撃が実際に実行された際にシステムが出力すべき特有のログデータをAIが自動作成。
          │
          ▼
【Step 4: 本番環境模擬テスト】
  攻撃自体を撃ち込むことなく、合成ログを既存の本番環境ルールと照合。
  現行の防衛網で検知可能かシミュレーション（例：9つのステップ中6つ検知、3つ未検知と評価）。
          │
          ▼
【Step 5: 検知ルールの自動生成と適用】
  未検知だった3つのステップに対し、最適化された検知ルール（YARA-L等）をAIが自動生成して即時適用。
```

### 3.3 クラウド全体を網羅する横断的セキュリティ：Wiz買収とセキュリティの民主化

#### Wiz買収の戦略的背景

[Google史上最大級の買収となったWizの統合](https://cloud.google.com/blog/products/identity-security/google-completes-acquisition-of-wiz)は、マルチクラウド環境（GCP, AWS, Azure等）における開発（Dev）からランタイム（Ops）に至る全工程のサイバーリスクを、グラフレイヤで一元管理するための画期的なイノベーションです。市場における信頼の裏付けとして、Fortune 100企業の65%が [Wiz](https://www.wiz.io/) を採用しており、クラウドネイティブ・セキュリティのデファクトスタンダードとして位置づけられています。

#### サイロ化の解消と「ゼロ・クリティカル」の実現

従来のクラウドセキュリティは、「開発」「インフラ」「アプリケーション」「運用」の各ドメインごとにツールが分断（サイロ化）され、膨大な誤検知（ノイズ）が発生していました。[Wiz](https://www.wiz.io/) はこれらを単一の構成グラフ（Wiz Security Graph）として統合し、実際の脅威となり得る「攻撃経路（アタックパス）」をエンドツーエンドで可視化します。

表面的なリスクに惑わされず、真に致命的なアタックパスのみを優先順位付けして自動修復へ導くことで、Wizを導入した顧客企業の実に **50%が「クリティカルリスクゼロ（Zero Critical Risk）」** という極めて高いセキュリティ水準の維持に成功しています。

#### セキュリティの民主化（全員の責任化）

Wizのもう一つの成果は、全利用者の実に50%がセキュリティ専門チーム以外の「開発者（デベロッパー）」や「インフラ担当者」である点です。Wizから直感的なリスク評価と具体的な修正コードが提示されるため、開発者自身が日常のワークフロー内で自律的な自己修復（Self-Remediation）を行っています。一部の専門チームにセキュリティを押し付けるのではなく、組織全体に権限と責任を分散する「セキュリティの民主化」が結実しています。

### 3.4 統合プラットフォームの導入成果と実績

#### 防御側のアドバンテージ：内部コンテキスト（Context）の保有

攻撃者が外部からターゲットの内部構造を手探りで探索するのに対し、防御側（ディフェンダー）は自社システムの詳細な設定情報（コンフィグレーション）、クラウド資産（アセット）、ログ、コードレポジトリ、権限設定といった膨大な「内部コンテキスト」を独占的に保有しています。

このコンテキストをAIモデルに網羅的にフィードすることで、AIの分析精度と判断スピードを極限まで高めることが可能となり、攻撃者に対して決定的な構造的優位性を確立できます。

#### コスト最適化を支える「Data Base Program」

通常、複数のセキュリティツールにログを流し込むと、その都度データの取り込み（インジェスト）費用が発生し、防御側のコストを圧迫します。[Google Cloud Security](https://cloud.google.com/security) では「Data Base Program」を提供しており、1度データをインジェストすれば、追加のデータ取り込み費用を支払うことなく、[Google SecOps](https://cloud.google.com/security/products/security-operations) や [Wiz](https://www.wiz.io/) など複数の統合ツール群で自由にデータを活用できます。これにより、経済面においても防御側に極めて高いアドバンテージをもたらします。

#### 定量的な導入効果の実績

[Google Cloudの統合セキュリティプラットフォーム](https://cloud.google.com/customers)を導入した企業における定量成果は以下の通りです。

* [**Innersloth**](https://www.innersloth.com/)**（『[Among Us](https://www.innersloth.com/games/among-us/)』開発元等）の事例**:

  * 平均検出時間（[MTTD](https://www.wiz.io/academy/detection-and-response/mttd-and-mttr)）を **99.x%短縮**。

  * 平均修復時間（[MTTR](https://www.wiz.io/academy/detection-and-response/mttd-and-mttr)）を **98%短縮**。

* [**Fiserv**](https://www.fiserv.com/)**（大手金融サービス）の事例**:

  * 資産管理およびリスク対応における業務負荷を **75%軽減**。

* [**Mandiant**](https://cloud.google.com/security/mandiant) **のインシデント対応実績と Retainer の価値**:

  * 年間50万時間（通常換算で莫大なリソース）に及ぶインシデント調査ノウハウをプラットフォームへリアルタイム反映。

  * 有事発生後に契約を結ぶ時間的猶予がない現代において、平時のうちにインシデント即応契約（[Mandiant Retainer](https://cloud.google.com/security/consulting/mandiant-retainer)）を締結しておくことで、インシデント発生の瞬間に即座にエキスパートチームを投入し、被害を最小限に抑える先手の対応体制を確立。

## 4. 専門用語・サービス完全解説（注釈＆参照ガイド）

### 4.1 導入：本レポートにおける学習インフラの位置付け

本セクションは、本レポートの読者や専門知識を持たないビジネスパーソン、エンジニア初学者が、最新のサイバーセキュリティ技術や用語をいつでも即座に参照・理解できるようにするための「学習インフラ」として設計されています。

基調講演内で言及された高度な技術概念およびプロダクト・サービス群について、その定義と「なぜ本講演で注目されたのか」という実務的文脈、ならびに公式リファレンスURLを付記して掲載します。

### 4.2 主要セキュリティ・IT専門用語解説

* **3つのS（Scale, Speed, Sophistication）**

  * **定義**: 生成AIがサイバー攻撃の性質にもたらした「規模の爆発的拡大」「マシンスピードへの加速」「攻撃手法の高度化とスキルの民主化」という3つの変化軸。

  * **基調講演での注目ポイント**: 従来の人間主導・境界防御型モデルが破綻した根本原因として、村山氏およびハルトキスト氏により提示された。

* **能動的サイバー防御（サイバー対処能力強化法）**

  * **定義**: 重要インフラや安全保障上の懸念を生じさせるサイバー攻撃に対し、兆候段階で先んじて検知・分析・無害化等の措置を講じる日本の法制度および安全保障枠組み。

  * **基調講演での注目ポイント**: 特定社会基盤事業者やサプライチェーン全体に対し、侵入前提（Assume Breach）での迅速な異常検知と報告が法的義務化された背景として言及された。

  * **関連URL**: [内閣官房 サイバー安全保障に関する取組](https://www.cas.go.jp/jp/seisaku/cyber_anzen_hosyo_torikumi/index.html) / [国家サイバー統括室（NCO）](https://www.cyber.go.jp/)

* **MCP（Model Context Protocol）**

  * **定義**: AIモデルが外部のツール、データソース、および記憶領域（メモリ）と安全かつ標準化された方法でやり取りを行うためのオープン標準プロトコル。

  * **基調講演での注目ポイント**: 日本の技術系企業を襲った攻撃ツール「X-Strike AI」において、AIエージェントに反復学習と探査機能を持たせる基盤技術として悪用例が紹介された一方、[Google SecOps](https://docs.cloud.google.com/chronicle/docs/secops/event-simulation) 等の防御側エージェント連携にも活用されている。

  * **関連URL**: [Model Context Protocol 公式ドキュメント](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro) / [MCP 仕様書](https://modelcontextprotocol.io/specification/2026-07-28)

* **PoC（Proof of Concept / 概念実証コード）**

  * **定義**: 特定の脆弱性が実際に攻撃可能であることを証明するために作成される検証用プログラム。

  * **基調講演での注目ポイント**: 修正パッチの公開からパッチをリバースエンジニアリングしてPoCが作成されるまでの時間が「数時間」に短縮され、従来のパッチ適用猶予（48〜72時間）が消滅した。

* **ゼロデイ（Zero-day）脆弱性 / Big Sleep**

  * **定義**: 修正パッチが開発・配布される前、または発見された当日に悪用される未知の脆弱性。

  * **基調講演での注目ポイント**: [Google DeepMind](https://deepmind.google/) と [Google Project Zero](https://projectzero.google/) によるAIエージェント「[Big Sleep](https://projectzero.google/2024/10/from-naptime-to-big-sleep.html)」等により自動発見が進む一方、攻撃者側もAIを用いてゼロデイ探査を高速化させている点が指摘された。

  * **関連URL**: [Google Project Zero: From Naptime to Big Sleep](https://projectzero.google/2024/10/from-naptime-to-big-sleep.html) / [Google Cloud Blog: Big Sleep agent](https://cloud.google.com/blog/products/identity-security/cloud-ciso-perspectives-our-big-sleep-agent-makes-big-leap)

* **SOC（Security Operations Center）**

  * **定義**: 組織のネットワークやシステムを24時間態勢で監視し、サイバー攻撃の検知、分析、対応を一元的に行う専門組織・拠点。

  * **基調講演での注目ポイント**: アラート過多と人材不足に悩むSOCの現場に対し、[Google SecOps](https://cloud.google.com/solutions/security/agentic-soc) のAIエージェントが調査時間を30分から60秒へ縮小させる解決策が示された。

  * **関連URL**: [Agentic SOC | Google Cloud](https://cloud.google.com/solutions/security/agentic-soc)

* **SOAR（Security Orchestration, Automation and Response）**

  * **定義**: 異なるセキュリティツール間の連携（オーケストレーション）と、事故対応プロセスの自動化を実現する技術。

  * **基調講演での注目ポイント**: 単なる定型タスクの自動化にとどまらず、AIによる高度な文脈理解を組み込むことで、調査・判断プロセスの自律化が可能になった。

* **マイターアタック（MITRE ATT&CK）**

  * **定義**: サイバー攻撃者の攻撃手法や戦術・技術・手順（TTPs）を実世界の観察データに基づいて分類・体系化した世界共通のフレームワーク。

  * **基調講演での注目ポイント**: [Detection Engineering Agent](https://docs.cloud.google.com/chronicle/docs/secops/agentic-detection-engineering) において、脅威レポートから攻撃手順をMITRE ATT&CKに即座にマッピングし、合成ログの生成と検知ルール作成を90秒で自動完結させる工程で活用された。

  * **関連URL**: [MITRE ATT&CK® 公式サイト](https://attack.mitre.org/)

* **MTTD / MTTR**

  * **定義**: 平均検出時間（Mean Time to Detect）および平均修復・対応時間（Mean Time to Respond / Remediate）。セキュリティ運用の効率性と対応速度を示す重要指標（KPI）。

  * **基調講演での注目ポイント**: [Innersloth](https://www.innersloth.com/) 社において、Google Cloud Securityの導入によりMTTDが99.x%短縮、MTTRが98%短縮された定量実績として紹介された。

  * **関連URL**: [Wiz Academy: MTTD and MTTR in Cybersecurity Incident Response](https://www.wiz.io/academy/detection-and-response/mttd-and-mttr)

### 4.3 登場プロダクト・サービス一覧および機能概要

* **Google SecOps (Google Security Operations)**

  * **概要**: 自然言語処理（日本語対応）および自律型AIエージェントを内蔵し、ログ分析・アラート調査（60秒）・検知ルール自動生成（90秒）を超高速化する次世代統合SOCプラットフォーム。

  * **基調講演での注目ポイント**: 「[Triage & Investigation Agent](https://docs.cloud.google.com/chronicle/docs/secops/triage-investigation-agent)」および「[Detection Engineering Agent](https://docs.cloud.google.com/chronicle/docs/secops/agentic-detection-engineering)」の2大エージェントにより、SOCの運用能力を劇的に拡張。

  * **関連URL**:

    * 製品ページ: https://cloud.google.com/security/products/security-operations

    * Agentic SOC ソリューション: https://cloud.google.com/solutions/security/agentic-soc

    * Triage & Investigation Agent ドキュメント: https://docs.cloud.google.com/chronicle/docs/secops/triage-investigation-agent

    * Detection Engineering Agent ドキュメント: https://docs.cloud.google.com/chronicle/docs/secops/agentic-detection-engineering

* **Google Threat Intelligence（GTI）**

  * **概要**: Mandiantの最前線のインシデント対応知見、VirusTotalのグローバル脅威データ、およびGoogle DeepMindのAI分析能力を統合した脅威インテリジェンス基盤。

  * **基調講演での注目ポイント**: ジョン・ハルトキスト氏が所属するGoogle Threat Intelligence Group（GTIG）による攻撃者動向・AI悪用実態の解明と、防御基盤へのリアルタイムな知見還元を担う。

  * **関連URL**: https://cloud.google.com/security/products/threat-intelligence

* **Mandiant（マンディアント） / Mandiant Retainer**

  * **概要**: 世界最高峰のインシデント対応実績と脅威インテリジェンスを誇る Google Cloud のセキュリティ専門コンサルティング・調査組織。

  * **基調講演での注目ポイント**: 年間50万時間の調査知見をプラットフォームへ還元。有事の際に即座に専門家チームが介入できる事前契約「[Mandiant Retainer](https://cloud.google.com/security/consulting/mandiant-retainer)」の重要性を提示。

  * **関連URL**:

    * Mandiant トップページ: https://cloud.google.com/security/mandiant

    * Mandiant Retainer（事前即応契約）: https://cloud.google.com/security/consulting/mandiant-retainer

* **VirusTotal（ウイルストータル）**

  * **概要**: 世界最大級のファイル・URLマルウェア分析およびグローバル脅威データ収集プラットフォーム。

  * **基調講演での注目ポイント**: GoogleのフロンティアAIモデルに対し、最新のマルウェア動向と脅威インテリジェンスを供給する中核データソースとして機能。

  * **関連URL**: https://www.virustotal.com/ （ドキュメント: https://gtidocs.virustotal.com/ ）

* **Wiz（ウィズ）**

  * **概要**: AWS、GCP、Azureなどのマルチクラウド環境において、開発からランタイムまでのリスクを構成グラフレイヤ（Wiz Security Graph）で一元可視化・優先順位付けするCNAPPプラットフォーム。

  * **基調講演での注目ポイント**: [Google史上最大級の買収](https://cloud.google.com/blog/products/identity-security/google-completes-acquisition-of-wiz)。Fortune 100の65%が導入し、顧客の50%が「クリティカルリスクゼロ」を達成。利用者の半数が開発者であり「セキュリティの民主化」を実現。

  * **関連URL**:

    * Wiz 公式サイト: https://www.wiz.io/

    * Google Cloud による Wiz 買収完了発表ブログ: https://cloud.google.com/blog/products/identity-security/google-completes-acquisition-of-wiz

* **Google AI Threat Defense / CodeMender（コードメンダー）**

  * **概要**: Geminiの推論能力、Wizのリスク優先順位付け、Mandiantの知見を融合した「AI Threat Defense」の中核をなすAIコードセキュリティエージェント。コードレポジトリを自動検査し、悪用可能な脆弱性の検証からテスト済み修正パッチ（フィックス）の生成までを開発者ワークフロー内で自動化する。

  * **基調講演での注目ポイント**: 「AI Threat Defense」の中核として、攻撃者に悪用される前に先回りした脆弱性修復を実現するソリューションとして紹介。

  * **関連URL**:

    * CodeMender 製品ページ: https://cloud.google.com/security/codemender

    * CodeMender プレビュー発表ブログ: https://cloud.google.com/blog/products/identity-security/find-and-fix-software-vulnerabilities-with-codemender

    * Google AI Threat Defense 製品ページ: https://cloud.google.com/security/ai-threat-defense

* **Chrome Enterprise / Android Enterprise セキュリティインテリジェンス**

  * **概要**: 世界中で稼働する何十億台ものブラウザやモバイルデバイスからリアルタイムに収集されるグローバル脅威センサー網およびエンタープライズ保護基盤。

  * **基調講演での注目ポイント**: Googleが世界最大級の脅威可視性を持ち、民間企業として他社を凌駕する防御基盤を提供する根拠として言及。

  * **関連URL**:

    * Chrome Enterprise Premium: https://chromeenterprise.google/products/chrome-enterprise-premium/

    * Android Enterprise Security: https://www.android.com/enterprise/security/
