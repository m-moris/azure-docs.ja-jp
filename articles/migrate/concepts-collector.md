---
title: "Azure Migrate のコレクター アプライアンス | Microsoft Docs"
description: "コレクター アプライアンスの概要とその構成方法について説明します。"
author: ruturaj
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 01/23/2017
ms.author: ruturajd
services: azure-migrate
ms.openlocfilehash: fcf6d2bf13af785eae26ff60035a4754f6ec702e
ms.sourcegitcommit: 782d5955e1bec50a17d9366a8e2bf583559dca9e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/02/2018
---
# <a name="collector-appliance"></a>コレクター アプライアンス

[Azure Migrate](migrate-overview.md) は、オンプレミスのワークロードを評価することによって、Azure への移行を支援します。 この記事では、コレクター アプライアンスの使用方法について説明します。



## <a name="overview"></a>概要

Azure Migrate Collector は、オンプレミスの vCenter 環境の検出に使用できる軽量アプライアンスです。 このアプライアンスは、オンプレミスの VMware マシンを検出し、そのメタデータを Azure Migrate サービスに送信します。

コレクター アプライアンスは、Azure Migrate プロジェクトからダウンロードできる OVF です。 4 コア、8 GB の RAM、および 80 GB のディスク 1 台を装備した VMware 仮想マシンをインスタンス化します。 アプライアンスのオペレーティング システムは Windows Server 2012 R2 (64 ビット) です。

[コレクター VM の作成方法](tutorial-assessment-vmware.md#create-the-collector-vm)の手順を実行してコレクターを作成できます。


## <a name="collector-pre-requisites"></a>コレクターの前提条件

コレクターが Azure Migrate サービスに接続し、検出されたデータをアップロードするためには、いくつかの前提条件チェックに合格する必要があります。 この記事では、各前提条件と、それが必要な理由について説明します。

### <a name="internet-connectivity"></a>インターネット接続

コレクター アプライアンスは、検出されたマシン情報を送信するためにインターネットに接続されている必要があります。 マシンは、次の 2 つの方法のいずれかで、インターネットに接続できます。

1. コレクターをインターネットに直接接続するよう構成することができます。
2. コレクターをプロキシ サーバー経由で接続するよう構成することができます。
    * プロキシ サーバーで認証が必要な場合は、接続設定でユーザー名とパスワードを指定できます。
    * プロキシ サーバーの IP アドレスまたは FQDN は、http://IP アドレス、または http://FQDN の形式である必要があります。 サポートされるのは HTTP プロキシのみです。

> [!NOTE]
> HTTPS ベースのプロキシ サーバーは、コレクターではサポートされていません。

#### <a name="whitelisting-urls-for-internet-connection"></a>インターネット接続用のホワイトリスト URL の作成

指定された設定を使用してコレクターがインターネットに接続できる場合は、前提条件のチェックに合格します。 接続チェックは、次の表に指定されている URL リストに接続することによって検証されます。 URL ベースのファイアウォール プロキシを使用して送信接続を制御している場合は、次の必須の URL を必ずホワイトリストに登録してください。

**URL** | **目的**  
--- | ---
*.portal.azure.com | Azure サービスとの接続性を確認し、時刻の同期の問題を検証するために必要です。

また、チェックでは次の URL への接続も検証されますが、アクセスできない場合もチェックは失敗しません。 次の URL のホワイトリストの作成は任意ですが、前提条件のチェックの工数を軽減するために手動の手順を実行する必要があります。

**URL** | **目的**  | **ホワイトリストを作成しない場合**
--- | --- | ---
*.oneget.org:443 | powershell ベースの vCenter PowerCLI モジュールをダウンロードするために必要です。 | PowerCLI のインストールが失敗します。 モジュールを手動でインストールします。
*.windows.net:443 | powershell ベースの vCenter PowerCLI モジュールをダウンロードするために必要です。 | PowerCLI のインストールが失敗します。 モジュールを手動でインストールします。
*.windowsazure.com:443 | powershell ベースの vCenter PowerCLI モジュールをダウンロードするために必要です。 | PowerCLI のインストールが失敗します。 モジュールを手動でインストールします。
*.powershellgallery.com:443 | powershell ベースの vCenter PowerCLI モジュールをダウンロードするために必要です。 | PowerCLI のインストールが失敗します。 モジュールを手動でインストールします。
*.msecnd.net:443 | powershell ベースの vCenter PowerCLI モジュールをダウンロードするために必要です。 | PowerCLI のインストールが失敗します。 モジュールを手動でインストールします。
*.visualstudio.com:443 | powershell ベースの vCenter PowerCLI モジュールをダウンロードするために必要です。 | PowerCLI のインストールが失敗します。 モジュールを手動でインストールします。

### <a name="time-is-in-sync-with-the-internet-server"></a>時刻がインターネットサーバーと同期されています

サービスに対する要求が認証されるには、コレクターはインターネット時刻サーバーと同期されている必要があります。 時刻を検証できるよう、portal.azure.com の URL にコレクターから接続できる必要があります。 マシンが同期していない場合は、次の手順を実行して、コレクター VM のクロックの時刻を現在の時刻と一致するよう変更する必要があります。

1. VM で管理者用のコマンド プロンプトを開きます。
1. タイム ゾーンを確認するには、w32tm /tz を実行します。
1. 時刻を同期するには、w32tm /resync を実行します。

### <a name="collector-service-should-be-running"></a>コレクター サービスを実行する必要があります

Azure Migrate Collector サービスが、マシンで実行されている必要があります。 このサービスは、マシンが起動したときに自動的に開始されます。 サービスが実行されていない場合は、コントロール パネルから *Azure Migrate Collector* サービスを開始できます。 コレクター サービスは、vCenter サーバーに接続し、マシンのメタデータとパフォーマンス データを収集し、サービスに送信します。

### <a name="vmware-powercli-65"></a>VMware PowerCLI 6.5 

コレクターが vCenter server と通信し、マシンの詳細情報とパフォーマンス データを取得するためには、VMware PowerCLI powershell モジュールをインストールする必要があります。 Powershell モジュールは、前提条件のチェック中に、自動的にダウンロードされ、インストールされます。 自動ダウンロードでは、いくつかの URL がホワイトリスト化されている必要があり、ダウンロードに失敗した場合は、ホワイトリストに URL を追加してアクセスできるようにするか、モジュールを手動でインストールする必要があります。

次の手順を実行して、手動でモジュールをインストールします。

1. インターネットに接続せず PowerCli をコレクターにインストールするには、[このリンク](https://blogs.vmware.com/PowerCLI/2017/04/powercli-install-process-powershell-gallery.html)に記載されている手順を実行してください。
2. インターネットに接続された別のコンピューターに PowerShell モジュールをインストールしたら、そのマシンから、コレクターがインスト―ルされているマシンに VMware.* ファイルをコピーします。
3. 前提条件のチェックを再開し、PowerCLI がインストールされていることを確認します。

## <a name="connecting-to-vcenter-server"></a>vCenter Server への接続

コレクターは、vCenter Server に接続し、仮想マシン、メタデータ、およびパフォーマンス カウンターを取得できる必要があります。 このデータは、評価を計算するためにプロジェクトで使用されます。

1. vCenter Server に接続するには、以下の表に記載されているアクセス許可を持つ読み取り専用アカウントを使用して検出できます。 

    |タスク  |必要なロールおよびアカウント  |アクセス許可  |
    |---------|---------|---------|
    |コレクター アプライアンスによる検出    | 少なくとも読み取り専用ユーザーが必要です。        |データ センター オブジェクト -> 子オブジェクトへのプロパゲート、ロール=読み取り専用         |

2. 指定された vCenter アカウントにアクセスできるデータ センターのみが検出できます。
3. vCenter サーバーに接続するには vCenter の FQDN または IP アドレスを指定する必要があります。 既定では、ポート 443 に接続します。 別のポート番号をリッスンするように vCenter が構成されている場合は、IPAddress:Port_Number または FQDN:Port_Number の形式でサーバー アドレスの一部として指定できます。
4. vCenter Server の統計情報の設定は、デプロイを始める前に、レベル 3 に設定する必要があります。 レベルが 3 未満の場合、検出は実行されますが、ストレージとネットワークのパフォーマンス データは収集されません。 この場合の推奨評価サイズは、CPU およびメモリのパフォーマンス データと、ディスクおよびネットワーク アダプターの構成データのみに基づいて計算されます。 収集されるデータ、およびそのデータの評価への影響について詳しくは、[詳細をお読み](./concepts-collector.md)ください。
5. コレクターには、vCenter サーバーにつながるネットワーク接続が必要です。

> [!NOTE]
> vCenter Server のバージョン 5.5、6.0、および 6.5 のみが公式にサポートされています。

> [!IMPORTANT]
> すべてのカウンターが正しく収集されるように、統計レベルを一般の最高レベル (3) に設定することをお勧めします。 vCenter を低いレベルに設定した場合、完全な形で収集できるカウンターがわずかになり、残りのカウンターがレベル 0 に設定したのと同じ結果になってしまう可能性があります。 その結果、評価が不完全なデータを示す場合があります。 

### <a name="selecting-the-scope-for-discovery"></a>検出範囲の選択

vCenter に接続すると、検出範囲を選択できます。 検出範囲を選択すると、指定した vCenter インベントリ パスからすべての仮想マシンが検出されます。

1. 検出範囲には、データ センター、フォルダー、または ESXi ホストのいずれかを指定できます。 
2. 一度に選択できる検出範囲は 1 つのみです。 より多くの仮想マシンを選択するには、1 つの検出を完了し、新しい検出範囲で検出プロセスを再開します。
3. 選択できるのは、*1000 台未満の仮想マシン*のある検出範囲のみです。 1000 以上の仮想マシンのある検出範囲を選択した場合は、フォルダーを作成してより小さな単位に検出範囲を分割する必要があります。 次に、より小さなフォルダーの検出を個別に実行する必要があります。

## <a name="specify-migration-project"></a>移行プロジェクトの指定

オンプレミスの vCenter に接続し、検出範囲を指定したら、検出および評価に使用する必要がある移行プロジェクトの詳細を指定できます。 プロジェクト ID とキーを指定し、接続します。

## <a name="start-discovery-and-view-collection-progress"></a>検出の開始と収集の進捗状況の表示

検出が開始されると、vCenter 仮想マシンが検出され、そのメタデータとパフォーマンス データがサーバーに送信されます。 進行状況では、次の ID も通知されます。

1. コレクター ID: コレクター マシンに指定されている一意の ID。 この ID は、異なる検出にまたがっても、特定のマシンについて一定です。 この ID は、Microsoft サポートに問題を報告するときに使用できます。
2. セッション ID: 実行中のコレクション ジョブの一意の ID。 検出ジョブが完了すると、ポータルで同じセッション ID を参照できます。 この ID は、コレクション ジョブごとに変更されます。 障害が発生した場合は、Microsoft サポートにこの ID を報告できます。

### <a name="what-data-is-collected"></a>どのデータが収集されますか。

コレクション ジョブでは、選択した仮想マシンに関する次の静的メタデータを検出します。 

1. VM の表示名 (vCenter 上の)
2. VM のインベントリ パス (vCenter でのホスト/フォルダー)
3. IP アドレス
4. MAC アドレス
5. コア、ディスク、NIC の数
6. RAM、ディスク サイズ
7. 次の表に記載されている、VM、ディスク、およびネットワークのパフォーマンス カウンター。

次の表は、収集されるパフォーマンス カウンターと、特定のカウンターが収集されなかった場合に影響がある評価結果を示します。

|カウンター                                  |Level    |デバイスごとのレベル  |評価の影響                               |
|-----------------------------------------|---------|------------------|------------------------------------------------|
|cpu.usage.average                        | 1       |該当なし                |推奨される VM サイズとコスト                    |
|mem.usage.average                        | 1       |該当なし                |推奨される VM サイズとコスト                    |
|virtualDisk.read.average                 | 2       |2                 |ディスク サイズ、ストレージ コスト、VM サイズ         |
|virtualDisk.write.average                | 2       |2                 |ディスク サイズ、ストレージ コスト、VM サイズ         |
|virtualDisk.numberReadAveraged.average   | 1       |3                 |ディスク サイズ、ストレージ コスト、VM サイズ         |
|virtualDisk.numberWriteAveraged.average  | 1       |3                 |ディスク サイズ、ストレージ コスト、VM サイズ         |
|net.received.average                     | 2       |3                 |VM サイズとネットワーク コスト                        |
|net.transmitted.average                  | 2       |3                 |VM サイズとネットワーク コスト                        |

> [!WARNING]
> 統計レベルを高く設定した場合は、パフォーマンス カウンターを生成するのに最大 1 日かかります。 そのため、1 日経ってから検出を実行することをお勧めします。

### <a name="time-required-to-complete-the-collection"></a>収集を完了するために必要な時間

コレクターは、マシン データのみを検出し、プロジェクトに送信します。 プロジェクトによっては、ポータルに検出されたデータが表示され、評価の作成を開始できるまで時間がかかる場合があります。

選択された検出範囲にある仮想マシンの数によっては、プロジェクトに静的メタデータを送信するまで最大 15 分かかる場合があります。 静的メタデータがポータルに表示されると、ポータルにマシンの一覧が表示され、グループの作成を開始できます。 コレクション ジョブが完了し、プロジェクトでデータが処理されるまで、評価は作成できません。 選択された検出範囲にある仮想マシンの数によっては、コレクション ジョブがコレクターで完了してからポータルにパフォーマンス データが表示されるまで、最大 1 時間かかることがあります。

## <a name="next-steps"></a>次の手順

[オンプレミスの VMware VM のアセスメントを設定する](tutorial-assessment-vmware.md)
