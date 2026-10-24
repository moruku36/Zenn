---
title: "セキュリティ専任じゃなくても対策は確かめられる ── 脆弱アプリとスキャナーで学ぶ Web Security Control Lab"
emoji: "🔐"
type: "tech"
topics: ["security", "python", "fastapi", "web", "初心者"]
published: false
---

## はじめに：セキュリティが「専門家の仕事」でなくなってきた

Web アプリを作っていると、こんな場面が増えていないでしょうか。

- ログイン機能や管理画面は自分が作った。でも、セキュリティの確認は「詳しい人がいないから後回し」になっている
- AI にコードを書かせるようになった。動くものは早くできるが、そのコードが安全かどうかを判断する材料がない
- 社内の AI 推進やデジタル化を担当していて、「セキュリティは大丈夫か」と問われても、何をもって大丈夫と言えるのかが分からない

セキュリティの専任担当がいる組織は、実は多くありません。それでも事故が起きれば対応するのは開発した本人や、推進した担当者です。いま多くのエンジニアにとっての課題は、高度な攻撃手法を知ることよりも先に、**「自分が入れた対策が、本当に効いているのかを確かめる方法」を持つこと**だと思います。

困りやすいのはこの点です。「認可チェックを入れた」「Cookie に設定を付けた」と言っても、それを確認する条件が曖昧だと、「検査は通ったが、何を保証したのか説明できない」状態になります。対策したつもり、で終わってしまう。

**Web Security Control Lab**（以下、本ラボ）は、その「確かめ方」を練習するための学習用システムです。セキュリティの専門家向けの診断ツールではありません。ふだん Web アプリを作ったり、社内の技術活用を推進したりしているエンジニアが、自分の PC の中で安全に試しながら、対策と確認の基本的な型を身につけるための教材です。

この記事では、本ラボが何のために作られていて、どう使ってほしいのかを先に説明し、そのあとでアプリ側と検査側の実装をコードで追います。技術的な中身を薄くはしません。用語は初出で説明するので、セキュリティが専門でない方も、自分のアプリに置き換えながら読んでみてください。

リポジトリ： https://github.com/moruku36/web-security-control-lab

> **安全上の前提**：本ラボは学習専用です。アプリは 127.0.0.1 にだけバインドし、インターネットや共有ネットワークに公開しないでください。スキャナーは後述のとおりローカル対象以外を拒否します。実データ・実パスワードは使わず、第三者のシステムを検査しないでください。[SECURITY.md](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/SECURITY.md)

本記事で扱う版はコミット [`4dc4d9d`](https://github.com/moruku36/web-security-control-lab/commit/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886)（2026年10月22日時点の main）です。実行結果は 2026年10月24日に Python 3.12.3 の独立環境で筆者が確認しました。リンクは原則としてこのコミットに固定しています。

## このラボは何のためにあるのか

本ラボの基本の流れは、**観察 → 検出（スキャナー） → 原因分析 → 修正 → 再検証**です。座学で用語を覚えるのではなく、弱点のあるアプリを実際に触り、検査で見つけ、なぜ起きたかを考え、直った状態と比べる。この一連の流れを、小さな題材で体験することが目的です。

構成は対になっています。

- 意図的に弱点を残した FastAPI アプリ（`VULNERABLE` モード）と、対策を適用した同じアプリ（`HARDENED` モード）
- 両モードに同じ六つの条件を適用し、結果を `PASS` / `FAIL` / `ERROR` で返す検査スキャナー（CLI）

なぜ「弱いアプリ」と「直ったアプリ」と「検査ツール」をセットにするのか。弱点に気づくには慣れが要りますし、直ったアプリだけでは「何がどう変わったのか」が見えません。検査ツールが間に入ることで、「どの条件を満たせば直ったと言えるのか」が先に決まり、学習が「何となく分かった」で終わらなくなります。

想定している使い方は、次の三つのどれかです。

| 使い方 | やること | 得られるもの |
| --- | --- | --- |
| 1. まず比べる | 両モードを起動し、同じ検査をかけて結果の違いを見る | 対策の有無で何が変わるのかの全体像 |
| 2. 一つを深掘りする | LAB-01 のワークシートに沿って、予測 → 観察 → 原因説明 → 修正 → 再検証を行う | 認可という一つのテーマを、説明できるレベルまで理解する |
| 3. 自分のアプリに応用する | 六つの検査条件をチェックリストとして、自分のアプリのログイン・管理画面・ログを見直す | 「対策と確認を対で考える」という考え方そのもの |

逆に、本ラボが**何ではないか**も、はっきりさせておきます。本ラボは本番サービスを診断するツールではなく、検査結果は本番の安全性の証明にはなりません（「限界」で詳しく述べます）。他人のシステムを試すためのものでもありません。スキャナーはローカル以外の対象をそもそも受け付けない設計です。

## 全体構成：部品は三つだけ

![学習者とホスト側スキャナーがローカルHTTPで1つのFastAPIアプリに接続し、SQLiteはアプリのプロセス内にある全体構成](/images/web-security-control-lab/wscl-architecture.png)

*図1：全体構成。色分けは、青が学習者、緑がスキャナー、紫が対象アプリ、橙が起動手段、黄がデータ。図はリポジトリの Mermaid を編集元に Pele 1.8.0 で生成。*

| 要素 | 実体 | 備考 |
| --- | --- | --- |
| 対象アプリ | FastAPI（`app/`）、1プロセス | 起動ごとに一つの `LAB_MODE` を取る |
| データ | SQLite（アプリのプロセス内） | 既定はインメモリ。`LAB_DB_PATH` で変更可 |
| スキャナー | ホスト側CLI `security-lab`（`scanner/`） | HTTP のみで検査し、SQLite は直接開かない |

ポイントは、スキャナーがアプリの内部（データベース）を覗くのではなく、外から HTTP でアクセスして観察する点です。攻撃者や利用者が見える範囲と同じところから確認するので、「外から見てどうなのか」という実務に近い観点になります。

起動はホストの uvicorn か、Docker Compose の唯一の `web` サービス（`VULNERABLE`、127.0.0.1:8000）です。DB コンテナーもスキャナーコンテナーもありません。モードは環境変数 `LAB_MODE` で起動時に読み込まれ、プロセスの途中で切り替わることはありません。両モードを比べるときは、ポートを分けて別プロセスで起動します。

## まず動かしてみる

読むだけで終わらせないために、最小限の手順を示します。必要なのは Python 3.11 以上と Git です（以下は Bash 用のコマンドです。Windows の PowerShell では環境変数の指定方法が異なるので、その点だけ後述します）。

```bash
git clone https://github.com/moruku36/web-security-control-lab.git
cd web-security-control-lab
git checkout 4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886
python -m venv .venv
source .venv/bin/activate
pip install -e '.[dev]'
```

記事で扱う版をそろえるため、コミットを指定しています。この状態（detached HEAD）は、読む・動かす目的では問題ありません。自分で修正する演習をする場合は、先に `git switch -c my-lab-practice` などで作業用ブランチを作ってください。

ターミナルを二つ使い、両モードを起動します。

```bash
# ターミナル1：弱点を残したモード
LAB_MODE=VULNERABLE uvicorn app.main:app --host 127.0.0.1 --port 8000

# ターミナル2：対策を適用したモード
LAB_MODE=HARDENED uvicorn app.main:app --host 127.0.0.1 --port 8001
```

PowerShell の場合は、起動前に `$env:LAB_MODE="HARDENED"` のように環境変数を設定してから uvicorn を実行してください。モードは起動時に読み込まれるので、変更するときはサーバーの再起動が必要です。

三つ目のターミナルで、起動とモードを確認してから検査します。

```bash
curl -s http://127.0.0.1:8000/health
curl -s http://127.0.0.1:8001/health

security-lab scan http://127.0.0.1:8000 --format json --expected-mode VULNERABLE
security-lab scan http://127.0.0.1:8001 --format json --expected-mode HARDENED
```

`/health` の返却 JSON の `mode` が期待どおりになっているかを確認してから進んでください。`--expected-mode` は「見ているアプリが想定したモードか」をスキャナーに確かめさせる指定で、食い違うと全検査が `ERROR` になります。対象には `/admin` のようなパスを付けず、ベース URL を渡します。

ブラウザーで触ってみるのも有効です。一般ユーザー `alice` / `user` でログインして管理者ページ `/admin` を開くと、`VULNERABLE` では表示され、`HARDENED` では 403 で拒否されます。ただし両モードを同じブラウザーで比べるときは、プロファイルを分けるか、一つずつ試して Cookie を消してください。Cookie はポートでは分離されないため、別タブだけではセッションが干渉することがあります（[RFC 6265 の 8.5 節](https://www.rfc-editor.org/rfc/rfc6265#section-8.5)）。

### 検査結果の読み方（まずはここだけ）

結果は三値です。

- `PASS`：その検査条件を満たした
- `FAIL`：弱点を観測した
- `ERROR`：前提を確認できず、検査が未完了になった。**成功ではありません**

正常に検査できた場合の期待結果は、`VULNERABLE` が 6 FAIL（HIGH 1 / MEDIUM 4 / LOW 1）、`HARDENED` が 6 PASS（findings 0）です。終了コードにも注意してください。`0` は「レポートが完了した」という意味で、`FAIL` を含んでいても 0 になります。「終了コード 0 ＝ 問題なし」ではない、という読み方は後半でもう一度取り上げます。

## 六つの検査で見るもの：どれも「普通のアプリ」に出てくるもの

六つの検査対象は、特殊なシステムだけの話ではありません。ログインのある Web アプリなら、どれも日常的に出てくる要素です。先に「自分のアプリではどの場面か」を挙げます。

| ID | 対象 | 身近な場面 | 弱いと何が起きるか（概要） |
| --- | --- | --- | --- |
| LAB-01 | 管理者ページの認可 | 管理画面、承認機能 | 管理者でない人が管理操作できてしまう |
| LAB-02 | セッション Cookie の HttpOnly | ログイン全般 | スクリプト経由でログイン状態を悪用されやすくなる |
| LAB-03 | セッション Cookie の SameSite | ログイン全般、外部サイト連携 | 意図しないリクエストにログイン状態が使われやすくなる |
| LAB-04 | セキュリティヘッダー | すべての画面・API | ブラウザー側の防御が働かない |
| LAB-05 | エラー応答の情報露出 | API のエラーハンドリング | 内部構成を推測する手がかりになる |
| LAB-06 | ログイン失敗の記録 | ログイン全般、運用監視 | 不正な試行が起きても誰も気づけない |

ここで用語を二つだけ整理します。**認証**は「誰か」を確かめること（ログイン）、**認可**は「その操作を許すか」を決めることです。ログインできていることと、管理者ページを見てよいことは別の話で、LAB-01 はこの混同を突きます。また **Cookie** は、ブラウザーに保存され、条件に応じてリクエストに自動で付く小さなデータです。ログイン状態の識別子を Cookie に載せる方式は広く使われており、その「値の読み取り制限（HttpOnly）」と「送信条件（SameSite）」が LAB-02、LAB-03 の対象です。

![ブラウザー側のCookie属性、サーバー側の権限確認・応答・記録という境界ごとに確認ポイントが分かれる概念図](/images/web-security-control-lab/wscl-boundaries.png)

*図2：統制ごとに守る境界と確認ポイントが異なる。色分けは、青がブラウザー側、紫がサーバー側、緑がスキャナー、赤がこの検査では未確認の範囲。*

## アプリ側の実装：対策はどこに書くのか

### モード分岐の置き方

モードは [app/config.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/app/config.py) で環境変数から読み込まれ、各機能がその値を参照して分岐します。弱点は「設定ミス」としてではなく、コード上に `INTENTIONAL_VULNERABILITY_FOR_LOCAL_LAB` というマーカーコメント付きで明示されています。レビュー時に「意図的な弱点」と「うっかり」を区別できるようにするためです。実務のコードでも、「意図的にそうしている箇所」に印を付けておく発想は応用できます。

エンドポイントは次のとおりです。

| パス | 役割 | モードで変わる点 |
| --- | --- | --- |
| `GET /health` | 状態とモードの返却 | なし（スキャナーの前提確認に使う） |
| `GET/POST /login`、`GET /logout` | 認証とセッション発行・破棄 | 失敗ログの記録有無（LAB-06） |
| `GET /profile`、`GET /api/profile` | ログイン必須ページ／同一内容の JSON | なし |
| `GET /admin` | 管理者ページ | ロール確認の有無（LAB-01） |
| `GET /api/diagnostic` | 診断用エラー応答 | 内部情報の露出有無（LAB-05） |
| `GET /audit-logs` | 監査ログの返却 | なし（両モードとも認証なし。後述の限界） |

練習用アカウントは固定の平文フィクスチャです（一般ユーザー `alice` / `user`、管理者 `admin` / `admin`）。本番の認証情報の持ち方とは無関係の教材用データです。

### LAB-01：認可はエンドポイント自体で確認する

`/admin` の実装（[app/main.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/app/main.py)）では、未ログインなら両モードともログインへ 303 で戻します。違いはその次です。`HARDENED` ではロールが `admin` でなければ 403 を返し、`VULNERABLE` ではこの確認自体がありません。

```python
if LAB_MODE == LabMode.HARDENED.value:
    if user.get("role") != "admin":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Access denied: Administrator role required.",
        )
```

実務でよくある失敗は、「一般ユーザーには管理画面へのリンクを見せていないから大丈夫」というものです。リンクが見えなくても URL を直接叩かれれば届いてしまいます。この実装が示しているのは、画面の表示制御ではなく、**確認はエンドポイント自体で行い、そこで拒否する**という原則です。自分のアプリで管理機能を作るときは、まずここを確認してください。

期待する観測結果を表にすると、次のとおりです。

| 観察対象 | VULNERABLE | HARDENED |
| --- | --- | --- |
| 未ログインで `/admin` | 303 でログインへ | 303 でログインへ |
| 一般ユーザーで `/admin` | 200 で表示される | 403 で拒否 |
| 管理者で `/admin` | 200 で表示 | 200 で表示 |
| ログアウト後の旧トークンで `/admin` | 303 でログインへ | 303 でログインへ |

すべてが両モードで違うわけではない点が大切です。違いは一般ユーザーのアクセスだけで、ログアウト後の旧トークン無効化は両モードに共通する正しい実装として置かれています。「対策済みの状態」と「もともと正しい部分」を区別して見る練習になります。

### セッションと Cookie

ログインに成功すると `secrets.token_hex(24)` でトークンを発行し、セッション表にレコードを作って Cookie `lab_session` に載せます（[app/auth.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/app/auth.py)）。ログアウトではセッション表のレコードを削除するので、旧トークンは両モードとも無効になります。

Cookie 属性はモードで分岐します。`HARDENED` では `httponly=True`、`samesite="lax"` を付け、`VULNERABLE` ではどちらも付けません（LAB-02、LAB-03）。なお `secure=False` は localhost の HTTP で使うための意図的な選択で、両モード共通の制約です（「限界」参照）。フレームワークの既定値に任せきりにせず、セッション Cookie にどの属性が付いているかを一度確認する――それだけで、多くのアプリでは見直しポイントが見つかります。

### ヘッダー、エラー、ログ

- **LAB-04（セキュリティヘッダー）**：ブラウザーに「こう振る舞ってほしい」という指示を応答に添える仕組みです。`HARDENED` のときだけ HTTP ミドルウェアで `Content-Security-Policy: default-src 'self'`、`X-Content-Type-Options: nosniff`、`Referrer-Policy: strict-origin-when-cross-origin`、`X-Frame-Options: DENY` を付けます。設定の有無がそのまま検査結果に表れます。
- **LAB-05（エラー応答）**：`/api/diagnostic` は、`VULNERABLE` では模擬のトレースバックや実行環境情報を含む JSON を 500 で返し、`HARDENED` では汎用メッセージと `error_code` のみに絞ります。開発中は便利な詳細エラーも、本番で利用者に返すと内部構成を推測する手がかりになります。露出する値はすべて教材用の模擬データで、実際の秘密ではありません。
- **LAB-06（監査ログ）**：認証成功は常に監査ログへ `AUTH_SUCCESS` を記録しますが、失敗時の `AUTH_FAILURE` は `HARDENED` でだけ記録します。「成功した操作」だけ記録して失敗を記録しないと、パスワードの総当たりのような試行が起きても検知できません。失敗を記録しないこと自体が、ここでは弱点として扱われます。

## 検査側の設計：PASS は何を証明するのか

ここからは検査する側の話です。自分のアプリに自動チェックを入れたい人、ベンダーの検査結果を読まなければならない人には、特に関わる部分です。

スキャナー（[scanner/checks.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/scanner/checks.py)）の六つの検査は、次の条件で判定します。

| ID | 対象 | PASS の条件 | FAIL の理由コード（例） | 重要度 |
| --- | --- | --- | --- | --- |
| LAB-01 | `/admin` の認可 | 匿名はログインへ、一般ユーザーは 403、管理者は表示、ログアウト後の旧トークンは無効 | `USER_ADMIN_ACCESS` ほか | HIGH |
| LAB-02 | セッション Cookie | `HttpOnly` 属性がある | `COOKIE_ATTRIBUTE_MISSING` | MEDIUM |
| LAB-03 | セッション Cookie | `SameSite` がちょうど一つあり、値が `Lax` または `Strict` | `COOKIE_ATTRIBUTE_MISSING` / `COOKIE_SAMESITE_VALUE_INVALID` | MEDIUM |
| LAB-04 | `/health` の応答ヘッダー | CSP、X-Content-Type-Options、Referrer-Policy が存在し空でない | `HEADER_MISSING` | MEDIUM |
| LAB-05 | `/api/diagnostic` | 内部情報を含まず、汎用エラーを返す | `DIAGNOSTIC_EXPOSURE` | LOW |
| LAB-06 | 監査ログ | 失敗ログインの前後で `AUTH_FAILURE` の件数が増える | `AUTH_FAILURE_NOT_LOGGED` | MEDIUM |

### ERROR を成功にしない

各検査の結果は `PASS`（条件を満たした）、`FAIL`（弱点を観測した）、`ERROR`（前提を確認できず未完了）の三値です。ここで重要なのは、**前提が崩れたら弱点の有無を断定せず ERROR に倒す**ことです。自動検査でありがちな失敗は、「うまく動かなかった検査」が「問題なし」に化けることです。本ラボはその逆を徹底しています。

例えば LAB-01 は、いきなり `/admin` を叩きません。まず合成アカウントでログインし、`/api/profile` で本人とロールが期待どおりかを確認してから（この確認結果は証拠の `verified` に残ります）、匿名・一般ユーザー・管理者・ログアウト後の旧トークンの四つの観測を行います。ログイン前提が成立しなければ、その検査は ERROR です。観測された弱点が複数ある場合は、匿名アクセス ＞ 一般ユーザーのアクセス ＞ セッション残存、という優先順位で主たる理由コードを決めます。

LAB-03 の条件も厳密です。対象は `lab_session` の `Set-Cookie` ヘッダーだけ（他の Cookie の SameSite は無視）、属性はちょうど一つ、値の大文字小文字と前後の空白は無視して `Lax` / `Strict` のみ合格。属性が重複したり対象ヘッダーが複数あったりして実効値が定まらない場合は、FAIL ではなく ERROR です。なお `SameSite=None` は「このラボの条件を満たさない」という判定であって、一般に危険という評価ではありません。属性未指定時にブラウザーがどう扱うかも、この検査の対象外です。

逆に、意図的に狭くしてある検査もあります。LAB-04 はヘッダーの存在と非空だけを見て、CSP の内容の強さは評価しません。LAB-06 は件数の増加だけを見て、特定の試行とログの厳密な対応やアラート発報は対象外です。**PASS は「その検査条件を満たした」ことだけを意味し、それ以上を主張しない**のが設計方針です。検査結果を読む側も、この線引きを理解していると、ベンダーレポートや自動診断の結果を過大にも過小にも読まずに済みます。

### 証拠を最小化する

レポート（スキーマ 2.0）の証拠に残るのは、`actor`（anonymous / user / admin / user_old_token）、`endpoint`、ステータスコード、許可されたリダイレクト先だけに allowlist で制限された観測記録（最大16件）と、固定の理由コードです。メッセージは理由コードごとの固定テンプレートで、応答本文や例外の文字列がレポートに混ざることはありません。検証は [scanner/models.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/scanner/models.py) の allowlist バリデーションで担保されています。検査ツール自身が、対象の内部情報を証拠として取り込まないための設計です。検査結果を社内で共有したりチケットに貼ったりする場面では、この配慮があるかどうかで扱いやすさが変わります。

## スキャナー側の安全設計：対象を間違えたら、送信前に止まる

![起動、対象の検証、前提確認、六つの検査、期待値との比較までの検証フローと終了コード](/images/web-security-control-lab/wscl-verification-flow.png)

*図3：検証フロー。前提を確立できなければ ERROR で終了し、成功扱いにしない。色分けは、緑がスキャナーの処理と HARDENED の期待値、橙が VULNERABLE の期待値、赤が異常終了（拒否・ERROR）。*

「検査ツールだから、どこを検査してもよい」わけではありません。対象の検証は Fail-Closed（疑わしければ止まる）です（[scanner/target.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/scanner/target.py)、[scanner/cli.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/scanner/cli.py)）。

- 許可するホストは `localhost`、ループバック IP（`ipaddress` で判定）、Compose のサービス名のみ。それ以外はリクエストを送る前に拒否
- 対象 URL はベース URL の形（スキーム＋ホスト［＋ポート］）だけを受け付け、認証情報・パス・クエリ・フラグメント付きは拒否。拒否した URL はエラーメッセージにエコーしない
- HTTP クライアントはプロキシ環境変数を無視し（`trust_env=False`）、リダイレクトを追随せず、タイムアウトは5秒
- 使う資格情報は合成フィクスチャのみ。検査開始時に `/health` でサービス名とモードを確認し、`--expected-mode` と食い違えば全検査を ERROR（`MODE_MISMATCH`）にする

AI エージェントや自動化ツールに作業を任せる場面が増えるなかで、「対象を間違えたら送信前に止まる」「権限のない対象にはそもそも届かない」という設計は、検査ツールに限らず自動化全般に応用できる考え方です。

終了コードは次のとおりです。自動化（CI）に組み込むときの要点になります。

| 終了コード | 意味 |
| --- | --- |
| 0 | レポート完了（**FAIL を含んでいても 0**） |
| 1 | サブコマンドなし |
| 2 | 対象の拒否、または引数エラー |
| 3 | 検査未完了（ERROR を含む）、または実行時エラー |

つまり「終了コード 0 ＝ 弱点なし」ではありません。合否はレポート内の `status_summary` と `findings` で判断します。自前のチェックを CI に組み込む場合も、「ツールが正常終了したか」と「検査に合格したか」は分けて扱う必要があります。

## 実行結果

筆者の環境（Python 3.12.3、コミット `4dc4d9d`、2026年10月24日）での確認結果です。記事の手順どおりに両モードを起動し、検査した結果が期待値と一致しました。

- テスト：126 件すべて成功（unit / integration / vulnerabilities）
- `VULNERABLE`（ポート8000）：6 FAIL、findings は HIGH 1 / MEDIUM 4 / LOW 1、終了コード 0
- `HARDENED`（ポート8001）：6 PASS、findings 0、終了コード 0
- わざと `--expected-mode` を食い違わせると全検査 ERROR（`MODE_MISMATCH`）で終了コード 3 になること、`https://example.com` を対象にするとリクエスト前に拒否されて終了コード 2 になることもあわせて確認しました

テキスト形式のレポート（VULNERABLE）は次のような出力です（抜粋）。

```text
Web Security Control Lab
Target: http://127.0.0.1:8000 (mode reported by target: VULNERABLE)
Scope: six local educational checks only; not a production or compliance assessment.
[HIGH] LAB-01 Broken Access Control
  /admin is accessible by a non-admin user
[MEDIUM] LAB-02 Session Cookie
  HttpOnly flag is missing
[MEDIUM] LAB-03 Session Cookie
  SameSite policy is missing
[MEDIUM] LAB-04 Security Header
  Content-Security-Policy is missing
[LOW] LAB-05 Information Disclosure
  Detailed error information is exposed
[MEDIUM] LAB-06 Security Logging
  Failed authentication attempt was not recorded
```

リポジトリの CI には、両モードに対してライブ検査を行うジョブを含む構成があり、対象コミットの実行は成功しています（[CI 実行結果](https://github.com/moruku36/web-security-control-lab/actions/runs/39791370267)、2026年10月22日確認）。

## 限界：HARDENED は「本番で安全」ではない

ここは誤解しやすいので、はっきり書きます。`HARDENED` は「六つのラボ統制を適用した状態」という意味に限定されます。README が明記しているとおり、両モードに共通して次の制約が残っています。

- 固定の平文テスト資格情報、セッションの有効期限なし、`GET` によるログアウト、認証なしの `/audit-logs`
- Cookie の `Secure` なし（localhost の HTTP 前提）
- 検査は HTTP クライアントによる観測であり、ブラウザーでの実動作の検証ではない。Cookie 属性の確認は、クロスサイト操作の安全性の証明にはならない
- CSRF 対策そのもの（トークン等）はラボの統制に含まれない。スキャナーはブラウザーを動かして CSRF の成立を試すものでもない（[MDN の CSRF 解説](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/CSRF)）
- 六つの検査の成功は、本番の安全性や、この教材の教育効果を証明しない（教育効果は未評価）

実サービスで使うなら、HTTPS の強制、パスワードのハッシュ保存、セッションの期限と失効設計、CSRF トークン、ログへのアクセス制御と保存期間、CSP の内容設計などが別途必要です。本ラボの範囲は、その入口にあたる「基本の六つを確認できる形にする」ところまでです。

## 自分の仕事にどう活かすか

最後に、読者像ごとに活かし方を整理します。

**Web アプリを開発している人へ。** 機能追加のときに、六つの条件をそのまま見直しの観点として使えます。管理機能を足したら「未ログイン／一般ユーザー／管理者」で同じ URL を試す。ログインを足したらセッション Cookie の属性と、失敗がログに残るかを見る。レビューで「対策しました」と言われたら、「どの観測をもって確認しましたか」と聞けるようになることが、この教材の実務上の価値です。

**AI にコードを書かせている人、AI 推進を担当している人へ。** 生成されたコードについては、「動くこと」と「対策が効いていること」は別の話です。本ラボの考え方――対策と検査条件を対で決め、前提が崩れたら合格にしない――は、AI が生成した認証・認可まわりのコードをレビューするときの物差しになります。また、自動化ツール側の Fail-Closed（対象の allowlist、証拠の最小化）も、エージェントに作業を任せる範囲を設計するときの参考になります。

**チームで開発している人へ。** 六つの検査条件は、セキュリティレビューの共通言語として使えます。「何となく危なそう」ではなく「LAB-01 の条件で見ると、この画面は一般ユーザーに開いている」という言い方ができると、専門家でなくても議論が前に進みます。

## まとめ

本ラボで一番伝えたいのは、個々の対策の名前ではなく、確認の設計です。

1. **対策と検査条件を対で定義する**：何を直すかと、どの観測をもって直ったとするかを同時に決める
2. **ERROR を成功にしない**：前提が確認できない検査は「合格」ではなく「未完了」として扱い、終了コードでも区別する
3. **PASS の意味を広げない**：検査していないこと（CSP の強さ、ブラウザーの実動作、CSRF 全体）を、通ったことのように言わない
4. **検査ツール側も Fail-Closed にする**：対象の allowlist、証拠の最小化、例外文字列を記録しない

まずは両モードを起動して結果の違いを眺め、次に LAB-01 の[ワークシート](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/docs/worksheets/LAB01.ja.md)（予測 → 観察 → 原因説明 → 修正 → 再検証）で一つだけ深掘りしてみてください。リポジトリは MIT ライセンスです。

## 参考リンク

- リポジトリ： https://github.com/moruku36/web-security-control-lab
- [README（日本語）](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/README.ja.md) ── 起動方法、六つの検査、限界
- [SECURITY.md](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/SECURITY.md) ── ローカル専用の範囲
- [docs/SECURITY_CONTROLS.md](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/docs/SECURITY_CONTROLS.md) ── 各統制の理論・検知ロジック・OWASP 対応
- [MDN: CSRF](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/CSRF)
- [RFC 6265 §8.5](https://www.rfc-editor.org/rfc/rfc6265#section-8.5) ── Cookie とポートの境界
