---
title: "そのPASSは本当に信用できるか ── 脆弱版と対策版を同じCIで検査するセキュリティ検証ラボを作った"
emoji: "🔐"
type: "tech"
topics: ["security", "python", "fastapi", "githubactions", "ci"]
published: true
published_at: 2026-10-27 06:00
---

セキュリティ検査をCIに組み込むと、緑のチェックが付くようになります。ただ、その緑が「対策が効いている」ことを示しているのか、「検査が何も見ていない」だけなのかは、緑のチェックだけでは区別できません。

個人開発のOSSとして作った **[Web Security Control Lab](https://github.com/moruku36/web-security-control-lab)**（以下、WSCL）は、この区別をつけることを目的にしています。意図的に弱点を残したWebアプリと、その弱点を検出する小さなスキャナー、両者を毎回突き合わせるCIで構成した、localhost専用の検証環境です。

本稿では、WSCLで採用したセキュリティ検査の設計を紹介します。

- 弱い版と対策版の**両方に期待値を置き**、検査器そのものの故障を検出する
- 結果を **PASS / FAIL / ERROR** の3値にし、「検査できなかった」ことを成功扱いにしない
- スキャナー自身を **Fail-Closed** にして、検査対象の取り違えや誤用を防ぐ
- CIの証跡を**許可リスト形式で検査**してから保存する
- 検査の抜けを見つけたら、**失敗するテストを先に**追加して塞ぐ（実際に2回ありました）

教材として作ったリポジトリですが、ここで紹介する設計の多くは、業務のCIに組み込んだ脆弱性スキャンや、自作のセキュリティチェックにもそのまま使えます。

:::message
WSCLは学習用のアプリです。`127.0.0.1` だけで起動し、インターネットや共有ネットワークには公開しないでください。実データや実パスワードは使わず、第三者のシステムには使わないでください（[SECURITY.md](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/SECURITY.md)）。本稿は検査の設計を扱うもので、攻撃手法は扱いません。
:::

本稿の記述は、2026年10月22日時点のmain（`4dc4d9d`）に基づいています。

## 全体像

構成は単純です。FastAPIのアプリが1プロセスで動き、SQLiteはそのプロセスの中にあります（既定ではメモリー上）。アプリは起動時の環境変数 `LAB_MODE` によって `VULNERABLE` か `HARDENED` のどちらかで動きます。スキャナー `security-lab` はホスト側のCLIで、HTTPだけを使ってアプリを検査し、SQLiteは直接開きません。

![学習者のブラウザーやcurlと、ホスト側のスキャナーCLIが、ループバックのHTTPで同じFastAPIアプリに接続する。アプリは起動時のLAB_MODEでVULNERABLEかHARDENEDになり、SQLiteはアプリのプロセス内にある。CIは同じCLIで両モードを検査する](/images/web-security-control-lab/fig1-overview.png)
*図1：WSCLの構成。スキャナーはHTTPで観測するだけで、アプリの内部には触れない（Pele で作成）*

検査は次の6つです。

| ID | スキャナーが確認する条件（満たせばPASS） | VULNERABLE で残している弱点 | 重要度 |
| --- | --- | --- | --- |
| LAB-01 | `/admin` で、未ログインはリダイレクト、一般ユーザーは拒否、管理者は許可。ログアウト後の旧トークンは拒否 | 一般ユーザーが `/admin` を開ける | HIGH |
| LAB-02 | セッションCookieに `HttpOnly` がある | `HttpOnly` がない | MEDIUM |
| LAB-03 | セッションCookieに `SameSite` 属性がちょうど1つあり、値が `Lax` か `Strict` | `SameSite` がない | MEDIUM |
| LAB-04 | `/health` に CSP、`X-Content-Type-Options`、`Referrer-Policy` があり、値が空でない | ヘッダーがない | MEDIUM |
| LAB-05 | 診断用エンドポイントのエラーが内部情報を含まない汎用メッセージ | スタックトレース風の模擬データを返す | LOW |
| LAB-06 | ログイン失敗で監査ログの `AUTH_FAILURE` 件数が増える | ログイン失敗を記録しない | MEDIUM |

アクセス制御、セッションCookie、応答ヘッダー、エラー処理、監査ログと、現場で「最低限ここは確認したい」と言われる項目を一通り並べました。いずれもアプリ側の実装は数行の分岐で、たとえばLAB-01の対策は次のサーバー側のロール確認だけです（[app/main.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/app/main.py)）。

```python:app/main.py（抜粋）
if LAB_MODE == LabMode.HARDENED.value:
    if user.get("role") != "admin":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Access denied: Administrator role required.",
        )
```

アプリ側はあえて単純にしてあります。本稿で扱う工夫は、ほとんどが**検査する側**にあります。

## 工夫1：弱い版にも期待値を置き、検査器そのものを検査する

セキュリティスキャンをCIに入れたとき、いちばん気づきにくい失敗は**スキャナーが静かに壊れること**です。対象URLの設定ミス、認証の前提が崩れた、レスポンスの形式が変わった。こうした理由で検査が何も見なくなっても、結果は「検出0件」になり、CIは緑のままです。

WSCLでは、同じスキャナー・同じ6検査を両方のモードに当て、**両方に期待値を置いています**。

| モード | 期待するスキャナー結果 |
| --- | --- |
| `VULNERABLE` | 6件 FAIL（HIGH 1 / MEDIUM 4 / LOW 1） |
| `HARDENED` | 6件 PASS、findings 0 |

![同じスキャナーでVULNERABLEとHARDENEDを検査する。VULNERABLEは6件FAIL、HARDENEDは6件PASSが期待値で、どちらかがずれたらCIを失敗にする。検出漏れはVULNERABLE側で、誤検知や対策の後退はHARDENED側で気づける](/images/web-security-control-lab/fig2-two-sided.png)
*図2：両側の期待値。片側だけでは、検査器の故障と対策の成功を区別できない（Pele で作成）*

`HARDENED` だけを検査していると、「6件PASS」はスキャナーの故障でも起こり得ます。そこで、弱点が**確実に存在する**ことが分かっている `VULNERABLE` も同時に検査し、6件すべてが期待どおりFAILすることを確かめます。FAILが減ったら、検出漏れか検査の故障です。逆に `HARDENED` がPASSにならなければ、誤検知か対策の後退（デグレ）です。

業務のアプリに意図的な脆弱版を用意するのは難しいかもしれません。ただ、考え方は持ち込めます。たとえば次のような形です。

- 自作のセキュリティチェックには、**必ず落ちるべき入力**（既知の悪い設定、テスト用の弱いフィクスチャー）を一緒に流す
- 依存関係スキャナーなら、既知の脆弱バージョンを固定した検証用のマニフェストで、検出されることを定期的に確認する
- WAFやヘッダー設定の確認なら、設定を外した環境でFAILすることも確認する

「対策済みのものがPASSする」ことと「未対策のものがFAILする」ことを、セットで確かめるという考え方です。

## 工夫2：PASS / FAIL / ERROR の3値と終了コード

二つ目の落とし穴は「findings が空＝安全」という読み方です。WSCLのスキャナーは、各検査の結果を3値で返します（[scanner/checks.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/scanner/checks.py)）。

- **PASS**：検査の条件を満たした
- **FAIL**：弱点を観測した
- **ERROR**：前提を確立できず、検査を完了できなかった。**成功ではない**

たとえばLAB-01は、一般ユーザーと管理者で合成ログインし、`/api/profile` で「本当にその人としてログインできているか」を確かめてから `/admin` を試します。ログインに失敗した状態で `/admin` が拒否されても、それは認可が効いた証拠にはなりません。こうしたケースはPASSではなくERRORにします。

![対象URLが許可リスト外なら送信前に終了コード2。healthの確認や合成ログインといった前提が確立できなければERRORで終了コード3。前提が揃えば各検査がPASSかFAILになり、レポートが完了すれば終了コード0。0はFAILを含みうる](/images/web-security-control-lab/fig3-status.png)
*図3：結果と終了コードの決まり方（Pele で作成）*

終了コードの意味も分けています（[scanner/cli.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/scanner/cli.py)）。

| 終了コード | 意味 |
| --- | --- |
| `0` | レポートを完了した（**FAILを含むことがある**） |
| `1` | サブコマンドの指定がない |
| `2` | 対象URLの拒否・引数エラー（リクエストは送らない） |
| `3` | 検査が未完了（ERRORを含む）、または実行時エラー |

以前のバージョンでは、想定外の応答を受けても終了コード0になることがありました。現在は、1つでもERRORがあれば3を返します。終了コード0だけを見て合否を決めるCIでは「弱点なし」と判定できない点に注意してください。WSCLのCIは、JSONレポートの中身を期待値と照合して判定しています。

さらに、`--expected-mode` で「どのモードを検査しているつもりか」を明示できます。`/health` が返すモードと食い違えば、全検査がERRORになります。ポートの取り違えで `HARDENED` のつもりが `VULNERABLE` を検査していた、といった事故をここで止めます。

```bash
security-lab scan http://127.0.0.1:8000 --format json --expected-mode VULNERABLE
security-lab scan http://127.0.0.1:8001 --format json --expected-mode HARDENED
```

一つの検査の中でERRORとFAILが両方起きた場合は、ステータスはERRORにしつつ、観測できた弱点はfindingとして残します。「検査が未完了だったから弱点が消えた」ことにならないようにするためです。

## 工夫3：スキャナー自身をFail-Closedにする

検査ツールは、それ自体が事故の原因になり得ます。WSCLは意図的に脆弱なアプリを扱うので、スキャナーが「どこにでもリクエストを送れる」状態は避けたいところです。そこで、スキャナーには次の制約を入れています。

**対象は許可リストのローカル対象だけ。** `localhost`、ループバックIP、Composeのコンテナ名以外は、リクエストを送る前に終了コード2で拒否します。判定できない入力も拒否側に倒します（[scanner/target.py](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/scanner/target.py)）。

```python:scanner/target.py（抜粋）
hostname = hostname.lower().strip("[]")

# Explicitly allowed hostname list
if hostname in ("localhost", "web-security-control-lab-app"):
    return True

# Check IP address
try:
    ip = ipaddress.ip_address(hostname)
    # Only loopback is permitted
    return ip.is_loopback
except ValueError:
    # Not an IP and not in local hostnames
    return False
```

**URLはベースURLの形だけ。** `scheme://host[:port]` 以外、つまり認証情報、パス、クエリ、フラグメントを含むURLは拒否します。拒否したURLの中身はエラーメッセージに出しません。

**HTTPクライアントは環境に引きずられない。** すべてのクライアントで `trust_env=False`（プロキシ環境変数を無視）と `follow_redirects=False`（リダイレクトを辿らない）を指定しています。ループバックに送ったつもりのリクエストがプロキシ経由で外に出たり、リダイレクトでローカル外に誘導されたりするのを防ぐためです。

**出力に生データを載せない。** レポートの観測記録は、`actor`・`endpoint`・`status`・`redirect` の4キーだけを許可リストで検証します。エラーメッセージは14種類の理由コードに対応する固定文言だけで、レスポンス本文や例外の文字列は出しません。テストでは `CANARY-…` という合成トークンを流し、JSONやテキストの出力に、その文字列や `Set-Cookie`、Cookie名が現れないことを確認しています。

業務のCIでも、スキャナーやヘルスチェックのスクリプトが本番やステージングのURLを誤って叩く事故や、ログにトークンが残る事故は起こります。「対象の許可リスト」「出力の許可リスト」「プロキシとリダイレクトを明示的に切る」の3点は、自作の検査スクリプトにも入れておく価値があると考えています。

## 工夫4：CIの証跡を、保存する前に検査する

CI（[.github/workflows/ci.yml](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/.github/workflows/ci.yml)）は3つのジョブで構成しています。

![mainへのpushかプルリクエストで、Python 3.11のlintとテスト、3.12のテストが走る。ライブスキャンはテストが失敗しても実行し、両モードの検査、provenanceの記録、redaction確認、証跡のアップロードを行い、最後のevidence gateがすべて成功していなければ失敗にする](/images/web-security-control-lab/fig4-ci.png)
*図4：CIの流れ。証跡を残すためにライブスキャンは常に実行し、合否は最後のゲートでまとめて判定する（Pele で作成）*

ポイントは3つあります。

1. **ライブスキャンはテストが失敗しても実行する。** `if: always()` を指定し、テストが落ちた回でも両モードのスキャン結果を残します。何が壊れたのかを後から調べられるようにするためです。合否は最後の evidence gate で判定し、すべてのジョブと手順が成功していなければ失敗にします。
2. **provenance を記録する。** コミットSHA、実行したコマンド、Pythonと依存パッケージのバージョン、上流ジョブの結果を証跡に含めます。
3. **アップロード前に証跡そのものを検査する。** すべての証跡ファイルを、まず許可リスト形式のスキーマで検証し、そのうえで秘密情報らしきパターン（Cookie名、`password`、長い16進文字列、ホームディレクトリのパスなど）が含まれないか確認します。秘密パターンの検査は多層防御の一つという位置づけで、主な制御は許可リストのほうです。

サニタイズ済みの `scan-evidence` アーティファクトは7日間保持します。アーティファクトはいずれ消えるので、記事やREADMEで結果を示すときは、CI runのリンクとコミットSHAを併記するようにしています。本稿の対象コミット `4dc4d9d` の [CI run](https://github.com/moruku36/web-security-control-lab/actions/runs/39791370267) では、3ジョブ、両モードのライブスキャン、redaction確認、evidence gateがすべて成功しています。

スキャン結果をアーティファクトに保存するCIはよくありますが、**保存する前に中身を検査する**ところまでやっている例は多くないと思います。スキャナーの出力には、対象のレスポンスの一部やエラー文字列が混ざりやすく、うっかりトークンや内部パスが公開リポジトリのアーティファクトに残ることがあるためです。

## うまくいかなかったこと：「属性があるだけでPASS」していた

ここまで偉そうに書きましたが、WSCLの検査にも抜けがありました。どちらも「名前があるか」だけを見ていて、「中身が条件を満たすか」を見ていなかった、という同じ種類の抜けです。

### LAB-04：空のヘッダーでもPASSしていた

当初のLAB-04は、必要なヘッダーの**名前**が `/health` の応答にあればPASSにしていました。そのため、`Content-Security-Policy:` のように値が空のヘッダーでも通っていました。修正後は、値が空または空白だけのヘッダーをFAILにしています（PR #9）。どのヘッダーが「ない」のか「空」なのかも、メッセージで区別するようにしました。

### LAB-03：`SameSite=None` でもPASSしていた

当初のLAB-03は、セッションCookieに `SameSite` という**属性名**があればPASSにしていました。

| `lab_session` の Set-Cookie | 修正前 | 修正後 |
| --- | --- | --- |
| `SameSite=Lax` / `samesite= STRICT ` | PASS | PASS |
| 属性なし | FAIL | FAIL |
| `SameSite=None`、空の値、未知の値 | **PASS** | FAIL |
| `SameSite` 属性や `lab_session` ヘッダーが重複 | **PASS**（最初の属性名だけを見ていた） | ERROR（有効な値を判断できない） |

修正後の判定は短い関数にまとめています。

```python:scanner/checks.py（抜粋）
def _samesite_outcome(set_cookie: str) -> str:
    values = []
    for part in set_cookie.split(";")[1:]:
        name, _, value = part.partition("=")
        if name.strip().lower() == "samesite":
            values.append(value.strip().lower())
    if not values:
        return "missing"
    if len(values) > 1:
        return "ambiguous"
    return "ok" if values[0] in SAMESITE_ACCEPTED else "invalid"
```

重複は、新しいFAILの理由にはせず、ERRORにしました。有効な値が判断できない状態を「弱点がある」とも「問題ない」とも言えないからです。また、ここでの `SameSite=None` のFAILは「このラボの条件を満たさない」という意味です。`SameSite=None` が一般に脆弱だと主張するものではありません。

### 直し方：失敗するテストを先に入れる

どちらも、修正より先に**失敗する回帰テスト**を入れました。LAB-03では、テストだけを追加したコミット（`9dee854`）と、検査を直したコミット（`bbf50d6`）を分けています。前者をチェックアウトして `pytest` を実行すると、抜けが再現できます。

![検査の抜けに気づいたら、まず失敗するテストを追加し、次に検査を直し、CIで全テストと両モードの実スキャンを確認する](/images/web-security-control-lab/fig5-regression.png)
*図5：検査の抜けを塞ぐ手順（Pele で作成）*

この抜けは、工夫1の「両側の期待値」では見つけられませんでした。`VULNERABLE` は `SameSite` 属性自体を出さないので、修正前でも期待どおりFAILしていたからです。両側の期待値で分かるのは「検査器が全体として動いているか」までで、**検査条件の境界が正しいか**は別に確かめる必要があります。境界値のテストは、ユニットテストで `httpx.MockTransport` を使った偽のアプリを作り、`SameSite=None`、空の値、`"Lax"`（引用符付き）、`Lax,Strict`、重複といった入力を一つずつ当てて確認しています。

### もう一つの反省：図が実態と違うことを言っていた

以前のREADMEの構成図には、AIエージェントのタスク定義（`factory/tasks/*.yaml`）からCIへ矢印が伸びていました。これでは、CIがタスク定義を読んで自動実行するように読めます。実際の `ci.yml` には、そのような手順はありません。セキュリティの文書で、図が実装と違う主張をしているのはよくないので、図はすべてMermaidのソースから作り直し、実際の `ci.yml` に合わせて描き直しました。

## AIエージェントと開発するときの境界

WSCLは、AIエージェントに実装を任せ、人間がマージを判断する形で開発しました（ガバナンスの仕組みは別リポジトリの [AI Governance Control](https://github.com/moruku36/ai-governance-control) にまとめています）。

途中、最初に実装を担当したエージェントが、本来は後のフェーズで行う `HARDENED` 側の対策まで先に実装していたことが、別のエージェントによる独立レビューで見つかりました。このときは実装をやり直さず、レビュー側で `VULNERABLE` と `HARDENED` の両方の回帰テストを強化し、両モードを期待値で照合する今のCIにつながっています。

セキュリティの検証環境では、「弱点が残っていること」も仕様です。AIに任せると、気を利かせて弱点を直してしまうことがあります。意図的な弱点にはソース中で `INTENTIONAL_VULNERABILITY_FOR_LOCAL_LAB` という目印を付け、テストで「VULNERABLEでは弱点が観測されること」も固定しています。

## 現場に持ち帰るためのチェックリスト

WSCLの設計から、業務のセキュリティ検査に持ち帰れそうな項目をまとめます。

- [ ] 検査が**必ず落ちるべき入力**を用意し、検出されることもCIで確認している
- [ ] 「検査できなかった」を成功扱いにせず、PASSとFAILとは別の状態として扱っている
- [ ] 終了コード0の意味（完了なのか、弱点なしなのか）をチームで共有している
- [ ] 検査対象を許可リストで限定し、判定できない入力は拒否している
- [ ] 検査スクリプトのHTTPクライアントで、プロキシとリダイレクトの扱いを明示している
- [ ] スキャン結果やログを保存・公開する前に、秘密情報が含まれないか確認している
- [ ] 証跡に、コミットSHAやツールのバージョンなどの来歴を残している
- [ ] 検査条件が「名前があるか」だけでなく「値が条件を満たすか」まで見ているか、境界値のテストで確認している

## 限界

最後に、WSCLが**保証しないこと**を書いておきます。

- `HARDENED` は「このラボの6つの対策が有効」という意味です。アプリ全体が安全、本番で使える、という意味ではありません。6件PASSでもアプリケーションの安全性は証明されません。
- 両モードとも、平文の固定テスト資格情報、`Secure` なしのCookie（localhostのHTTPで動かすため）、有効期限のないセッション、`GET` によるログアウト、認証なしの `/audit-logs` が、ラボの制約として残っています。
- 検査はHTTPクライアントによる観測で、ブラウザーでの検証ではありません。Cookie属性を確認しても、クロスサイトの操作に対して安全であることやCSRF対策全体の有効性は証明できません。
- CSPの値の強さの評価や、監査ログと特定のイベントの厳密な対応づけは未実装です。
- 教材としての教育効果は評価していません。

## 動かしてみる

Python 3.11以上が必要です。以下はBash用のコマンドです（Windows PowerShellでは `$env:LAB_MODE="HARDENED"` を先に設定してください）。

```bash
git clone https://github.com/moruku36/web-security-control-lab.git
cd web-security-control-lab
python -m venv .venv
source .venv/bin/activate
pip install -e '.[dev]'

# ターミナル1：弱点を残したモード
LAB_MODE=VULNERABLE uvicorn app.main:app --host 127.0.0.1 --port 8000

# ターミナル2：対策を有効にしたモード
LAB_MODE=HARDENED uvicorn app.main:app --host 127.0.0.1 --port 8001

# ターミナル3：両方を検査して比べる
security-lab scan http://127.0.0.1:8000 --format json --expected-mode VULNERABLE
security-lab scan http://127.0.0.1:8001 --format json --expected-mode HARDENED
```

JSONの `checks` と `status_summary` を見て、`VULNERABLE` が6件FAIL、`HARDENED` が6件PASSになっていることを確かめてください。LAB-01の観察には[ワークシート](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/docs/worksheets/LAB01.ja.md)もあります。

:::message alert
ブラウザーで2つのモードを比べるときは、Cookieを消してから切り替えるか、別のブラウザープロファイルを使ってください。Cookieはポート番号で分離されないので、同じブラウザーの別タブでは `127.0.0.1:8000` と `:8001` のセッションが干渉することがあります（[RFC 6265 8.5節](https://www.rfc-editor.org/rfc/rfc6265#section-8.5)）。
:::

## おわりに

セキュリティ検査で大事なのは、緑のチェックを増やすことよりも、**その緑が何を意味しているかを説明できること**だと考えています。WSCLは、その説明を「弱い版も期待どおりに弱い」「検査できなかったことを隠さない」「検査ツール自身も事故を起こさない」という形で、コードとCIに落とし込んだものです。

リポジトリは [MITライセンス](https://github.com/moruku36/web-security-control-lab/blob/4dc4d9dc1adfdf9cfcf07abf3593f04a5722d886/LICENSE) で公開しています。自分の現場の検査に取り入れられそうな部分があれば、ぜひ使ってみてください。

*本稿の図はすべて Mermaid で記述し、[Pele](https://pele.run/) 1.8.0 で SVG に変換したうえで PNG にしています。*
