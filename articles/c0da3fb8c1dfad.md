---
title: "Azure Boost のメリットとは？"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Azure, Azure Boost, Microsoft, AWS ]
published:true
---

Azure Boost のメリットとは？

今回は、2023年7月にプレビュー版が発表された Azure Boost のメリットについて個人的に整理してみました。殆ど以下の公式ドキュメントの翻訳になります

https://techcommunity.microsoft.com/t5/azure-infrastructure-blog/introducing-microsoft-azure-boost-preview/ba-p/3876742


Azure Boost とは、ハイパーバイザーやホスト OS が行っていた仮想化プロセスを、ネットワーキング、ストレージ、ホスト管理などの専用のハードウェアとソフトウェアにオフロードする新しいシステムです。これにより、Azure Boost は以下のようなメリットを提供します。

- ネットワーク性能の向上
- ストレージ性能の向上
- セキュリティの向上
- メンテナンスの軽減

それぞれのメリットについて詳しく見ていきましょう。

## ネットワーク性能の向上

Azure Boost VM は、プレビュー中に最大 200 Gbps のネットワークスループットを達成できます。これは、他の Azure VM と比べて2倍の性能向上を意味します。この高いネットワーク性能は、Microsoft Azure Network Adapter (MANA) という重要なコンポーネントによって可能になります。MANA は、Azure の要求に合わせて最適化されたネットワーク性能を提供するために開発されたデバイスです。MANA は、仮想マシンに高い信頼性をもたらし、Linux と Windows OS に対応した安定したドライバーを提供します。これにより、Azure Boost システムは、より高速で効率的なデータ転送を可能にし、高いネットワーク可用性と安定性を実現できます。また、MANA の一貫したドライバーインターフェースは、将来のハードウェア変更に対しても互換性を保証し、今後の Azure の性能向上や機能追加にもシームレスに対応できます。MANA を統合することで、Azure Boost のネットワークメリットを最大限に活用できます。

## ストレージ性能の向上

Azure Boost VM は、プレビュー中に最大 10 GBps と 400K IOPS のリーモートストレージスループットを達成できます。これは、現在利用可能な最速のストレージワークロードを可能にします。この高いストレージ性能は、Azure Storage Accelerator (ASA) という重要なコンポーネントによって可能になります。ASA は、Azure の要求に合わせて最適化されたストレージ性能を提供するために開発されたデバイスです。ASA は、仮想マシンから Azure Storage へのアクセスを高速化し、低遅延かつ高スループットなストレージ操作を実現します。また、ASA の一貫したドライバーインターフェースは、将来のハードウェア変更に対しても互換性を保証し、今後の Azure の性能向上や機能追加にもシームレスに対応できます。ASA を統合することで、Azure Boost のストレージメリットを最大限に活用できます。

## セキュリティの向上

Azure Boost は、ハイパーバイザーとホスト OS の機能をホストインフラから分離することで、セキュリティを向上させます。これは、仮想マシンとホストインフラの間にもう一つの論理的な分離レイヤーを追加することを意味します。この分離レイヤーは、仮想マシンに対する潜在的な攻撃や妨害のリスクを低減し、Azure の信頼性と安全性を高めます。また、Azure Boost は、MANA と ASA のドライバーを通じて、仮想マシンのネットワークとストレージのアクセスを制御し、不正なアクセスやデータ漏洩のリスクを低減します。Azure Boost は、セキュリティに敏感なお客様やパートナーにとって、安心して Azure を利用できるシステムです。

## メンテナンスの軽減

Azure Boost は、将来の Azure ソフトウェアとハードウェアのアップグレードに伴うメンテナンスの影響を軽減します。これは、仮想化プロセスを専用のハードウェアとソフトウェアにオフロードすることで、ホストインフラの負荷を減らし、アップグレード時のダウンタイムやパフォーマンス低下を最小限に抑えることを意味します。また、Azure Boost は、MANA と ASA のドライバーを通じて、仮想マシンのネットワークとストレージのインターフェースを安定化し、将来のハードウェア変更に対しても互換性を保証します。これにより、Azure Boost は、お客様やパートナーが Azure の最新の機能や性能を迅速かつ容易に利用できるようにします。

---

以上が、Azure Boost のメリットについてのまとめです。高いネットワークやストレージ性能が必要なお客様やパートナー、特に Data Plane Development Kit (DPDK) を使用している方には有益なアップデートのようです。

プレビューへのアクセス申請は以下のフォームから可能です。

https://forms.office.com/pages/responsepage.aspx?id=v4j5cvGGr0GRqy180BHbR4ALpcxfcMZNi-25j9GfQ1tUNzZLMVg4SklWS1NIVFFENzg2VkRDR01NMS4u

## 考察
このサービスが出てきた背景には AWS Nitoro の存在があるようですが、自分はまだAWS の機構に明るくないのでいつか比較してみたいと思います。Azure Portal 上から直接変更できるリソースではないので検証しずらいですが、個人的に興味のある機構なので今後追加アナウンスや研究論文がだできたら追記していきたいと思います。

## 参考　　

https://azure.microsoft.com/en-us/updates/preview-azure-boost/


https://techcommunity.microsoft.com/t5/azure-infrastructure-blog/introducing-microsoft-azure-boost-preview/ba-p/3876742

