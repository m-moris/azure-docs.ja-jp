Azure App Service の Mobile Apps 機能は [Azure Notification Hubs] を使用してプッシュを送信するため、ここではモバイル アプリの通知ハブを構成します。

1. [Azure Portal] で、**[App Services]** に移動し、アプリ バックエンドを選択します。 **[設定]** で **[プッシュ]** を選択します。
2. 通知ハブ リソースをアプリに追加するには、**[接続]** を選択します。 ハブを作成することも、既存のハブに接続することもできます。

    ![ハブの構成](./media/app-service-mobile-create-notification-hub/configure-hub-flow.png)

これで、通知ハブが Mobile Apps のバックエンド プロジェクトに接続されました。 後でこの通知ハブを構成して、デバイスにプッシュ通知を送信するプラットフォーム通知システム (PNS) に接続します。

[Azure Portal]: https://portal.azure.com/
[Azure Notification Hubs]: https://azure.microsoft.com/en-us/documentation/articles/notification-hubs-push-notification-overview/
