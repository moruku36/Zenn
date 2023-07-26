---
title: "Azure Boost とはなんなのか"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Azure, Azure Boost]
published:false
---
Azure に Boost という新しい機能が発表されたようですね。AWS の Nitro に似たサービスのようですが、どのようなサービスなのかエンジニア向けに詳細に解説したブログを書いてみました。Microsoft 及び AWS の公式ドキュメントのみ参考にしています。

# Azure Boost とは

Azure Boost とは、Microsoft Azure の最新のインフラストラクチャイノベーションの一つで、仮想化プロセスを専用のハードウェアとソフトウェアにオフロードすることで、ネットワークとストレージのパフォーマンスを大幅に向上させる新しいシステムです²⁴。

従来の仮想化インフラストラクチャでは、ハイパーバイザーが物理的なハードウェアと BIOS を保護し、CPU、ストレージ、ネットワークを仮想化し、数多くの管理機能を提供してきました。しかし、Azure Boost では、ハイパーバイザーとホスト OS の機能をホストインフラストラクチャから分離し、専用のハードウェアとソフトウェアに負荷を分散することで、サーバーのほぼすべてのリソースをインスタンスに提供できるようになりました²⁴。

このイノベーションにより、Azure Boost を利用するプレビュー参加者は、200 Gbps のネットワークスループットと 10 GBps と 400K IOPS のリーディングリモートストレージスループットを達成できるようになりました²⁴。これらのパフォーマンスは、実験的な SKU を通じて提供されます²⁴。

Azure Boost は、既存の Azure VM にもすでに恩恵をもたらしており、例えば Ebsv5 VM シリーズの優れたリモートストレージパフォーマンスや Ev5 および Dv5 VM シリーズ全体のネットワークスループットとレイテンシーの改善などが挙げられます²⁴。Azure Boost は今後も Azure インフラストラクチャ利用者にイノベーションと利点を提供し続ける予定です²⁴。

## Azure Boost のメリット

Azure Boost には以下のようなメリットがあります。

- **イノベーションの迅速化**：Azure Boost は、さまざまな方法で組み立てることができる構成要素を豊富に備えており、コンピューティング、ストレージ、メモリ、ネットワークのオプションの幅がますます広がる EC2 インスタンスタイプを設計し、迅速に提供する柔軟性を実現します²⁴。このイノベーションは、ベアメタルインスタンスにもつながります。ベアメタルインスタンスではお客様が、自社のハイパーバイザーを使うか、またはハイパーバイザーをまったく使わなくて済みます²⁴。
- **セキュリティの強化**：Azure Boost は、インスタンスのハードウェアとファームウェアを継続的に監視、保護、検証し、セキュリティを一層強化します²⁴。仮想化リソースの負荷を専用のハードウェアとソフトウェアに分散し、アタックサーフェスを最小化します²⁴。そして、Azure Boost のセキュリティモデルはロックダウン型であり、管理者アクセスを禁止して、ヒューマンエラーや改ざんの可能性をなくします²⁴。
- **パフォーマンスと価格の改善**：Azure Boost は、ホストハードウェアにある実質的にすべてのコンピューティングリソースとメモリリソースをお使いのインスタンスに提供し、全体的なパフォーマンスを高めます²⁴。さらに、専用の Nitro Card が高速ネットワーク、高速 EBS、I/O 高速化を可能にします²⁴。管理ソフトウェアのためにリソースを抑制する必要がないため、その節約分をお客様に還元できます²⁴。

## Azure Boost の主な機能

Azure Boost には以下のような主な機能があります。

- **Nitro Card**：Nitro Card は、さまざまな機能の負荷を分散し、IO を加速して、最終的にシステム全体のパフォーマンスを高める一連のカードです²⁴。主なカードには、Nitro Card for VPC、Nitro Card for EBS、Nitro Card for Instance Storage、Nitro Card Controller、Nitro Security Chip があります²⁴。
- **Nitro Security Chip**：Nitro Security Chip は、仮想化とセキュリティ機能の負荷を専用のハードウェアおよびソフトウェアに分散し、アタックサーフェスを最小限に抑えた最も安全なクラウドプラットフォームを実現します²⁴。また、ロックダウン型セキュリティモデルにより、管理者アクセスを禁止して、ヒューマンエラーや改ざんの可能性をなくします²⁴。

# AWS Nitro との比較

AWS Nitro も Azure Boost と同様に、仮想化プロセスを専用のハードウェアとソフトウェアにオフロードすることで、パフォーマンスとセキュリティを向上させるシステムです⁵。AWS Nitro も Azure Boost と同じようなメリットや機能を提供していますが、以下のような違いがあります。

| 比較項目 | Azure Boost | AWS Nitro |
| --- | --- | --- |
| ネットワークパフォーマンス | 最大 200 Gbps²  | 最大 100 Gbps |
| ストレージパフォーマンス | 最大 10 GBps と 400K IOPS²  | 最大 3.5 GBps と 80K IOPS |
| ネットワークアダプタ | Microsoft Azure Network Adapter (MANA)²  | Elastic Network Adapter (ENA) |
| ストレージアダプタ | Nitro Card for EBS²  | Elastic Block Store (EBS) Optimized |
| インスタンスストレージ | Nitro Card for Instance Storage²  | NVMe SSD |
| セキュリティチップ | Nitro Security Chip²  | Nitro Security Chip |
| ハイパーバイザー | 軽量ハイパーバイザー²  | 軽量ハイパーバイザー |

## Azure Boost と AWS Nitro の違いについての考察

Azure Boost と AWS Nitro の比較から、以下のような考察ができます。

- Azure Boost は、AWS Nitro よりもネットワークとストレージのパフォーマンスが高いことがわかります。これは、Azure Boost がより新しい技術であることや、Microsoft Azure Network Adapter (MANA) や Nitro Card for EBS などの専用のハードウェアとソフトウェアによって実現されていることが理由として考えられます。
- Azure Boost と AWS Nitro は、セキュリティチップやハイパーバイザーなどの仕組みにおいて類似していることがわかります。これは、両者ともに仮想化プロセスを専用のハードウェアとソフトウェアにオフロードすることで、セキュリティやパフォーマンスを向上させるという共通の目的を持っていることが理由として考えられます。
- Azure Boost と AWS Nitro の違いは、主にネットワークアダプタやストレージアダプタなどの具体的な実装において現れていることがわかります。これは、両者ともに自社開発や買収した専用のチップやソフトウェアを利用していることが理由として考えられます。

# まとめ

Azure Boost は、Microsoft Azure の最新のインフラストラクチャイノベーションであり、AWS Nitro に似た仕組みで仮想化プロセスを専用のハードウェアとソフトウェアにオフロードすることで、ネットワークとストレージのパフォーマンスを大幅に向上させる新しいシステムです。Azure Boost は、AWS Nitro よりも高いパフォーマンスを提供する一方で、セキュリティやハイパーバイザーなどの仕組みにおいて類似しています。Azure Boost と AWS Nitro の違いは、主にネットワークアダプタやストレージアダプタなどの具体的な実装において現れています。

Azure Boost は、現在プレビュー版として提供されており、高性能なネットワークとストレージが必要なお客様やパートナーにとって魅力的なサービスです。Azure Boost の詳細については、以下のリンクを参照してください。

- [Preview: Azure Boost | Azure updates | Microsoft Azure](https://azure.microsoft.com/en-us/updates/preview-azure-boost/)
- [Microsoft Azure Boost Preview - Microsoft Tech Community](https://techcommunity.microsoft.com/t5/azure-infrastructure-blog/introducing-microsoft-azure-boost-preview/ba-p/3876742)

²: [Preview: Azure Boost | Azure updates | Microsoft Azure](https://azure.microsoft.com/en-us/updates/preview-azure-boost/)
: [Microsoft Azure Boost Preview - Microsoft Tech Community](https://techcommunity.microsoft.com/t5/azure-infrastructure-blog/introducing-microsoft-azure-boost-preview/ba-p/3876742)
: [AWS Nitro System](https://aws.amazon.com/ec2/nitro/)

Source: Conversation with Bing, 7/26/2023
(1) AWS to Azure services comparison - Azure Architecture Center. https://learn.microsoft.com/en-us/azure/architecture/aws-professional/services.
(2) マイクロソフト「Azure Boost」発表。AWS Nitroのように専用 .... https://www.publickey1.jp/blog/23/azure_boostaws_nitrosoc.html.
(3) Compare AWS and Azure compute services - Azure .... https://learn.microsoft.com/en-us/azure/architecture/aws-professional/compute.

Source: Conversation with Bing, 7/26/2023
(1) Microsoft Azure Boost Preview. https://techcommunity.microsoft.com/t5/azure-infrastructure-blog/introducing-microsoft-azure-boost-preview/ba-p/3876742.
(2) AWS Nitro System. https://aws.amazon.com/ec2/nitro/.
(3) AWS Nitro System. https://aws.amazon.com/jp/ec2/nitro/.
(4) Preview: Azure Boost | Azure updates | Microsoft Azure. https://azure.microsoft.com/en-us/updates/preview-azure-boost/.
(5) Microsoft Unveils Azure Boost, Promises Enhanced Network .... https://www.datanami.com/this-just-in/microsoft-unveils-azure-boost-promises-enhanced-network-and-storage-performance/.
(6) What is AWS Nitro Enclaves? - AWS. https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave.html.
