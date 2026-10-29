---
title: "AIにコードを書かせる時代のDB選定：Supabase・Neon・MongoDB・Upstash・Convexを要件で選ぶ"
emoji: "🗄️"
type: "tech"
topics: ["データベース", "supabase", "neon", "mongodb", "convex"]
published: false
---

![記事ヘッダー画像](https://static.zenn.studio/user-upload/79b3bf1d7b6e-20261029.png)

## 1. はじめに
AIがコードの大半を書く時代になり、DB選定の重心は変わってきました。無料で試せる速さ、AIが扱いやすい型とドキュメント、そして認証などの周辺機能を買うか自作するかが主な軸になります。この記事ではSupabase・Neon・MongoDB Atlas・Upstash Redis・Convexを作るものとの相性で整理します。

## 2. まず整理：5つは同じ種類ではない
この5つを同じ土俵のDBとして比べると迷子になりやすいです。SupabaseとConvexはDBを含むBaaS型、NeonはサーバーレスPostgresが核で周辺機能も持つ構成です。MongoDB AtlasはドキュメントDB本体、Upstash Redisはキャッシュやレート制限を受け持つ補助役と捉えると見通しが良いです。

## 3. Supabaseの要点
SupabaseはPostgres＋認証（Auth）＋ファイル保存（Storage）＋Realtime＋Edge FunctionsのBaaSです。オープンソースでセルフホストも選べます。Freeは$0でDB 500MB、ファイル1GB、認証50,000 MAU、アクティブなプロジェクトは2つまでとなります。認証つきの普通のWebアプリを最速で立ち上げたい場合との相性が良いです。Proは$25/月でDB 8GB、認証100,000 MAU、毎日バックアップになります。

参考：https://supabase.com/pricing

## 4. Neonの要点
NeonはサーバーレスPostgresが核で、現在はManaged Better AuthやObject Storage、Functions、Realtime（Beta）、AI Gatewayまで含みます。特徴はブランチによるDBの分岐コピーとオートスケール、pgvectorなどの拡張にあります。Freeは$0で期間制限なし・カード不要、プロジェクト100、100 CU-時間/月・プロジェクト、ストレージ1GB/プロジェクトです。SQLとPostgresの資産を活かしつつ、プレビュー環境を量産したい場合に向いています。Launchは従量課金で公式の目安$15/月、$0.106/CU-時間となります。

参考：https://neon.com/pricing

## 5. MongoDB Atlasの要点
MongoDB AtlasはドキュメントDBが中心で、認証やAPI層は自前かApp Servicesの組み合わせで作ります。データの形が定まらない段階で、ドキュメントの柔軟さを活かしたい場合との相性が良いです。FreeのM0はFree foreverで512MB、共有vCPU/RAMとなります。Atlas SearchやVector Searchを組み合わせた検索用途も同じ基盤で検討できます。Flexは$8〜30/月、DedicatedはM10で$0.08/時からです。

参考：https://www.mongodb.com/pricing

## 6. Upstash Redisの要点
Upstash RedisはサーバーレスRedisで、キャッシュ・セッション・レート制限の補助役です。主系DBの横に足して体感速度や制御を改善したい場合に向いています。Freeは$0で250MB、月500Kコマンド、無料DBは1つとなります。公式FAQではPOSTリクエストだけでサインアップなしに無料Redisを即発行でき、AIエージェントがすぐ使える設計だと説明されています。従量は$0.2/100Kコマンド、固定プランは$10/月からです。

参考：https://upstash.com/pricing

## 7. Convexの要点
ConvexはTypeScriptでクエリ・ミューテーション・アクションを書くリアクティブなバックエンドです。DBはドキュメント型でSQLではなく、リアルタイム同期やファイル保存、スケジュール実行、認証連携、ベクトル検索を内蔵します。React・TypeScriptでリアルタイム同期したい場合との相性が良いです。Free/Starterは$0＋超過分は従量で、関数呼び出し月100万回、DB 0.5GB、ファイル1GB、AIアプリ生成Chefのトークン付きとなります。Professionalは$25/開発者/月で関数呼び出し月2,500万回、DB 50GBになります。オープンソース化されセルフホストも選べます。

参考：https://www.convex.dev/pricing ／ https://docs.convex.dev/production/state/limits

## 8. 選び方：作るもので決める
作るものから逆引きすると選びやすいです。認証つきの普通のWebアプリを最速で作りたい場合はSupabase、SQLとPostgresの資産を活かしプレビュー環境を量産したい場合はNeonが候補になります。データの形が定まらない場合はMongoDB Atlas、主系DBは別にありキャッシュやレート制限を足したい場合はUpstash Redis、React・TypeScriptでリアルタイム同期したい場合はConvexが候補になります。

![作るもので決めるDB選定フローチャート](https://static.zenn.studio/user-upload/dae9965c44e8-20261029.png)

AIに書かせる観点では、Postgresの型が欲しいか、ドキュメントの柔軟さが欲しいか、周辺機能まで一括で任せたいかを先に決めると手戻りが少ないです。要件が変われば組み合わせを変えれば良く、最初から一つに絞り込む必要はありません。

## 9. 比較表
| サービス | 種類 | データモデル | 認証 | リアルタイム | 無料枠 | 有料の入口 |
|---|---|---|---|---|---|---|
| Supabase | BaaS | Postgres | Auth内蔵 | Realtimeあり | $0でDB 500MB、ファイル1GB、認証50,000 MAU、プロジェクト2つまで | Pro $25/月でDB 8GB、認証100,000 MAU |
| Neon | サーバーレスPostgres＋周辺機能 | Postgres | Managed Better Authを含む | Realtime（Beta）を含む | $0でプロジェクト100、100 CU-時間/月・プロジェクト、ストレージ1GB/プロジェクト | Launchは従量課金で$0.106/CU-時間 |
| MongoDB Atlas | ドキュメントDB | ドキュメント型 | 自前かApp Servicesを組み合わせる | — | M0は無料で512MB、共有vCPU/RAM | Flex $8〜30/月、DedicatedはM10で$0.08/時から |
| Upstash Redis | サーバーレスRedis・補助役 | KVS | 主系の仕組みに合わせる | キャッシュ等の用途で使う | $0で250MB、月500Kコマンド、無料DBは1つ | 従量$0.2/100Kコマンド、固定$10/月から |
| Convex | リアクティブなバックエンド | ドキュメント型でSQLではない | 認証連携を内蔵 | リアルタイム同期を内蔵 | $0＋従量で関数月100万回、DB 0.5GB、ファイル1GB | Professional $25/開発者/月で関数月2,500万回、DB 50GB |

## 10. 注意点
Supabaseの無料枠は1週間アクセスがないと一時停止するため、放置した検証用とは相性を確認したいです。Upstashは主系DBの代わりではなく、主系DBと組ませて使う位置づけです。ConvexはSQLではないため、既存のSQL資産をそのまま活かしたい場合は別候補と比べたいです。料金と制限は変わるため、最終確認は必ず公式で行いたいです。

## 11. まとめ
BaaSが欲しいか、Postgresを使い倒したいか、形の変わりやすさを優先するかで候補は自然に絞れます。補助が必要ならUpstashを足し、リアルタイム同期を核にするならConvexを検討するのが分かりやすいです。まずは無料枠でAIに触らせ、周辺機能と運用の手触りで決めるのが近道です。

## 出典
- https://supabase.com/pricing
- https://neon.com/pricing
- https://www.mongodb.com/pricing
- https://upstash.com/pricing
- https://www.convex.dev/pricing
- https://docs.convex.dev/production/state/limits

---

P.S. この記事は、AIエージェントの Muse（フワッチ）と一緒に書きました。もし Muse を使ってみたくなったら、招待コード EOA1TR を入力してもらえると嬉しいです。https://muse.ai/join
