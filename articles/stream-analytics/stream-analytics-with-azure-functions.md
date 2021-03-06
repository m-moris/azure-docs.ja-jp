---
title: "Azure Stream Analytics ジョブを使用した Azure Functions の実行 | Microsoft Docs"
description: "Stream Analytics ジョブへの出力シンクとして Azure Functions を構成する方法を説明します。"
keywords: "データ出力, ストリーミング データ, Azure Functions"
documentationcenter: 
services: stream-analytics
author: SnehaGunda
manager: kfile
ms.assetid: 
ms.service: stream-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-services
ms.date: 12/19/2017
ms.author: sngun
ms.openlocfilehash: f2f4a8d8cda752dc6ed197b8402119f7cbcaf58f
ms.sourcegitcommit: 28178ca0364e498318e2630f51ba6158e4a09a89
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/24/2018
---
# <a name="run-azure-functions-with-azure-stream-analytics-jobs"></a>Azure Stream Analytics ジョブを使用した Azure Functions の実行 

Azure Stream Analytics で Azure Functions を実行するには、Stream Analytics ジョブへの出力シンクの 1 つとして Functions を構成します。 Functions はイベント ドリブン型コンピューティング オンデマンド エクスペリエンスであり、これにより、Azure またはサード パーティのサービスで発生するイベントによってトリガーされるコードを実装できます。 トリガーに応答する Azure Functions の機能によって、それは Azure Stream Analytics への自然な出力になります。

Stream Analytics では、HTTP トリガーを使用して Functions を呼び出します。 Functions の出力アダプターにより、ユーザーは Functions を Stream Analytics に接続し、Stream Analytics クエリに基づいてイベントをトリガーできるようになります。 

このチュートリアルでは、[Azure Functions](../azure-functions/functions-overview.md) を使用して、Stream Analytics を [Azure Redis Cache](../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md) に接続する方法について説明します。 

## <a name="configure-a-stream-analytics-job-to-run-a-function"></a>関数を実行するための Stream Analytics ジョブの構成を行う 

このセクションでは、Azure Redis Cache にデータを書き込む関数を実行するための Stream Analytics ジョブの構成方法について説明します。 Stream Analytics ジョブは、Azure Event Hubs からイベントを読み取り、関数を呼び出すクエリを実行します。 この関数は、Stream Analytics ジョブからデータを読み取って、Azure Redis Cache に書き込みます。

![Azure サービス間の関係を示す図](./media/stream-analytics-with-azure-functions/image1.png)

このタスクを実現するためには、次の手順が必要です。
* [入力としての Event Hubs で Stream Analytics ジョブを作成する](#create-stream-analytics-job-with-event-hub-as-input)  
* [Azure Redis Cache インスタンスを作成する](#create-an-azure-redis-cache)  
* [Azure Redis Cache にデータを書き込むことができる関数を Azure Functions で作成する](#create-an-azure-function-that-can-write-data-to-the-redis-cache)    
* [出力としての関数で Stream Analytics ジョブを更新する](#update-the-stream-analytic-job-with-azure-function-as-output)  
* [Azure Redis Cache の結果を確認する](#check-redis-cache-for-results)  

## <a name="create-a-stream-analytics-job-with-event-hubs-as-input"></a>入力としての Event Hubs で Stream Analytics ジョブを作成する

チュートリアル「[Azure Stream Analytics の使用 | リアルタイムの不正行為の検出](stream-analytics-real-time-fraud-detection.md)」の説明に従って、イベント ハブを作成し、イベント ジェネレーター アプリケーションを起動し、Stream Analytics ジョブを作成します  (クエリおよび出力を作成する手順は省略します。 Functions 出力を設定するには後続のセクションを参照してください)。

## <a name="create-an-azure-redis-cache-instance"></a>Azure Redis Cache インスタンスを作成する

1. 「[キャッシュの作成](../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache)」に記載の手順を使用して、Azure Redis Cache でキャッシュを作成します。  

2. キャッシュを作成したら、**[設定]** にある **[アクセス キー]** を選択します。 **プライマリ接続文字列**をメモします。

   ![Azure Redis Cache の接続文字列のスクリーンショット](./media/stream-analytics-with-azure-functions/image2.png)

## <a name="create-a-function-in-azure-functions-that-can-write-data-to-azure-redis-cache"></a>Azure Redis Cache にデータを書き込むことができる関数を Azure Functions で作成する

1. Functions ドキュメントの[関数アプリの作成](../azure-functions/functions-create-first-azure-function.md#create-a-function-app)に関するセクションを参照してください。 関数アプリと [HTTP によってトリガーされる関数を Azure Functions で作成](../azure-functions/functions-create-first-azure-function.md#create-function) (CSharp 言語を使用して) する方法が説明されています。  

2. **run.csx** 関数を参照します。 これを以下のコードで更新します  (“\<your redis cache connection string goes here \> (ここに、ご利用の Redis Cache 接続文字列が入ります)” を前のセクションで取得した Azure Redis Cache のプライマリ接続文字列に必ず置き換えます)。  

   ```csharp
   using System;
   using System.Net;
   using System.Threading.Tasks;
   using StackExchange.Redis;
   using Newtonsoft.Json;
   using System.Configuration;

   public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
   {
      log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");
    
      // Get the request body
      dynamic dataArray = await req.Content.ReadAsAsync<object>();

      // Throw an HTTP Request Entity Too Large exception when the incoming batch(dataArray) is greater than 256 KB. Make sure that the size value is consistent with the value entered in the Stream Analytics portal.

      if (dataArray.ToString().Length > 262144)
      {        
         return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
      }
      var connection = ConnectionMultiplexer.Connect("<your redis cache connection string goes here>");
      log.Info($"Connection string.. {connection}");
    
      // Connection refers to a property that returns a ConnectionMultiplexer
      IDatabase db = connection.GetDatabase();
      log.Info($"Created database {db}");
    
      log.Info($"Message Count {dataArray.Count}");

      // Perform cache operations using the cache object. For example, the following code block adds few integral data types to the cache
      for (var i = 0; i < dataArray.Count; i++)
      {
        string time = dataArray[i].time;
        string callingnum1 = dataArray[i].callingnum1;
        string key = time + " - " + callingnum1;
        db.StringSet(key, dataArray[i].ToString());
        log.Info($"Object put in database. Key is {key} and value is {dataArray[i].ToString()}");
       
      // Simple get of data types from the cache
      string value = db.StringGet(key);
      log.Info($"Database got: {value}");
      }

      return req.CreateResponse(HttpStatusCode.OK, "Got");
    }    

   ```

   Stream Analytics では、関数から "HTTP 要求エンティティが大きすぎる" という例外を受け取ると、Functions に送信するバッチのサイズを削減します。 関数内では、次のコードを使用して、Stream Analytics からサイズ超過のバッチが送信されていないことを確認します。 関数で使用する最大バッチ カウントおよび最大バッチ サイズの値が Stream Analytics ポータルに入力した値と矛盾しないことを確認します。

   ```csharp
   if (dataArray.ToString().Length > 262144)
      {        
        return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
      }
   ```

3. 任意のテキスト エディターで、**project.json** という名前の JSON ファイルを作成します。 次のコードを使用し、ローカル コンピューターに保存します。 このファイルには、C# 関数で必要とされる NuGet パッケージの依存関係が含まれています。  
   
   ```json
       {
         "frameworks": {
             "net46": {
                 "dependencies": {
                     "StackExchange.Redis":"1.1.603",
                     "Newtonsoft.Json": "9.0.1"
                 }
             }
         }
     }

   ```
 
4. Azure Portal に戻ります。 **[プラットフォーム機能]** タブで、目的の関数を参照します。 **[開発ツール]** で **[Azure App Service]** を選択します。 
 
   ![App Service エディターのスクリーンショット](./media/stream-analytics-with-azure-functions/image3.png)

5. App Service エディターで、ルート ディレクトリを右クリックし、**project.json** ファイルをアップロードします。 アップロードが成功したら、ページを更新します。 これで、自動生成された **project.lock.json** という名前のファイルが表示されるはずです。 自動生成されたファイルには、project.json ファイルで指定されている .dll ファイルへの参照が含まれています。  

   ![App Service エディターのスクリーンショット](./media/stream-analytics-with-azure-functions/image4.png)

 

## <a name="update-the-stream-analytics-job-with-the-function-as-output"></a>出力としての関数で Stream Analytics ジョブを更新する

1. Azure Portal で Stream Analytics ジョブを開きます。  

2. 目的の関数を参照し、**[概要]** > **[出力]** > **[追加]** の順に選択します。 新しい出力を追加するには、シンク オプションとして **Azure Function** を選択します。 次のプロパティを持つ新しい Functions 出力アダプターが使用できるようになります。  

   |**プロパティ名**|**説明**|
   |---|---|
   |出力エイリアス| 入力を参照するジョブのクエリで使用するわかりやすい名前です。 |
   |インポート オプション| 現在のサブスクリプションから関数を使用できます。あるいは関数が別のサブスクリプションにある場合は、設定を手動で指定できます。 |
   |Function App| Function App の名前です。 |
   |関数| Function App にある関数の名前です (run.csx 関数の名前)。|
   |最大バッチ サイズ|関数に送信される、各出力バッチの最大サイズを設定します。 既定では、この値は 256 KB に設定されます。|
   |最大バッチ カウント|関数に送信される各バッチ内の最大イベント数を指定します。 既定値は 100 です。 このプロパティは省略可能です。|
   |キー|別のサブスクリプションから関数を使用できるようにします。 関数にアクセスするキー値を指定します。 このプロパティは省略可能です。|

3. 出力エイリアスの名前を指定します。 このチュートリアルでは、出力イリアスに **saop1** という名前を付けています (任意の名前を使用することができます)。 その他の詳細を入力します。  

4. Stream Analytics ジョブを開き、クエリを次の内容に更新します  (出力シンクに異なる名前を付けている場合は、文字列 "saop1" を必ず置き換えます)。  

   ```sql
    SELECT 
            System.Timestamp as Time, CS1.CallingIMSI, CS1.CallingNum as CallingNum1, 
            CS2.CallingNum as CallingNum2, CS1.SwitchNum as Switch1, CS2.SwitchNum as Switch2
        INTO saop1
        FROM CallStream CS1 TIMESTAMP BY CallRecTime
           JOIN CallStream CS2 TIMESTAMP BY CallRecTime
            ON CS1.CallingIMSI = CS2.CallingIMSI AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
        WHERE CS1.SwitchNum != CS2.SwitchNum
   ```

5. コマンドラインで次のコマンドを実行して telcodatagen.exe アプリケーションを起動します (使用する形式は `telcodatagen.exe [#NumCDRsPerHour] [SIM Card Fraud Probability] [#DurationHours]` となります)。  
   
   **telcodatagen.exe 1000 .2 2**
    
6.  Stream Analytics ジョブを開始します。

## <a name="check-azure-redis-cache-for-results"></a>Azure Redis Cache の結果を確認する

1. Azure Portal を参照して Azure Redis Cache を見つけます。 **[コンソール]** を説明します。  

2. [Redis キャッシュ コマンド](https://redis.io/commands)を使用して、Redis キャッシュに目的のデータがあることを確認します  (コマンドの形式は Get {key} となります)。例: 

   **Get "12/19/2017 21:32:24 - 123414732"**

   このコマンドにより、指定したキーの値が出力されるはずです。

   ![Azure Redis Cache 出力のスクリーンショット](./media/stream-analytics-with-azure-functions/image5.png)

## <a name="known-issues"></a>既知の問題

Azure Portal では、最大バッチ サイズ/最大バッチ カウントの値を空 (既定値) にリセットしようとしても、保存時には以前に入力した値に戻ります。 この場合は、それらのフィールドに既定値を手動で入力します。

