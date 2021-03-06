<properties
	pageTitle="アプリケーション プロキシのトラブルシューティング | Microsoft Azure"
	description="Azure AD アプリケーション プロキシのエラーのトラブルシューティングを行う方法について説明します。"
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/22/2016"
	ms.author="kgremban"/>



# アプリケーション プロキシのトラブルシューティング

発行されたアプリケーションへのアクセス中、またはアプリケーションの発行中にエラーが発生する場合は、Microsoft Azure AD アプリケーション プロキシが正しく機能しているかどうかを次のオプションで確認します。

- Windows サービス コンソールを開き、**Microsoft AAD アプリケーション プロキシ コネクタ** サービスが有効で実行されていることを確認します。次の図のように、アプリケーション プロキシ サービスのプロパティ ページで確認することもできます。![Microsoft AAD アプリケーション プロキシ コネクタのプロパティのスクリーンショット](./media/active-directory-application-proxy-troubleshoot/connectorproperties.png)

- イベント ビューアーを開き、**[アプリケーションとサービス ログ]**、**[Microsoft]**、**[AadApplicationProxy]**、**[コネクタ]** の順にクリックして、**[Admin]** の下にあるアプリケーション プロキシ コネクタに関連付けられているイベントを探します。
- 必要に応じて、分析およびデバッグ ログを有効にし、サービス アプリケーション プロキシ コネクタのセッション ログを有効にすることで、より詳細なログを使用できます。


## 一般エラー

エラー | 説明 | 解決策
--- | --- | ---
この企業のアプリにはアクセスできません。このアプリケーションにアクセスする権限がありません。承認に失敗しました。このアプリケーションへのアクセス権をユーザーに割り当ててください。 | このアプリケーションにユーザーを割り当てていない可能性があります。 | **[アプリケーション]** タブに移動し、**[ユーザーとグループ]** でこのユーザーまたはユーザー グループをこのアプリケーションに割り当ててください。
この企業のアプリにはアクセスできません。このアプリケーションにアクセスする権限がありません。承認に失敗しました。ユーザーが Azure Active Directory Premium または Basic のライセンスを持っていることを確認します。 | アプリケーションにアクセスしようとしているユーザーが、サブスクライバーの管理者によって Premium/Basic ライセンスを明示的に割り当てられていない場合、発行されたアプリにユーザーがアクセスしようとすると、このエラーが発生することがあります。 | サブスクライバーの Active Directory **[ライセンス]** タブに移動し、このユーザーまたはユーザー グループに Premium または Basic ライセンスが割り当てられていることを確認します。


## コネクタのトラブルシューティング
コネクタ ウィザードのインストール中に登録が失敗した場合、エラーの原因を調べるには、**[Windows ログ]** の **[アプリケーション]** でイベント ログを見るか、次の Windows PowerShell コマンドを実行します。

    Get-EventLog application –source “Microsoft AAD Application Proxy Connector” –EntryType “Error” –Newest 1

| エラー | 説明 | 解決策 |
| --- | --- | --- |
| コネクタ登録に失敗しました: Microsoft Azure 管理ポータルでアプリケーション プロキシを有効化したことと、Active Directory ユーザー名とパスワードを正しく入力したことを確認してください。エラー: '1 つ以上のエラーが発生しました。' | Azure AD へのログインをせずに登録ウィンドウを閉じました。 | コネクタ ウィザードを再実行して、コネクタを登録します。 |
| コネクタ登録に失敗しました: Microsoft Azure 管理ポータルでアプリケーション プロキシを有効化したことと、Active Directory ユーザー名とパスワードを正しく入力したことを確認してください。エラー: 'AADSTS50001: リソース `https://proxy.cloudwebappproxy.net/registerapp` が無効です。' | アプリケーション プロキシが無効です。 | コネクタの登録を試行する前に、Azure クラシック ポータルでアプリケーション プロキシを有効にしてください。アプリケーション プロキシを有効にする方法の詳細については、「[アプリケーション プロキシ サービスを有効にする](active-directory-application-proxy-enable.md)」を参照してください。 |
| コネクタ登録に失敗しました: Microsoft Azure 管理ポータルでアプリケーション プロキシを有効化したことと、Active Directory ユーザー名とパスワードを正しく入力したことを確認してください。エラー: '1 つ以上のエラーが発生しました。' | 登録ウィンドウが開き、ログインを許さずにすぐに閉じてしまった場合は、通常、このエラーが発生します。このエラーは、ある種のネットワーク エラーがシステムで起きた場合に発生します。 | ブラウザーから一般 Web サイトに接続できることと、「[アプリケーション プロキシの前提条件](active-directory-application-proxy-enable.md)」で指定されているようにポートが開かれていることを確認してください。 |
| コネクタの登録に失敗しました: コンピューターがインターネットに接続されていることを確認してください。エラー: 'メッセージを受信できる `https://connector.msappproxy.net:9090/register/RegisterConnector` でリッスンしているエンドポイントがありませんでした。これは一般に、アドレスまたは SOAP アクションが正しくない場合に発生します。詳細については、InnerException を参照してください (ある場合)。' | Azure AD のユーザー名とパスワードを使用してログインしても、このエラーが発生する場合は、8081 より上のすべてのポートがブロックされている可能性があります。 | 必要なポートが開かれていることを確認します。詳細については、「[アプリケーション プロキシの前提条件](active-directory-application-proxy-enable.md)」を参照してください。 |
| 登録ウィンドウに明確なエラーが表示されています。続行できません。ウィンドウを閉じるしかありません。 | 間違ったユーザー名またはパスワードを入力しました。 | やり直してください。 |
| コネクタ登録に失敗しました: Microsoft Azure 管理ポータルでアプリケーション プロキシを有効化したことと、Active Directory ユーザー名とパスワードを正しく入力したことを確認してください。エラー: 'AADSTS50059: テナントを識別する情報が要求内に見つからず、指定されたどの資格情報でも暗黙的に示されなかったため、サービス プリンシパル URI による検索が失敗しました。 | Microsoft アカウントと、アクセスしようとしているディレクトリの組織 ID の一部であるドメイン以外を使用して、ログインしようとしています。 | admin がテナント ドメインと同じドメイン名の一部であることを確認します。たとえば、Azure AD ドメインが contoso.com である場合、admin は admin@contoso.com である必要があります。 |
| PowerShell スクリプトを実行するための現在の実行ポリシーを取得できませんでした。 | コネクタのインストールに失敗した場合は、PowerShell 実行ポリシーが無効になっていないことを確認します。 | グループ ポリシー エディターを開きます。**[コンピューターの構成]**、**[管理用テンプレート]**、**[Windows コンポーネント]**、**[Windows PowerShell]** の順に移動して、**[スクリプトの実行を有効にする]** をダブルクリックします。これは、**[未構成]** または **[有効]** に設定できます。**[有効]** に設定した場合は、[オプション] で実行ポリシーが **[ローカル スクリプトおよびリモートの署名済みスクリプトを許可する]** または **[すべてのスクリプトを許可する]** に設定されていることを確認します。 |
| コネクタが構成のダウンロードに失敗しました。 | 認証に使用される、コネクタのクライアント証明書が期限切れです。このエラーは、コネクタをプロキシの背後にインストールした場合にも発生することがあります。その場合、コネクタはインターネットにアクセスできず、リモート ユーザーにアプリケーションを提供できません。 | Windows PowerShell で `Register-AppProxyConnector` コマンドレットを使用して、手動で信頼を更新します。コネクタがプロキシの背後にある場合は、コネクタ アカウントの "ネットワーク サービス" および "ローカル システム" にインターネットへのアクセスを許可する必要があります。そうするには、プロキシへのアクセスを許可するか、プロキシをバイパスするように設定します。 |
| コネクタの登録に失敗しました: コネクタを登録する Active Directory のグローバル管理者であることを確認してください。エラー: '登録要求が拒否されました。' | ログインに使用しようとしているエイリアスは、このドメインの管理者ではありません。コネクタは必ず、ユーザーのドメインを所有しているディレクトリ用にインストールされます。 | ログインしようとしている管理者が Azure AD テナントに対するグローバル アクセス許可を持っていることを確認してください。|


## Kerberos のエラー

| エラー | 説明 | 解決策 |
| --- | --- | --- |
| PowerShell スクリプトを実行するための現在の実行ポリシーを取得できませんでした。 | コネクタのインストールに失敗した場合は、PowerShell 実行ポリシーが無効になっていないことを確認します。 | グループ ポリシー エディターを開きます。**[コンピューターの構成]**、**[管理用テンプレート]**、**[Windows コンポーネント]**、**[Windows PowerShell]** の順に移動して、**[スクリプトの実行を有効にする]** をダブルクリックします。これは、**[未構成]** または **[有効]** に設定できます。**[有効]** に設定した場合は、[オプション] で実行ポリシーが **[ローカル スクリプトおよびリモートの署名済みスクリプトを許可する]** または **[すべてのスクリプトを許可する]** に設定されていることを確認します。 |
| 12008 - Azure AD が、バックエンド サーバーに対する Kerberos 認証の試行で、許可されている最大数を超えました。 | このイベントは、Azure AD とバックエンド アプリケーション サーバー間の構成が正しくないことや、両方のコンピューターに日付と時刻の構成の問題があることを示している場合があります。 | バックエンド サーバーが、Azure AD によって作成された Kerberos チケットを拒否しました。Azure AD とバックエンド アプリケーション サーバーが正しく構成されていることを確認してください。Azure AD とバックエンド アプリケーション サーバーの日付と時刻の構成が同期されていることを確認してください。 |
| 13016 - Azure AD が、エッジ トークンまたはアクセス Cookie に UPN がないため、ユーザーに代わって Kerberos チケットを取得できません。 | STS 構成の問題があります。 | STS の UPN 要求の構成を修正してください。 |
| 13019 - Azure AD が、次の一般 API エラーのため、ユーザーに代わって Kerberos チケットを取得できません。 | このイベントは、Azure AD とドメイン コントローラー サーバー間の構成が正しくないことや、両方のコンピューターに日付と時刻の構成の問題があることを示している場合があります。 | ドメイン コントローラーが、Azure AD によって作成された Kerberos チケットを拒否しました。Azure AD とバックエンド アプリケーション サーバーが正しく構成されていることを確認してください (特に SPN 構成)。ドメイン コントローラーが Azure AD との信頼関係を確立できるように、Azure AD がドメイン コントローラーと同じドメインに参加しているドメインであることを確認してください。Azure AD とドメイン コントローラーの日付と時刻の構成が同期されていることを確認してください。 |
| 13020 - バックエンド サーバー SPN が定義されていないため、Azure AD がユーザーに代わって Kerberos チケットを取得できません。 | このイベントは、Azure AD とドメイン コントローラー サーバー間の構成が正しくないことや、両方のコンピューターに日付と時刻の構成の問題があることを示している場合があります。 | ドメイン コントローラーが、Azure AD によって作成された Kerberos チケットを拒否しました。Azure AD とバックエンド アプリケーション サーバーが正しく構成されていることを確認してください (特に SPN 構成)。ドメイン コントローラーが Azure AD との信頼関係を確立できるように、Azure AD がドメイン コントローラーと同じドメインに参加しているドメインであることを確認してください。Azure AD とドメイン コントローラーの日付と時刻の構成が同期されていることを確認してください。 |
| 13022 - バックエンド サーバーが Kerberos 認証の試行に対して HTTP 401 エラーを返すため、Azure AD がユーザーを認証できません。 | このイベントは、Azure AD とバックエンド アプリケーション サーバー間の構成が正しくないことや、両方のコンピューターに日付と時刻の構成の問題があることを示している場合があります。 | バックエンド サーバーが、Azure AD によって作成された Kerberos チケットを拒否しました。Azure AD とバックエンド アプリケーション サーバーが正しく構成されていることを確認してください。Azure AD とバックエンド アプリケーション サーバーの日付と時刻の構成が同期されていることを確認してください。 |
| Web サイトがページを表示できません。 | アプリケーションが IWA アプリケーションである場合、発行したアプリにユーザーがアクセスしようとしたときに、このエラーが表示されることがあります。このアプリケーションのために定義された SPN が正しくない可能性があります。 | IWA アプリの場合: このアプリケーションのために構成された SPN が正しいことを確認してください。 |
| Web サイトがページを表示できません。 | アプリケーションが OWA アプリケーションである場合、発行したアプリにユーザーがアクセスしようとしたときに、このエラーが表示されることがあります。これは次のいずれかが原因で発生することがあります。<br> - このアプリケーションのために定義された SPN が正しくない。<br> - アプリケーションにアクセスしようとしたユーザーが、サインインするための適切な企業アカウントではなく Microsoft アカウントを使用しているか、ユーザーがゲスト ユーザーである。<br> - アプリケーションにアクセスしようとしたユーザーが、オンプレミス側でこのアプリケーション用に適切に定義されていない。 | それぞれの対処手順: <br> - このアプリケーションのために構成された SPN が正しいことを確認します。<br> - ユーザーが発行済みのアプリケーションのドメインと一致する企業アカウントを使用して、サインインしていることを確認します。Microsoft アカウントのユーザーとゲスト ユーザーは IWA アプリケーションにはアクセスできません。<br> - このユーザーが、オンプレミス コンピューターでこのバックエンド アプリケーションに対して定義されている適切なアクセス許可を持っていることを確認します。 |
| この企業のアプリにはアクセスできません。このアプリケーションにアクセスする権限がありません。承認に失敗しました。このアプリケーションへのアクセス権をユーザーに割り当ててください。 | アプリケーションにアクセスしようとしたユーザーが、サインインするための適切な企業アカウントではなく Microsoft アカウントを使用しているか、ユーザーがゲスト ユーザーである場合、発行したアプリにユーザーがアクセスしようとしたときに、このエラーが表示されることがあります。 | Microsoft アカウントのユーザーとゲスト ユーザーは IWA アプリケーションにはアクセスできません。ユーザーが、発行されたアプリケーションのドメインに一致する企業アカウントを使用してサインインするようにします。 |
| この企業アプリには、現在、アクセスできません。後でもう一度やり直してください。コネクタがタイムアウトになりました。 | アプリケーションにアクセスしようとしたユーザーが、オンプレミス側でこのアプリケーション用に適切に定義されていない場合、発行したアプリにユーザーがアクセスしようとしたときに、このエラーが表示されることがあります。 | このユーザーが、オンプレミス コンピューターでこのバックエンド アプリケーションに対して定義されているような適切なアクセス許可を持っていることを確認します。 |


## 関連項目

- [Azure Active Directory のアプリケーション プロキシを有効にする](active-directory-application-proxy-enable.md)
- [アプリケーション プロキシを使用してアプリケーションを発行する](active-directory-application-proxy-publish.md)
- [シングル サインオンの有効化](active-directory-application-proxy-sso-using-kcd.md)
- [条件付きアクセスを有効にする](active-directory-application-proxy-conditional-access.md)

最新のニュースと更新情報については、[アプリケーション プロキシに関するブログ](http://blogs.technet.com/b/applicationproxyblog/)をご覧ください。


<!--Image references-->
[1]: ./media/active-directory-application-proxy-troubleshoot/connectorproperties.png
[2]: ./media/active-directory-application-proxy-troubleshoot/sessionlog.png

<!---HONumber=AcomDC_0622_2016-->