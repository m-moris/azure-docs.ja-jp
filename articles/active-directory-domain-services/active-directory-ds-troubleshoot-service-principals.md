---
title: "Azure Active Directory Domain Services: サービス プリンシパル構成のトラブルシューティング | Microsoft Docs"
description: "Azure AD Domain Services 向けのサービス プリンシパル構成のトラブルシューティング"
services: active-directory-ds
documentationcenter: 
author: eringreenlee
manager: 
editor: 
ms.assetid: f168870c-b43a-4dd6-a13f-5cfadc5edf2c
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/19/2018
ms.author: ergreenl
ms.openlocfilehash: 7388bb291f665f195355a01d19a82cba9ed453eb
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/21/2018
---
# <a name="troubleshoot-invalid-service-principal-configuration-for-your-managed-domain"></a>管理対象ドメインの無効なサービス プリンシパル構成のトラブルシューティング

この記事は、サービス プリンシパル関連の構成エラーのトラブルシューティングと解決に役立ちます。次のような警告メッセージは、このようなエラーにより表示されます。

## <a name="alert-aadds102-service-principal-not-found"></a>アラート AADDS102: サービス プリンシパルが見つかりません
**アラート メッセージ:** "*Azure AD Domain Services が正常に機能するために必要なサービス プリンシパルが Azure AD ディレクトリから削除されています。この構成は、管理対象ドメインに対する監視、管理、修正のプログラム適用、および同期を行うための Microsoft の能力に影響します。*"

[サービス プリンシパル](../active-directory/develop/active-directory-application-objects.md)は、Microsoft が管理対象ドメインを管理、更新、および維持するために使用するアプリケーションです。 それらが削除されると、ドメインにサービスを提供する Microsoft の機能が中断します。 


## <a name="check-for-missing-service-principals"></a>不足しているサービス プリンシパルを確認する
次の手順を使用して、どのサービス プリンシパルを再作成する必要があるかを判断してください。

1. Azure Portal で [[Enterprise Applications - すべてのアプリケーション]](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps) に移動します。
2. **[表示]** ドロップダウンで **[すべてのアプリケーション]** を選択し、**[適用]** をクリックします。
3. 次の表を使用して、各アプリケーション ID を検索ボックスに貼り付け、Enter キーを押して検索します。 検索結果が空の場合は、"解決策" 列の手順に従ってそのサービス プリンシパルを再作成する必要があります。

| アプリケーション ID | 解決策 |
| :--- | :--- | :--- |
| 2565bd9d-da50-47d4-8b85-4c97f669dc36 | [不足しているサービス プリンシパルを PowerShell を使用して再作成する](#recreate-a-missing-service-principal-with-powershell) |
| 443155a6-77f3-45e3-882b-22b3a8d431fb | [Microsoft.AAD 名前空間に再登録する](#re-register-to-the-microsoft-aad-namespace-using-the-azure-portal) |
| abba844e-bc0e-44b0-947a-dc74e5d09022  | [Microsoft.AAD 名前空間に再登録する](#re-register-to-the-microsoft-aad-namespace-using-the-azure-portal) |
| d87dcbc6-a371-462e-88e3-28ad15ec4e64 | [自動修正を行うサービス プリンシパル](#service-principals-that-self-correct) |

## <a name="recreate-a-missing-service-principal-with-powershell"></a>不足しているサービス プリンシパルを PowerShell を使用して再作成する
Azure AD ディレクトリで ID ```2565bd9d-da50-47d4-8b85-4c97f669dc36``` のサービス プリンシパルが不足している場合は、次の手順に従います。

**解決策:** この手順を完了するには Azure AD PowerShell が必要です。 Azure AD PowerShell のインストールについては、[こちらの記事](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0.)を参照してください。

この問題に対処するには、PowerShell ウィンドウで次のコマンドを入力します。
1. Azure AD PowerShell モジュールをインストールし、インポートします。
    
    ```powershell 
    Install-Module AzureAD
    Import-Module AzureAD
    ```
    
2. 次の PowerShell コマンドを実行して、Azure AD Domain Services に必要なサービス プリンシパルがディレクトリに不足しているかどうかをチェックします。
    
    ```powershell
    Get-AzureAdServicePrincipal -filter "AppId eq '2565bd9d-da50-47d4-8b85-4c97f669dc36'"
    ```
    
3. 次の PowerShell コマンドを入力して、サービス プリンシパルを作成します。

    ```powershell
    New-AzureAdServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
    ```
    
4. 不足しているサービス プリンシパルを作成した後、2 時間待機してから管理対象ドメインの正常性をチェックします。


## <a name="re-register-to-the-microsoft-aad-namespace-using-the-azure-portal"></a>Azure Portal を使用して、Microsoft AAD 名前空間に再登録する
Azure AD ディレクトリで ID ```443155a6-77f3-45e3-882b-22b3a8d431fb``` または ```abba844e-bc0e-44b0-947a-dc74e5d09022``` のサービス プリンシパルが不足している場合は、次の手順に従います。

**解決策:** 次の手順を使用して、ディレクトリに Domain Services を復元します。

1. Azure Portal の [[サブスクリプション]](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) ページに移動します。
2. テーブルから管理対象ドメインに関連付けられているサブスクリプションを選択します。
3. 左側のナビゲーションから、**[リソース プロバイダー]** を選択します。
4. テーブルで [Microsoft.AAD] を見つけ、**[再登録]** をクリックします。
5. アラートが解決されたことを確認するには、2 時間後に管理対象ドメインの正常性ページを表示します。


## <a name="service-principals-that-self-correct"></a>自動修正を行うサービス プリンシパル
Azure AD ディレクトリで ID ```d87dcbc6-a371-462e-88e3-28ad15ec4e64``` のサービス プリンシパルが不足している場合は、次の手順に従います。

**解決策:** Azure AD Domain Services では、この特定のサービス プリンシパルが不足している、正しく構成されていない、または削除されていることを特定できます。 サービス プリンシパルは、自動的に再作成されます。 サービス プリンシパルが再作成されていることを確認するには、2 時間後に管理対象ドメインの正常性をチェックします。


## <a name="contact-us"></a>お問い合わせ
[フィードバックの共有およびサポートについては](active-directory-ds-contact-us.md)、Azure Active Directory Domain Services 製品チームにお問い合わせください。
