---
title: "Azure Reserved Instance と Savings Planの違いと使い分け"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Azure, Cost, Reserved Instance, Savings Plan]
published: true
---
## はじめに
- Azure の Cost Assesment を実施する中で少し分かり辛かったReserved Instance とSavings Plan の違いについて簡単にまとめてみました。

- Azure Reserved InstanceとSavings Planは、どちらもクラウドサービスの利用に対して1年間または3年間のコミットメントをすることで割引を受けることができる方法です。しかし、両者には以下のような違いがあります。

| 項目 | Azure Reserved Instance | Azure Savings Plan |
| --- | --- | --- |
| 割引対象 | 特定の仮想マシンタイプやリージョンに限定される | 参加するコンピュートサービス全体に適用される |
| 割引率 | 最大72% | 最大65% |
| 柔軟性 | 仮想マシンタイプやリージョンを変更すると割引が適用されない | 仮想マシンタイプやリージョンを変更しても割引が適用される |
| 支払い方法 | 一括前払いまたは月額分割払い | 一括前払いまたは月額分割払い |
| 適用範囲 | サブスクリプション、リソースグループ、管理グループ、Azureアカウントのいずれかに限定される | サブスクリプション、リソースグループ、管理グループ、Azureアカウントのいずれかに適用できる |

使い分けとしては

- 安定して継続的に実行されるワークロードで、仮想マシンタイプやリージョンに変更がない場合は、Azure Reserved Instanceを選択するとより高い割引率を得られます。
- 動的なワークロードで、異なるサイズの仮想マシンやデータセンターリージョンを頻繁に変更する場合は、Azure Savings Planを選択すると柔軟に割引が適用されます。

## 交換と払い戻し

Azure Reserved Instance の交換と払戻しについて

- 同じ種類の予約製品（例えば、仮想マシンやSQLデータベースなど）であれば相互に交換できます。
- 異なるリージョンや期間に対しても交換できます。
- 交換は、キャンセル、払戻し、新規購入として処理されます。そのため、交換すると予約期間がリセットされます。
- 払戻しは、12か月間のローリングウィンドウで50,000米ドルを超えないように制限されています。
- 2024年1月1日以降に購入されたAzure Reserved Instanceは、仮想マシンタイプやリージョンの交換ができなくなります。ただし、2024年1月1日以前に購入されたAzure Reserved Instanceは、ポリシー変更後ももう一度だけ交換できます。

Azure Savings Planの交換と払戻しについて

- 同じ種類のコンピュートサービス（例えば、仮想マシンやApp Serviceなど）であれば相互に交換できます。
- 異なるリージョンや期間に対しても交換できます。
- 交換は、キャンセル、払戻し、新規購入として処理されます。そのため、交換すると予約期間がリセットされます。
- 払戻しは、12か月間のローリングウィンドウで50,000米ドルを超えないように制限されています。

## その他
- Savings Planは出てきて間もないサービスなので今後も追加情報や変更が入りそうです


## 参考サイト. 
https://learn.microsoft.com/ja-jp/azure/cost-management-billing/savings-plan/decide-between-savings-plan-reservation  


https://azure.microsoft.com/ja-jp/pricing/offers/savings-plan-compute/#benefits-and-features

https://learn.microsoft.com/ja-jp/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations

