---
title: Azure Virtual Network | Microsoft Docs
description: "Azure Virtual Network の概念と機能について説明します。"
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: 
tags: azure-resource-manager
ms.assetid: 9633de4b-a867-4ddf-be3c-a332edf02e24
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 3/1/2018
ms.author: jdial
ms.openlocfilehash: fadc1994cd930df36387a5bfb302c00d66f74fad
ms.sourcegitcommit: 782d5955e1bec50a17d9366a8e2bf583559dca9e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/02/2018
---
# <a name="what-is-azure-virtual-network"></a>Azure Virtual Network とは

Azure Virtual Network では、Azure リソースが相互に通信したり、インターネットと通信したりすることができます。 仮想ネットワークでは、リソースが Azure クラウド内の他のリソースから分離されます。 他の仮想ネットワークまたはオンプレミス ネットワークに仮想ネットワークを接続できます。 

Azure Virtual Network では、次に示す広範な機能が提供されます。
- **[分離:](#isolation)** 仮想ネットワークは相互に分離されます。 同じ CIDR (10.0.0.0/0 など) アドレス ブロックを使用する開発、テスト、運用環境に個別の仮想ネットワークを作成できます。 逆に、異なる CIDR アドレス ブロックを使用する複数の仮想ネットワークを作成し、ネットワークをまとめて接続することもできます。 仮想ネットワークは、複数のサブネットに分割できます。 Azure では、仮想ネットワークにデプロイされたリソースの内部名前解決が提供されます。 必要に応じて、Azure の内部名前解決を使用する代わりに、独自の DNS サーバーを使用するよう仮想ネットワークを構成できます。
- **[インターネット通信:](#internet)** 仮想ネットワークにデプロイされている仮想マシンなどのリソースは、既定でインターネットにアクセスできます。 また、必要に応じて、特定のリソースへの着信アクセスを有効にすることもできます。
- **[Azure リソース通信:](#within-vnet)** 仮想ネットワークにデプロイされている Azure リソースは、リソースが異なるサブネットにデプロイされている場合でも、プライベート IP アドレスを使用して相互に通信できます。 Azure では、サブネット、接続された仮想ネットワーク、オンプレミスのネットワークの間に既定のルーティングを提供しているため、ルートの構成と管理は必要ありません。 必要な場合は、Azure のルーティングをカスタマイズできます。
- **[仮想ネットワークの接続:](#connect-vnets)** 仮想ネットワークは、任意の仮想ネットワーク内のリソースが他の仮想ネットワーク内のリソースと通信しすることで、相互に接続できます。
- **[オンプレミスの接続性:](#connect-on-premises)** 仮想ネットワークをオンプレミスのネットワークに接続して、リソースが相互に通信できるようにすることができます。
- **[トラフィックのフィルター処理:](#filtering)** 仮想ネットワーク内のリソースとの間のネットワーク トラフィックを、送信元 IP アドレスおよびポート、送信先 IP アドレスおよびポート、プロトコルでフィルター処理できます。
- **[ルーティング:](#routing)** 必要に応じて、独自のルートを構成するかネットワーク ゲートウェイ経由で BGP ルートをプロパゲートすることで、Azure の既定のルーティングを上書きできます。

## <a name = "isolation"></a>ネットワークの分離とセグメント化

各 Azure [サブスクリプション](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription)と Azure [リージョン](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#region)内に複数の仮想ネットワークを実装できます。 各仮想ネットワークは、他の仮想ネットワークから分離されています。 各仮想ネットワークに対して、次の操作を実行できます。
- パブリックおよびプライベート (RFC 1918) アドレスを使用して、カスタム プライベート IP アドレス空間を指定する。 Azure は、ユーザー自身が割り当てるアドレス空間のプライベート IP アドレスに、仮想ネットワーク内のリソースを割り当てる。
- 仮想ネットワークを 1 つ以上のサブネットに分割し、各サブネットに仮想ネットワークのアドレス空間の一部を割り当てる。
- Azure で提供される名前解決を使用するか、仮想ネットワーク内のリソースで使用する独自の DNS サーバーを指定する。 仮想ネットワークでの名前解決について詳しくは、[仮想ネットワーク内でのリソースの名前解決](virtual-networks-name-resolution-for-vms-and-role-instances.md)に関する記事をご覧ください。

## <a name = "internet"></a>インターネット通信
仮想ネットワーク内のすべてのリソースでは、インターネットへ送信するための通信が可能です。 既定では、リソースのプライベート IP アドレスは、Azure インフラストラクチャによって選択されたパブリック IP アドレスへの送信元ネットワーク アドレス変換 (SNAT) が行われています。 インターネットへの送信接続について詳しくは、「[Azure の送信用接続の詳細](..\load-balancer\load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json)」をご覧ください。 インターネットへの送信接続を回避するために、カスタム ルートまたはトラフィック フィルターを実装できます。

インターネットから Azure リソースへの着信接続、または SNAT なしでインターネットへの送信接続を行うには、リソースにパブリック IP アドレスを割り当てる必要があります。 パブリック IP アドレスについて詳しくは、「[パブリック IP アドレス](virtual-network-public-ip-address.md)」をご覧ください。

## <a name="within-vnet"></a>Azure リソース間の安全な通信

仮想ネットワーク内に仮想マシンをデプロイできます。 仮想マシンは、ネットワーク インターフェイス経由で仮想ネットワークの他のリソースと通信します。 ネットワーク インターフェイスの詳細については、「[ネットワーク インターフェイス](virtual-network-network-interface.md)」をご覧ください。

Azure App Service Environment や Azure Virtual Machine Scale Sets などの仮想ネットワークに、他のいくつかの種類の Azure リソースをデプロイすることもできます。 仮想ネットワークにデプロイできる Azure リソースの詳細な一覧については、[Azure サービスの仮想ネットワーク サービスの統合](virtual-network-for-azure-services.md)に関するページをご覧ください。

一部のリソースは仮想ネットワークにデプロイできませんが、リソースへの通信を仮想ネットワーク内だけに制限することはできます。 リソースへのアクセスを制限する方法について詳しくは、「[仮想ネットワークのサービス エンドポイント](virtual-network-service-endpoints-overview.md)」をご覧ください。 

## <a name="connect-vnets"></a>仮想ネットワークを接続する

いずれかの仮想ネットワーク内のリソースが仮想ネットワークのピアリングを使用して相互に通信することで、仮想ネットワークを相互に接続できます。 異なる仮想ネットワークにあるリソース間の通信の帯域幅と待機時間は、リソースが同じ仮想ネットワーク内にある場合と同じです。 ピアリングの詳細については、「[仮想ネットワーク ピアリング](virtual-network-peering-overview.md)」を参照してください。

## <a name="connect-on-premises"></a>オンプレミス ネットワークに接続する

オンプレミス ネットワークを仮想ネットワークに接続するには、次のオプションを組み合わせて使用します。
- **ポイント対サイト仮想プライベート ネットワーク (VPN):** ネットワーク内の仮想ネットワークと 1 台の PC の間で確立されます。 仮想ネットワークとの接続を確立する各 PC では、個別に接続を構成する必要があります。 この接続の種類は、既存のネットワークへの変更をほとんどまたはまったく必要としないため、Azure を使い始めたばかりのユーザーまたは開発者に適しています。 接続は、SSTP プロトコルを使用して、PC と仮想ネットワーク間にインターネット経由の暗号化された通信を提供します。 トラフィックがインターネットを経由するため、ポイント対サイト VPN の待ち時間は予測できません。
- **サイト間 VPN:** 仮想ネットワークにデプロイされた VPN デバイスと Azure VPN Gateway の間で確立されます。 この接続の種類を使用すると、承認した任意のオンプレミス リソースが仮想ネットワークにアクセスできます。 この接続は IPSec/IKE VPN で、オンプレミスのデバイスと Azure VPN ゲートウェイの間にインターケット経由の暗号化された通信を提供します。 トラフィックがインターネットを経由するため、サイト間接続の待ち時間は予測できません。
- **Azure ExpressRoute:** ExpressRoute のパートナーを介して、ネットワークと Azure の間で確立されます。 この接続はプライベート接続です。 トラフィックはインターネットを経由しません。 トラフィックがインターネットを経由しないため、ExpressRoute 接続の待ち時間は予測可能です。

ここまでに説明したすべての接続オプションについて詳しくは、「[接続トポロジの図](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#diagrams)」をご覧ください。

## <a name="filtering"></a>ネットワーク トラフィックをフィルター処理する
次のオプションのいずれかまたは両方を使用して、サブネット間のネットワーク トラフィックをフィルター処理できます。
- **ネットワーク セキュリティ グループ:** 各ネットワーク セキュリティ グループには、送信元と送信先の IP アドレス、ポート、およびプロトコルでトラフィックをフィルター処理できるようにする受信と送信のセキュリティ規則を複数含めることができます。 ネットワーク セキュリティ グループは、仮想マシン内の各ネットワーク インターフェイスに適用できます。 また、ネットワーク セキュリティ グループは、ネットワーク インターフェイスや他の Azure リソースが含まれているサブネットにも適用できます。 ネットワーク セキュリティ グループの詳細については、[ネットワーク セキュリティ グループ](security-overview.md#network-security-groups)に関するページをご覧ください。
- **ネットワーク仮想アプライアンス:** ネットワーク仮想アプライアンスとは、ファイアウォールなどのネットワーク機能を実行するソフトウェアが動作している仮想マシンです。 利用可能なネットワーク仮想アプライアンスの一覧については、[Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances) で確認してください。 WAN の最適化やその他のネットワーク トラフィック機能を提供するネットワーク仮想アプライアンスも入手できます。 通常、ネットワーク仮想アプライアンスは、ユーザー定義ルートまたは BGP ルートで使用されます。 また、ネットワーク仮想アプライアンスを使用して、仮想ネットワーク間のトラフィックをフィルター処理することもできます。

## <a name="routing"></a>ネットワーク トラフィックをルーティングする

Azure では、仮想ネットワーク内の任意のサブネットに接続されている複数のリソースの相互通信とインターネット通信を可能にするルート テーブルが既定で作成されます。 次のオプションのいずれかまたは両方を実装して、Azure によって作成される既定のルートを上書きできます。
- **ルート テーブル:** サブネットごとにトラフィックのルーティング先を制御するルートを含むカスタム ルート テーブルを作成できます。 カスタム ルーティングについて詳しくは、[カスタム ルーティング](virtual-networks-udr-overview.md#user-defined)に関する記事をご覧ください。
- **BGP のルート:** Azure VPN ゲートウェイまたは ExpressRoute 接続を使用して仮想ネットワークをオンプレミス ネットワークに接続する場合、BGP ルートを仮想ネットワークに伝達できます。

## <a name="next-steps"></a>次のステップ

Azure Virtual Network の概要については以上です。 次に、仮想ネットワークを作成し、その中に Azure Virtual Machines をデプロイして、Azure Virtual Network の機能を使用する方法について説明します。

> [!div class="nextstepaction"]
> [仮想ネットワークの作成](quick-create-portal.md)
