---
title: "Azure AD Connect: シームレス シングル サインオン - クイック スタート| Microsoft Docs"
description: "この記事では、Azure Active Directory シームレス シングル サインオンの開始方法について説明します"
services: active-directory
keywords: "Azure AD Connect とは, Active Directory のインストール, Azure AD に必要なコンポーネント, SSO, シングル サインオン"
documentationcenter: 
author: swkrish
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/23/2017
ms.author: billmath
ms.openlocfilehash: 58ca992f9fcf9a03d917f0dc250a292c4d5f49e5
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/24/2018
---
# <a name="azure-active-directory-seamless-single-sign-on-quick-start"></a>Azure Active Directory シームレス シングル サインオン: クイック スタート

## <a name="deploy-seamless-single-sign-on"></a>シームレス シングル サインオンのデプロイ

Azure Active Directory (Azure AD) シームレス シングル サインオン (シームレス SSO) により、ユーザーは企業ネットワークにつながっている会社のデスクトップを使用するときに自動でサインインできます。 シームレス SSO により、オンプレミス コンポーネントの追加を必要とすることなく、ユーザーはクラウド ベースのアプリケーションに簡単にアクセスできるようになります。

シームレス SSO をデプロイするには、以下の手順に従います。

## <a name="step-1-check-the-prerequisites"></a>手順 1: 前提条件を確認する

次の前提条件が満たされていることを確認します。

* **Azure AD Connect サーバーがセットアップされている**: サインイン方法として[パススルー認証](active-directory-aadconnect-pass-through-authentication.md)を使用する場合、他に確認すべき前提条件はありません。 サインイン方法として[パスワード ハッシュ同期](active-directory-aadconnectsync-implement-password-synchronization.md)を使用する場合や、Azure AD Connect と Azure AD の間にファイアウォールがある場合は、次の点を確認してください。
   - Azure AD Connect バージョン 1.1.644.0 以降を使用している。 
   - ファイアウォールまたはプロキシで DNS ホワイトリストを許可している場合は、**\*.msappproxy.net** の URL に対するポート 443 での接続をホワイトリストに登録します。 そうでない場合は、毎週更新される [Azure データセンターの IP 範囲](https://www.microsoft.com/download/details.aspx?id=41653)へのアクセスを許可します。 この前提条件は、その機能を有効にした場合にのみ適用されます。 実際のユーザー サインインに必要な条件ではありません。

    >[!NOTE]
    >Azure AD Connect のバージョン 1.1.557.0、1.1.558.0、1.1.561.0、1.1.614.0 には、パスワード ハッシュ同期に関連する問題があります。 パスワード ハッシュ同期をパススルー認証と組み合わせて使用_しない_場合の詳細については、[Azure AD Connect のリリース ノート](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-version-history#116470)をご覧ください。

* **ドメイン管理者の資格情報がセットアップされている**: 次の各 Active Directory フォレストについて、ドメイン管理者の資格情報が必要です。
    * Azure AD Connect を使用して Azure AD に同期している。
    * シームレス SSO を有効にさせたいユーザーが含まれている。

## <a name="step-2-enable-the-feature"></a>手順 2: 機能を有効にする

[Azure AD Connect](active-directory-aadconnect.md) を使用してシームレス SSO を有効にします。

新しく Azure AD Connect をインストールする場合は、[カスタム インストール パス](active-directory-aadconnect-get-started-custom.md)を選択します。 **[ユーザー サインイン]** ページで、**[シングル サインオンを有効にする]** チェックボックスをオンにします。

![Azure AD Connect: [ユーザー サインイン]](./media/active-directory-aadconnect-sso/sso8.png)

Azure AD Connect を既にインストールしている場合は、Azure AD Connect の **[ユーザー サインインの変更]** ページを選択して、**[次へ]** を選択します。

![Azure AD Connect: [ユーザー サインインの変更]](./media/active-directory-aadconnect-user-signin/changeusersignin.png)

**[シングル サインオンを有効にする]** ページが表示されるまで、ウィザードの手順を続行します。 次の各 Active Directory フォレストのドメイン管理者の資格情報を入力します。
    * Azure AD Connect を使用して Azure AD に同期している。
    * シームレス SSO を有効にさせたいユーザーが含まれている。

ウィザードの完了後、シームレスな SSO がテナントで有効になります。

>[!NOTE]
> ドメイン管理者の資格情報は Azure AD Connect にも Azure AD にも保存されません。 機能を有効にするためにのみ使用されます。

この手順に従って、シームレス SSO の有効化を正しく行ったことを確認します。

1. テナントのグローバル管理者の資格情報を使用して、[Azure Active Directory 管理センター](https://aad.portal.azure.com)にサインインします。
2. 左ウィンドウで、**[Azure Active Directory]** を選択します。
3. **[Azure AD Connect]** を選びます。
4. **[シームレスなシングル サインオン]** 機能が **[有効]** になっていることを確認します。

![Azure Portal: Azure AD Connect ウィンドウ](./media/active-directory-aadconnect-sso/sso10.png)

## <a name="step-3-roll-out-the-feature"></a>手順 3: 機能をロールアウトする

この機能をユーザーにロールアウトするには、Active Directory のグループ ポリシーを使用して、ユーザーのイントラネット ゾーンの設定に次の Azure AD URL を追加する必要があります。

- https://autologon.microsoftazuread-sso.com


また、グループ ポリシーを使用して、**[スクリプトを介したステータス バーの更新を許可する]** というイントラネットのゾーン ポリシー設定を有効にする必要があります。 

>[!NOTE]
> 以下の手順は、Windows 上の Internet Explorer と Google Chrome (信頼済みサイト URL のセットを Internet Explorer と共有する場合) でのみ機能します。 Mac 上の Mozilla Firefox および Google Chrome をセットアップする方法については、次のセクションをご覧ください。

### <a name="why-do-you-need-to-modify-users-intranet-zone-settings"></a>ユーザーのイントラネット ゾーンの設定を変更する必要がある理由

既定では、ブラウザーは指定 URL から適切なゾーン (インターネットまたはイントラネット) を自動的に判断します。 たとえば、"http://contoso/" はイントラネット ゾーンにマップされ、"http://intranet.contoso.com/" はインターネット ゾーンにマップされます (URL にピリオドが含まれているため)。 Azure AD URL と同様に、クラウド エンドポイントの URL をブラウザーのイントラネット ゾーンに明示的に追加しなければ、ブラウザーは Kerberos チケットをクラウド エンドポイントに送信しません。

### <a name="detailed-steps"></a>詳細な手順

1. グループ ポリシー管理エディター ツールを開きます。
2. 一部またはすべてのユーザーに適用されるグループ ポリシーを編集します。 この例では、**既定のドメイン ポリシー**を使用します。
3. **[ユーザーの構成]** > **[管理用テンプレート]** > **[Windows コンポーネント]** > **[Internet Explorer]** > **[インターネット コントロール パネル]** > **[セキュリティ ページ]** の順に移動します。 次に **[サイトとゾーンの割り当て一覧]** を選択します。
    ![シングル サインオン](./media/active-directory-aadconnect-sso/sso6.png)
4. ポリシーを有効にしてから、ダイアログ ボックスに次の値を入力します。
   - **[値の名前]**: Kerberos チケットの転送先となる Azure AD URL。
   - **[値]** (データ): **1** (イントラネット ゾーンを示す)

    結果は次のようになります。

    値: https://autologon.microsoftazuread-sso.com
  
    Data 1

   >[!NOTE]
   > 一部のユーザーに対してシームレス SSO の使用を許可したくない場合 ( ユーザー共有キオスクでサインインする場合など) は、前述の値を **4** に設定します。 この操作により、Azure AD の URL が [制限付きゾーン] に追加され、シームレス SSO は常に失敗するようになります。
   >

5. **[OK]** を選択してから、もう一度 **[OK]** を選択します。

    ![シングル サインオン](./media/active-directory-aadconnect-sso/sso7.png)

6. **[ユーザーの構成]** > **[管理用テンプレート]** > **[Windows コンポーネント]** > **[Internet Explorer]** > **[インターネット コントロール パネル]** > **[セキュリティ ページ]** > **[イントラネット ゾーン]** の順に移動します。 次に、**[スクリプトを介したステータス バーの更新を許可する]** を選択します。

    ![シングル サインオン](./media/active-directory-aadconnect-sso/sso11.png)

7. ポリシー設定を有効にしてから、**[OK]** を選択します。

    ![シングル サインオン](./media/active-directory-aadconnect-sso/sso12.png)

### <a name="browser-considerations"></a>ブラウザーの考慮事項

#### <a name="mozilla-firefox-all-platforms"></a>Mozilla Firefox (すべてのプラットフォーム)

Mozilla Firefox は、Kerberos 認証を自動的には使用しません。 各ユーザーが、次の手順に従って、Firefox の設定に Azure AD の URL を手動で追加する必要があります。
1. Firefox を実行し、アドレス バーに「`about:config`」と入力します。 表示されているすべての通知を無視します。
2. **network.negotiate-auth.trusted-uris** の設定を検索します。 この設定は、Kerberos 認証用の Firefox の信頼済みサイトを一覧表示します。
3. 右クリックして **[変更]** を選択します。
4. フィールドに「https://autologon.microsoftazuread-sso.com」と入力します。
5. **[OK]** を選択してから、ブラウザーをもう一度開きます。

#### <a name="safari-mac-os"></a>Safari (Mac OS)

Mac OS を実行しているコンピューターが Azure AD に参加していることを確認します。 Azure AD への参加の詳細については、『[Best Practices for Integrating OS X with Active Directory (OS X と Active Directory の統合に関するベスト プラクティス)](http://training.apple.com/pdf/Best_Practices_for_Integrating_OS_X_with_Active_Directory.pdf)』をご覧ください。

#### <a name="google-chrome-all-platforms"></a>Google Chrome (すべてのプラットフォーム)

お使いの環境の [AuthNegotiateDelegateWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthNegotiateDelegateWhitelist) ポリシー設定または [AuthServerWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthServerWhitelist) ポリシー設定をオーバーライドした場合は、Azure AD の URL (https://autologon.microsoftazuread-sso.com) を必ず追加してください。

#### <a name="google-chrome-mac-os-only"></a>Google Chrome (Mac OS のみ)

Mac OS などの Windows 以外のプラットフォームで Google Chrome を使用し、統合認証で Azure AD の URL をホワイトリスト化する方法については、[Chromium プロジェクト ポリシー リスト](https://dev.chromium.org/administrators/policy-list-3#AuthServerWhitelist)に関する記事をご覧ください。

サード パーティの Active Directory グループ ポリシーの拡張機能を使用して、Mac 上で Firefox および Google Chrome を使用するユーザーに Azure AD の URL を ロールアウトする場合については、この記事では扱われません。

#### <a name="known-browser-limitations"></a>ブラウザーの既知の制限事項

シームレス SSO は、Firefox および Edge ブラウザーのプライベート ブラウズ モードでは動作しません。 拡張保護モードで実行されている場合は、Internet Explorer ブラウザーでも機能しません。

## <a name="step-4-test-the-feature"></a>手順 4: 機能をテストする

特定のユーザーについてこの機能をテストするには、次の条件がすべて満たされていることを確認してください。
  - ユーザーが会社のデバイスでサインインしている。
  - デバイスが Active Directory ドメインに参加している。
  - デバイスが、企業のワイヤードまたはワイヤレス ネットワーク上や、VPN 接続などのリモート アクセス接続を介してドメイン コントローラー (DC) に直接接続している。
  - グループ ポリシーを使用して、このユーザーに[機能がロールアウト](##step-3-roll-out-the-feature)されている。

ユーザーがユーザー名のみを入力し、パスワードを入力しないシナリオをテストする場合:
   - 新しいプライベート ブラウザー セッションで https://myapps.microsoft.com/ にサインインします。

ユーザーがユーザー名またはパスワードを入力する必要がないシナリオをテストする場合は、次の手順のいずれかに従います。 
   - 新しいプライベート ブラウザー セッションで https://myapps.microsoft.com/contoso.onmicrosoft.com にサインインします。 *contoso* を自分のテナント名に置き換えます。
   - 新しいプライベート ブラウザー セッションで https://myapps.microsoft.com/contoso.com にサインインします。 *contoso.com* を、自分のテナントで検証されたドメイン (フェデレーション ドメインではなく) に置き換えます。

## <a name="step-5-roll-over-keys"></a>手順 5: キーをロール オーバーする

手順 2 では、Azure AD Connect によって、シームレス SSO を有効にしたすべての Active Directory フォレスト内でコンピューター アカウント (Azure AD を表します) が作成されます。 詳細については、「[Azure Active Directory シームレス シングル サインオン: 技術的な詳細](active-directory-aadconnect-sso-how-it-works.md)」をご覧ください。 セキュリティを強化するために、これらのコンピューター アカウントの Kerberos 暗号化の解除キーを定期的にロールオーバーすることをお勧めします。 キーのロールオーバー方法の詳細については、「[Azure Active Directory シームレス シングル サインオン: よく寄せられる質問](active-directory-aadconnect-sso-faq.md#how-can-i-roll-over-the-kerberos-decryption-key-of-the-azureadssoacc-computer-account)」をご覧ください。

>[!IMPORTANT]
>この機能を有効にした後に、"_直ちに_" この手順を実行する必要はありません。 少なくとも 30 日に 1 回は、Kerberos 暗号化の解除キーをロールオーバーしてください。

## <a name="next-steps"></a>次のステップ

- [技術的な詳細](active-directory-aadconnect-sso-how-it-works.md): シームレス シングル サインオン機能のしくみを理解します。
- [よく寄せられる質問](active-directory-aadconnect-sso-faq.md): シームレス シングル サインオンに関してよく寄せられる質問への回答を示します。
- [トラブルシューティング](active-directory-aadconnect-troubleshoot-sso.md): シームレス シングル サインオン機能に関する一般的な問題の解決方法を説明します。
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect): Azure Active Directory フォーラムで、新しい機能の要望を出します。
