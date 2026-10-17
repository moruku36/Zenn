---
title: "Colab から RunPod まで、オープンモデルのセルフホストで確認できたこと・できなかったこと"
emoji: "🔬"
type: "tech"
topics: ["runpod", "llm", "セルフホスト", "colab", "vllm"]
published: true
---

## 要約

Colab で動かす Qwen の Gradio アプリを出発点に、RunPod の GPU を借りて、オープンウェイトモデルを Open WebUI につなぐ検証を 3 段階で行いました。

- **Phase 1（2026-10-09）**：小型の Qwen2.5-1.5B-Instruct から、Open WebUI 経由で 1 回答を表示。
- **Phase 2（2026-10-15）**：Qwen3.8-27B（BF16）から、Open WebUI 経由で 1 回答を表示。品質や速度のベンチマークは取っていません。
- **Phase 3（2026-10-15）**：GLM-5.3-Flash の GGUF を H200 SXM で起動。コンテナイメージの取得は完了しましたが、モデルの重みをダウンロードしている途中で私が止めました。**モデルの出力は 0 件で、A/B の評価もありません。**

その後、同日の最終振り返りで RunPod 検証を終了しています。公開された最終振り返りは、通常のコード・文書・最終成果物のレビューについて、MiMo-V2.6-Pro の API を「運用候補」として検討する方針を、**未検証**のものとして記録しています（API アカウント・キー・呼び出し・結果は未確認）。

これに加えて、**現在の私自身の説明**として書いておきます（公開リポジトリには書かれていない、私の言葉による補足です）。試すなかで、日常的に繰り返す用途は、コード・文章・成果物の最終レビューに絞られてきました。その利用頻度と GPU の運用負担を考えて、実用上の方向としては MiMo-V2.6-Pro の API を選びました。ただし、API を呼び出したことも、統合したことも、性能を測ったことも、他より安い・優れていると確かめたこともありません。また、セルフホストに取り組んだ作業は無駄ではなく、いまも技術的な学びとして役立っています。

この記事では、構成、試した順序、実際に確認できた範囲、つまずきと切り分け、GPU の時間課金の外にある作業、API とセルフホストを分ける軸、セルフホストが有力なままの場面を整理します。

## この記事の読み方

本文では、記録にある計測値（実測）、表示単価から計算した金額（試算）、私が下した判断、一般的な推論を、文中で言い分けます。

金額は特に断りがない限り、各回の計測区間（Deploy の直前から、Stop 後に $0/時の表示を確認するまで）に当時の表示単価を掛けた**試算**で、確定請求ではありません。計測区間は応答の遅延でも GPU の純粋な稼働時間でもありません。Phase 1 の金額は最終試験だけで先行試験を含まず、いずれの回も請求の突き合わせが済んでいないため、合算した総額は出しません。

根拠は、公開リポジトリの docs へのリンクを文中に付けています。リンク先は `main` の内容です。

## 構成：何をどうつないだか

全体像は次のとおりです（Phase 1・2 の簡略図です。Phase 3 も Pod 側の形は同じで推論サーバーが llama.cpp ですが、モデルの応答までは進んでいません）。

```text
Open WebUI
   │  認証付き HTTPS（RunPod の HTTPS プロキシ経由）
   ▼
RunPod 上の GPU Pod
   └─ 推論サーバー（Phase 1・2：vLLM ／ Phase 3：llama.cpp server）
        └─ オープンウェイトモデル（Qwen → GLM-5.3-Flash の GGUF）
```

- **Phase 1**：認証を先に通すガードを前段に置き、外に許可する経路を `GET /v1/models` と `POST /v1/chat/completions` の 2 つに絞りました。vLLM はその内側（loopback）で待ち受けます（[Phase 1 記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase1-verification-2026-10-09.md)。この記録は日本語のみです）。
- **Phase 2**：vLLM の API キーを RunPod の Secret 参照で注入し、ローカルの Open WebUI の OpenAI 互換接続でつなぎました。未認証の `/v1/models` が 401 になることを送信前に確認しています（[Phase 2 記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase2-verification-2026-10-15.ja.md)）。
- **Phase 3**：digest で固定した llama.cpp の server イメージに、`-hf` 指定で GGUF の重みを取得させる構成でした（[Phase 3 試行記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/phase3-trial-2026-10-15.ja.md)）。

運用の枠組みとして、Phase 2 と 3 では**事前に承認した時間と金額の上限**（Phase 2 は 45 分・$3、Phase 3 は 75 分・$7 と停止条件 6 つ）を決めて始めました。Phase 2 は、画面側の準備を GPU の起動前に終え、Deploy → 1 回の確認 → Stop → 課金速度 $0 の確認 → Terminate と進めています。Phase 3 は、上限の下で Stop → Terminate の順に止めました。

結果を保存してから Pod を止めるジョブ API と実行器も Phase 3 向けに設計していましたが、未実装です（[README.ja.md](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/README.ja.md)）。RunPod のライフサイクル管理コードは別リポジトリの [qwen-runpod-operations](https://github.com/moruku36/qwen-runpod-operations) にあります（Phase 2 記録の記載による。この記事ではそのリポジトリの中身は確認していません）。

## 試した順序

1. **Colab/Gradio の Qwen アプリ。** リポジトリは Qwen の Gradio アプリから始まり、その後 RunPod のオンデマンド推論を試しました（[README.ja.md](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/README.ja.md)）。README には、チャット専用とマルチモーダルの 2 本の Colab ノートブックが載っています。このアプリはいまも残してあります。ただし、既存アプリ全体やマルチモーダルのワークフロー全体を RunPod 上で検証したわけではありません（[最終振り返り](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-retrospective-2026-10-15.ja.md)）。RunPod に移った理由は公開記録に書かれていないため、この記事でも述べません。
2. **Phase 1（2026-10-09）。** 小型モデルで、画面から GPU 上のモデルまでの経路をつなぐ最小の接続試験。
3. **Phase 2（2026-10-15、[PR #44](https://github.com/moruku36/open-llm-multimodal-lab/pull/44)）。** 同じ経路でモデルを 27B 級に大型化した、1 回の短い接続試験。
4. **Phase 3（2026-10-15、[PR #45](https://github.com/moruku36/open-llm-multimodal-lab/pull/45)）。** GLM 系 3 モデル（A：公式モデルの GGUF、B：abliterated の派生 GGUF、C：セキュリティ向けの FP8 版。C は Flash ではなくフルの GLM-5.3 ベース）を同じ課題で比べる計画のうち、GPU なしでできる準備と、A の起動まで。README では過去の計画として扱われています。
5. **終了と方針（2026-10-15、[PR #46](https://github.com/moruku36/open-llm-multimodal-lab/pull/46)）。** 最終振り返りで RunPod 検証を終了し、レビュー用途の運用候補として MiMo-V2.6-Pro API を検討する方針を記録。

## 実際に確認できた範囲

| Phase | 構成 | 確認できたこと | 計測区間と試算 | 確認していないこと |
| --- | --- | --- | --- | --- |
| 1（10-09） | RTX PRO 4000 24 GB／Qwen2.5-1.5B-Instruct（vLLM） | Open WebUI で 1 回答を表示。日本語で返す指示に反し、英語の `Connected successfully.` | 起動前の計測開始から最初の停止確認まで 319.8 秒、約 $0.051（最終試験のみ） | 回答品質、指示遵守、速度 |
| 2（10-15） | A100 SXM 80 GB／Qwen3.8-27B BF16（vLLM） | Open WebUI の OpenAI 互換経路で 1 回答（指示どおり日本語の「接続確認できました」） | Deploy 直前から Stop 後の $0/時表示まで 875.1 秒、約 $0.44（$1.80/時） | 品質、速度、長文脈、画像入力、manual-hook 経路、ライフサイクル自動化 |
| 3（10-15） | H200 SXM 141 GB／GLM-5.3-Flash の GGUF（llama.cpp） | イメージ取得は完了（digest 一致）。重みは総量 108.7 GB のうち、container disk の使用量が約 69 GB（23:29 時点）まで進んだところで取得が約 1.6 GB/分に落ち、中止 | Deploy 直前から $0/時表示まで 1381.7 秒、約 $2.04（$5.31/時） | **モデル出力 0 件、A/B の評価なし**。B は未実施、C は費用試算のみ |

出典：[Phase 1](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase1-verification-2026-10-09.md)、[Phase 2](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase2-verification-2026-10-15.ja.md)、[Phase 3 試行](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/phase3-trial-2026-10-15.ja.md)、まとめは[最終振り返り](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-retrospective-2026-10-15.ja.md)。各回とも、Stop 後に $0/時の表示を確認し、Pod の削除（Terminate）後に支出速度が $0 であることを記録しています。Phase 2 と 3 では Pod 0 台も、Phase 3 ではエンドポイントの 404 も確認しています。

読み方として押さえておきたい点が二つあります。

- Phase 1 と 2 は**接続試験**です。Phase 2 の「1 回答」は、記録されている A100／Qwen3.8-27B BF16 の経路で 1 回返ってきたという意味で、一般的な品質や速度を測ったものではありません。
- Phase 3 の準備には、模擬サーバーで動作を確かめた採点器が含まれます。そこで出る数値は**テスト用フィクスチャの結果で、GLM の実測ではありません**。記事ではその数値を使いません。

## つまずきと切り分け

最終振り返りの整理に従うと、ここで挙げるのは**運用上の不具合**で、モデルの推論・回答能力とは別の話です。プラットフォームやセキュリティ上の制約を、モデルの能力不足と読まないよう、最終振り返りは注意しています。

### Stop したら GPU が戻らなかった（Phase 1 の先行試験）

先行試験ではモデルの準備完了までは進みましたが、Open WebUI での回答は確認できないまま、Stop 後に同じ Pod へ GPU を再割り当てできませんでした。記録によると、RunPod は Stop で GPU の占有を解放するため、元のホストで他の利用者が GPU を使っていれば再開できません。また、container disk は Stop で消えます。そこで最終試験では、画面側の準備を先に済ませ、新しい Pod の起動後に接続先だけを更新する形に変えました（[Phase 1 記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase1-verification-2026-10-09.md)）。

### 引用符が消えて再起動ループ（Phase 2）

JSON 値を取る起動引数を start command に書いたところ、RunPod が引用符を取り除いて渡し、vLLM が引数エラーで終了と再起動を繰り返しました。重みを取得する前だったので、損失は時間だけでした。引数を外して Pod を編集して直しています。`/health` が 404/502 を返しているだけでは、再起動ループと読み込みの遅さを区別できないため、起動直後の System／Container ログを最初の 1 分で見る、という手順を記録に加えました（[Phase 2 記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase2-verification-2026-10-15.ja.md)）。

### 最初の送信が拒否された（Phase 1・2）

Phase 2 では、Open WebUI 0.12.2 の Function Calling の既定が Native で、有効なツールが `tool_choice: "auto"` で送られ、vLLM が拒否しました（サーバー側に tool 関連の設定がないため）。チャットの Function Calling を Legacy にして再生成すると返りました。Phase 1 でも同じ操作で通りましたが、Phase 1 では拒否の原因を実際の送信内容で確定していません。

### 思考モデルと短い max_tokens（Phase 2）

Qwen3.8 は思考から始まるため、max_tokens 64 では本文が出る前に終わり得ます。Open WebUI のカスタムパラメータで `chat_template_kwargs = {"enable_thinking": false}` を送ることで、本文だけが返りました。

### テンプレートの既定値（Phase 2）

RunPod の「vLLM Latest」テンプレートは、API キーの既定が推測可能な形で、`HF_HOME` の既定は永続 volume を前提にしていました。前者は Secret 参照に、後者は container disk 側に変更しています。

### 取得中の `/health` が 502 のまま（Phase 3）

Phase 3 では `/health` が終始 502 でした。llama-server は `-hf` による取得が終わるまで待ち受けないためで、サーバーのエラーや再起動ループは起きていません。進み具合は container disk の使用量で見ており、記録上の推移は次のとおりです。

| 時刻（JST） | container disk の使用量 |
| --- | --- |
| 23:13 | 約 16 GB |
| 23:15 | 約 35 GB（約 9 GB/分） |
| 23:18 | 約 51 GB（約 5.5 GB/分） |
| 23:29 | 約 69 GB（約 1.6 GB/分） |

Deploy 直前の計測開始は 23:08:35.5、イメージ取得完了とコンテナ作成は 23:10:45 です。**イメージ取得は完了していて、止めたのは重みの取得中でした。**事前に承認していた停止条件 3（モデル起動から 25 分以内に準備完了）に達する見込みが立たなくなり（中止時点の見込みでは取得完了が 23:54 頃、準備完了が 23:58 頃）、23:31:12 に私が中止を指示して Stop、その後 Terminate しました（[Phase 3 試行記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/phase3-trial-2026-10-15.ja.md)）。取得速度がなぜ落ちたのかは、記録では特定していません。この記事でも推測は書きません。

最終振り返りは、Phase 3 を「イメージのダウンロードの失敗ではなく、モデルが劣る証拠でも、RunPod が一般に使えない証拠でもない」と整理しています。Phase 1 と 2 は、それぞれ 1 回答まで RunPod 上で到達しました。

### GPU の供給（Phase 3）

Phase 3 では、当初案の H200 NVL が承認の直後に在庫切れとなり、H200 SXM（単価は約 1.4 倍）に切り替えて上限を再承認しました。GPU の供給も変動要因として記録に残っています。

## GPU 課金の外にある作業

GPU の時間課金は目に見える数字ですが、検証の作業はそれ以外にもありました。公開記録から拾えるのは次のとおりです。

- **GPU を使わない準備。** Phase 3 の P3-0 では、モデル台帳と revision の固定（Hugging Face の公開 API で、ダウンロードなし）、GGUF ヘッダー先頭 8 MB の確認、推論イメージが対象モデルの実装を含むことの確認、評価セットの整備、採点器の模擬サーバーでの検証を、GPU なしで済ませました。採点器の初版は HTTP エラーを「拒否」と数えていたため、検証の中で直しています。
- **起動前の画面設定。** Function Calling、max_tokens、思考のオフ、Model IDs などを GPU の起動前に済ませたことで、GPU の稼働時間を最小にできた、と Phase 2 の記録にあります。
- **トラブルシュート。** Phase 2 の 875.1 秒のうち、引数エラーの修正が約 4 分（約 $0.12）でした。計測区間にはこうした作業の時間も含まれます。
- **停止の確認。** Stop 後に $0/時の表示になっているか、Terminate 後に支出速度が $0 か、Pod が 0 台か、外部からエンドポイントに届かない（404）かを確認しました（確認した項目は回によって異なり、上の「実際に確認できた範囲」の出典に書かれています）。
- **請求の突き合わせ。** Phase 3 では、残高表示の減少が $3.17、計算上の試算が約 $2.04 で、約 $1.1 の差がありました。請求反映の遅れの可能性がありますが、未確認です。最終請求は未照合で、報告書以外のストレージや volume の削除、最終請求の確認も未検証と記録されています（[最終振り返り](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-retrospective-2026-10-15.ja.md)）。
- **周辺の課題。** 環境・依存関係の互換性、認証設定、引用符と引数の扱い、Open WebUI の設定、GPU 供給、大容量の重みの転送と保存、停止と請求の確認が必要だった、と最終振り返りに書いています。

最終振り返りには、「約 2 週間」という記述もあります。これは私が体感した暦上の準備の負荷で、**実測した経過時間でも GPU 時間でもベンチマーク時間でもない**と明記されています。この記事ではこれ以上分解しません。上の項目のうち、時間が記録されているのは Phase 2 の引数エラーの約 4 分と各回の計測区間だけで、それ以外の準備や確認にかかった時間は、公開記録で計測されていません。

## API かセルフホストか：判断の軸

ここは三つの層に分けて読んでください。

- **公開記録（PR #46 の最終振り返り）：** 通常のコード・文書・最終成果物のレビューを前提に、MiMo-V2.6-Pro API（`mimo-v2.6-pro`）を運用候補として検討する、という**未検証の方針**。API アカウント、キー、呼び出し、結果は確認しておらず、統合も実装していません。同じワークロードで API とセルフホストを実測比較した結果もありません（[最終振り返り](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-retrospective-2026-10-15.ja.md)）。この位置づけは、記録された当時のまま変わりません。
- **私の現在の説明（公開記録にはない）：** 繰り返す用途がコード・文章・成果物の最終レビューに絞られ、その利用頻度と GPU の運用負担から、実用上の方向として API を選んだ。これは私の判断であって、測定結果ではありません。
- **測定結果：** API についてはありません。呼び出しも、統合も、ベンチマークも、費用や品質の比較もしていません。最安・最良や順位の主張でもありません。

なお最終振り返りは、このモデルに公式のオープンウェイトが公開されており、API 専用モデルではないと記しています。

以下は、判断に使える軸です。いずれも一般的な推論で、私の環境での測定ではありません。MiMo の現在の料金や、提供元のプライバシーポリシーの個別の条項は、この記事の作成時に公式ページで確認していないため、ここでは取り上げません。

1. **費用の比べ方。** 費用は想定する使用量と合わせて比べるものです。Pod は時間（ドル/時）、API はトークン（ドル/トークン）で課金されるため、単価を直接比べることはできません。私が測った Phase 1〜3 の金額は接続試験の計測区間のもので、トークンあたりの費用に換算できません。API の料金は、使う時点で提供元の公式ページを確認してください。
2. **データの扱い。** コードや文書を外部の API に送ってよいかは、送る前に確認すべき問いです。提供元の公式ページで、データの保持・利用の条件、適用される契約、権限、リージョンを確認してください。私自身は、この記事の作成時にそれらを確認していません。API が本質的にプライベートなわけでも、セルフホストが本質的に安全なわけでもありません。
3. **利用の量と継続性。** Pod は持っている時間に、API は使ったトークンに応じて費用が増えます。どちらが安いかは使用量の想定次第で、私は測っていません。（一般的な推論）
4. **運用に割ける時間。** 上の「GPU 課金の外にある作業」は、セルフホストでは自分で負う作業です。GPU を希望どおりに確保できない場面があったことも、記録に残っています。（一般的な推論）
5. **変更への備え。** API では料金や仕様が提供元の都合で変わり得ます。セルフホストでは、モデルや推論基盤の更新を自分で追うことになります。（一般的な推論）

## セルフホストが有力なままの場面

最終振り返りは、セルフホストが向く場合として、固定した再現性、重みの管理、独自の要件、プライバシー設計、継続的な高稼働率を挙げています。ただし、データの取り扱いの確認と、実ワークロードでの費用比較は必要です。ここに私の推論を足すと、次のようになります。

- **特定の重みや量子化を固定したいとき。** Phase 3 の A と B は、同じ量子化・同じ容量で重みだけが違うため、公平に比べられると記録されています（[Phase 3 試行記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/phase3-trial-2026-10-15.ja.md)）。量子化と容量を固定して重みだけを変える比較は、提供元が 1 つの API だけでは組みにくい種類のものです。（一般的な推論）
- **データを外に出せないとき。** 外部サービスに送れない入力があるなら、セルフホストは有力です。ただし、自分の環境でのデータの扱いも確認する必要があります。
- **推論環境そのものを調整したいとき。** サーバー側の設定や起動引数まで触る必要があるなら、セルフホストの領域です。
- **使用量が継続的に多いとき。** 運用の負担を引き受けるだけの稼働率があれば、費用の見え方が変わり得ます。実測の比較は別途必要です。

RunPod 自体は、Phase 1 と 2 で 1 回答まで動きました。Phase 3 の中止は、RunPod が使えないことを示すものではありません。

## 学び（記録にある教訓）

最終振り返りと各試行記録に書かれている教訓のうち、この記事の流れに関わるものを挙げます。いずれも**教訓として残すもので、予定された作業ではありません**。

- **起動引数は事前にローカルで分割結果を確認する。** JSON 値を取る引数は start command に書かず、環境変数か設定ファイルを使う。
- **起動直後のログを最初の 1 分で見る。** `/health` の 404/502 だけでは、再起動ループと読み込み中を区別できない。
- **画面側の設定は GPU の起動前に終える。**
- **大きな重みの取得経路が最大のリスク。** GPU 料金を払いながら 100 GB 級を `-hf` で取得するのは非効率で、速度も不安定だった。同じリージョンのネットワーク volume に安い CPU Pod などで事前取得する、並列ダウンロードと SHA-256 の照合を使う、GPU と volume のリージョンをそろえる、といった案が記録されている。
- **停止条件は取得と読込で分ける。** 今回の停止条件 3 は起動失敗を想定していたが、実際には取得速度で働いた。
- **代替の GPU と上限を最初から 2 段で承認しておく。** 在庫の変動で止まらずに済む。
- **停止の証拠をそろえる。** Stop 後の支出速度 $0、Terminate 後の Pod 0 台、外部からのエンドポイント不達までを 1 回分の完了条件にする。
- **秘密情報の扱い。** API キーは Secret 参照で注入し、テンプレートの既定キーは必ず上書きし、キー・接続先 URL・Pod ID を公開文書に残さない。
- **運用上の不具合と能力を分けて書く。** 設定や供給の問題を、モデルの推論能力の評価として読まない。

## この記事で言っていないこと

- すべての回を合算した総額。各金額は計測区間の試算で、確定請求ではありません。
- どのモデルについても、品質・速度・長文脈・画像入力のベンチマーク結果。
- GLM 系モデルの評価結果（Phase 3 のモデル出力は 0 件）。
- Phase 3 で取得速度が落ちた原因。
- RunPod 検証を終了した理由、Colab から RunPod に移った理由。公開記録に書かれていません。
- MiMo-V2.6-Pro API の動作・品質・遅延・費用の実測、データ取り扱いの確認、統合状況。
- MiMo の現在の料金、提供元のプライバシーポリシーの個別条項（公式ページでは確認していません）。
- RunPod や特定のサービスが使えない、あるいは API が常に安い・楽だという評価。

## 出典

- リポジトリ：[moruku36/open-llm-multimodal-lab](https://github.com/moruku36/open-llm-multimodal-lab)（[README.ja.md](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/README.ja.md)）
- [Phase 1 検証記録（日本語のみ）](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase1-verification-2026-10-09.md)
- [Phase 2 検証記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-phase2-verification-2026-10-15.ja.md)（[PR #44](https://github.com/moruku36/open-llm-multimodal-lab/pull/44)）
- [Phase 3 試行記録](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/phase3-trial-2026-10-15.ja.md)（[PR #45](https://github.com/moruku36/open-llm-multimodal-lab/pull/45)）
- [最終振り返り](https://github.com/moruku36/open-llm-multimodal-lab/blob/main/docs/runpod-retrospective-2026-10-15.ja.md)（[PR #46](https://github.com/moruku36/open-llm-multimodal-lab/pull/46)）
- MiMo の料金・プライバシーポリシー等の提供元ページは、この記事の作成時に確認していないため、出典として挙げていません。
