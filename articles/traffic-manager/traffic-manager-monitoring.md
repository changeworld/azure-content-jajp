<properties
   pageTitle="Traffic Manager エンドポイントの監視とフェールオーバー |Microsoft Azure"
   description="この記事では、Azure ユーザーが高可用性アプリケーションをデプロイできるように、Traffic Manager でエンドポイントの監視と自動フェールオーバーの機能がどのように使用されているかを説明します。"
   services="traffic-manager"
   documentationCenter=""
   authors="jtuliani"
   manager="carmonm"
   editor="tysonn" />
<tags
   ms.service="traffic-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="07/01/2016"
   ms.author="jtuliani" />

# Traffic Manager エンドポイントの監視とフェールオーバー

Azure Traffic Manager には、エンドポイントの監視と自動フェールオーバーの機能が組み込まれています。この機能を使用して、Azure リージョンの障害を含むエンドポイント障害に対する回復性を備えた高可用性アプリケーションを提供できます。

この機能は、各エンドポイントに通常の要求を行い、その応答を検証することで実現されています。エンドポイントが有効な応答を提供できない場合は、低下としてその状態を表示します。DNS 応答に含まれなくなり、代わりに使用可能な別のエンドポイントが返されます。これにより、ユーザーのトラフィックは、障害が発生しているエンドポイントを避けて、使用可能なエンドポイントに転送されます。

低下状態のエンドポイントに対してはエンドポイント正常性チェックが継続的に実行されるため、正常な状態に戻ったときには自動的にローテーションに復帰します。

## エンドポイント監視の構成

エンドポイント監視を構成するには、Traffic Manager プロファイルで次の設定を指定する必要があります。

- **プロトコル**HTTP または HTTPS を選択します。HTTPS 監視は、その証明書の存在のみをチェックして、SSL 証明書が有効であるかどうかを検証しないことに注意することが重要です。
- **ポート**要求に使用するポートを選択します。選択肢には、標準の HTTP ポートや HTTPS ポートがあります。
- **パス**監視機能の正常性チェックがアクセスを試行する相対パスと Web ページまたはファイルの名前を指定します。スラッシュ (/) は、相対パスの有効なエントリであり、ファイルがルート ディレクトリ (既定値) にあることを示します。

Traffic Manager は、各エンドポイントの正常性をチェックするために、指定されたプロトコル、ポート、および相対パスを使用して、エンドポイントに GET 要求を行います。

一般的な方法として、アプリケーション内に /health.aspx などのカスタム ページを実装します。Traffic Manager エンドポイント監視パスとしてこのページを構成します。ページ内で、データベースの可用性チェックなど、アプリケーションに応じて必要なチェックを実行してから、HTTP 200 (サービスが正常な場合) または他の状態コード (正常でない場合) を返します。

エンドポイント監視の設定は、エンドポイント単位ではなく、Traffic Manager プロファイル単位で構成されます。そのため、すべてのエンドポイントの正常性チェックに同じ設定が使用されます。複数のエンドポイントに対して異なる監視設定を使用する場合は、[入れ子になった Traffic Manager プロファイル](traffic-manager-nested-profiles.md#example-5-per-endpoint-monitoring-settings)を使用します。

## エンドポイントとプロファイルの状態

Traffic Manager のプロファイルとエンドポイントは、ユーザーが有効と無効を切り替えることができます。ただし、Traffic Manager の設定とプロセスを自動化した結果、エンドポイントの状態が変更される場合もあります。以降では、各パラメーターの挙動について詳しく説明します。

### エンドポイントの状態

エンドポイントの状態はユーザーが制御する設定で、個々のエンドポイントを簡単に有効または無効にすることができます。この設定は、基になるサービスの状態には影響を与えません (実行中状態が継続します)。Traffic Manager が対象エンドポイントを使用できるかどうかを制御します。エンドポイントの状態を無効にすると、正常性はチェックされず、DNS 応答に含まれなくなります。

### プロファイルの状態

プロファイルの状態は、ユーザーが制御する設定です。簡単にプロファイルを有効または無効にすることができます。エンドポイントの状態が 1 つのエンドポイントに影響を与えるのに対して、プロファイルの状態はすべてのエンドポイントを含むプロファイル全体に作用します。無効にすると、プロファイルでエンドポイントは正常性がチェックされず、DNS 応答にどのエンドポイントも含まれなくなります (代わりに、NXDOMAIN 応答が返されます)。

### エンドポイント監視の状態

エンドポイント監視の状態は Traffic Manager で生成される設定で、エンドポイントの現在の状態を示します。手動でこの設定を変更することはできません。実行中のエンドポイント監視の情報のほか、エンドポイントの状態など他の情報も反映されています。エンドポイント監視の状態がとりうる値を、次の表に示します。(入れ子になったエンドポイントのエンドポイント監視の状態を計算する方法の詳細については、「[入れ子になった Traffic Manager プロファイル](traffic-manager-nested-profiles.md)」を参照してください。)

|プロファイルの状態|エンドポイントの状態|エンドポイント監視の状態 (API およびポータル)|メモ|
|---|---|---|---|
|無効|有効|非アクティブ|プロファイルは、ユーザーによって無効にされています。エンドポイントの状態を有効にすることはできますが、プロファイルの状態 (無効) が優先されます。無効なプロファイル内のエンドポイントは監視されませんし、DNS 応答 (NXDOMAIN 応答が返されます) にエンドポイントが含まれていません。|
|&lt;いずれも&gt;|無効|無効|エンドポイントは、ユーザーによって無効にされています。無効状態のエンドポイントは監視されません。DNS 応答に含めることができないため、トラフィックを受信しません。|
|有効|有効|オンライン|エンドポイントは監視されており、正常です。DNS 応答に含めることができるため、トラフィックを受信できます。|
|有効|有効|低下しています|エンドポイント監視の正常性チェックが失敗しています。DNS 応答に含めることができないため、トラフィックを受信しません。|
|有効|有効|エンドポイン トチェック中|エンドポイントを監視していますが、最初のプローブの結果をまだ受信していません。新しいエンドポイントをプロファイルに追加するか、エンドポイントまたはプロファイルを有効にした直後は、一時的にこの状態になります。この状態のエンドポイントはDNS 応答に含めることができるため、トラフィックを受信できます。|
|有効|有効|停止済み|このエンドポイントが参照するクラウド サービスまたは Web アプリが実行されていません。クラウド サービスまたは Web アプリの設定を確認してください。停止状態のエンドポイントは監視されません。DNS 応答に含めることができないため、トラフィックを受信しません。|

### プロファイル モニターの状態

プロファイル モニターの状態は、プロファイルに含まれるすべてのエンドポイントのエンドポイント モニター状態の値と、構成済みプロファイル状態を組み合わせたものです。とりうる値を、次の表に示します。

|(構成された)プロファイルの状態|エンドポイント監視の状態|プロファイル監視の状態 (API およびポータル)|メモ|
|---|---|---|---|
|無効|&lt;いずれも&gt;または、プロファイルでエンドポイントが定義されていません。|無効|プロファイルは、ユーザーによって無効にされています。|
|有効|少なくとも 1 つのエンドポイントの状態が低下です。|低下しています|これは顧客のアクションが必要であることを示しています。個々のエンドポイントの状態の値を確認し、さらなる注意が必要なエンドポイントを特定します。|
|有効|少なくとも 1 つのエンドポイントの状態がオンラインです。低下の状態になっているエンドポイントがありません。|オンライン|サービスはトラフィックを受け入れており、顧客の操作は必要ありません|
|有効|少なくとも 1 つのエンドポイントの状態が、エンドポイント チェック中です。オンラインまたは低下のエンドポイントはありません。|エンドポイント チェック中|切り替え状態これは通常、プロファイルを作成または有効にしたばかりで、エンドポイントの正常性が初めてチェックされている場合に発生します。|
|有効|プロファイルで定義されているすべてのエンドポイントの状態が無効または停止のいずれかであるか、プロファイルにエンドポイントが定義されていません。|非アクティブ|アクティブなエンドポイントはありませんが、プロファイルは引き続き有効です。|

## エンドポイントのフェールオーバーとフェールバック

以前は正常だったエンドポイントに障害が発生したシナリオを考えてみましょう。この障害は Traffic Manager によって検出され、エンドポイントはローテーションから外されます。その後、エンドポイントが回復します。それが Traffic Manager によって検出され、エンドポイントはローテーションに戻されます。

>[AZURE.NOTE] 応答メッセージが 200 OK の場合にのみ、Traffic Manager は、エンドポイントがオンラインであると見なします。次のいずれかが発生した場合、そのチェックは失敗とみなされます。

>- 200 以外の応答を受信した (別の 2 xx コードまたは 301/302 リダイレクトを含む)。
>- クライアント認証の要求
>- タイムアウト (タイムアウトのしきい値は 10 秒)
>- 接続できない

>失敗したチェックのトラブルシューティングについての詳細については、「[ Azure Traffic Managerでの機能低下状態のトラブルシューティング](traffic-manager-troubleshooting-degraded.md)」を参照してください。

次のタイムラインで、一連の手順を発生順に詳しく説明します。

![Traffic Manager endpoint failover and failback sequence](./media/traffic-manager-monitoring/timeline.png)

1. **GET**監視設定で指定したパスとファイルで、Traffic Manager の監視システムにより GET が実行されます。これは、エンドポイントごとに繰り返されます。
2. **200 OK**監視システムでは、HTTP 200 OK メッセージが 10 秒以内に返されると想定しています。この応答を受信すると、サービスが使用可能であると認識されます。
3. **30 秒間隔のチェック**このエンドポイント正常性チェックは 30 秒ごとに繰り返されます。
4. **サービスが使用不可**サービスが使用できなくなります。Traffic Manager は、次の正常性チェックまで使用可能になったことを認識しません。
5. **監視対象ファイルへのアクセス試行 (4 回の試行)**監視システムは GET を実行していますが、タイムアウト期間である 10 秒以内に応答が返されていません (または、200 以外の応答が受信されている場合があります)。次に、監視システムは、30 秒間隔で、さらに 3 回試行します。3 回の試行のいずれかが成功した場合、試行の回数はリセットされます。
6. **機能低下として指定**4 回連続して失敗した後に、監視システムは、利用できないエンドポイントの状態を低下として指定します。
7. **他のエンドポイントへのトラフィックの転送**Traffic Manager DNS ネーム サーバーが更新され、Traffic Manager による DNS クエリへの応答でこのエンドポイントが返されなくなります。そのため、新しい接続は他の使用可能なエンドポイントに転送されます。ただし、以前の DNS 応答にこのエンドポイントが含まれていて、その応答が再帰 DNS サーバーや DNS クライアントにキャッシュされている場合があります。これにより、クライアントがこのエンドポイントに接続しようとする場合があります。これらのキャッシュが期限切れになり、クライアントが新たに DNS クエリを実行すると、他のエンドポイントに転送されます。キャッシュ期間は、Traffic Manager プロファイルの TTL 設定によって制御され、30 秒などに設定されています。
8. **正常性チェックの続行**Traffic Manager は、低下状態にある間も、エンドポイントの正常性を継続的にチェックします。これは、エンドポイントが正常状態に戻ったときに検出できるようにするためです。
9. **サービスがオンラインに復帰**サービスが使用できるようになります。監視システムが次の正常性チェックを実行するまで、Traffic Manager ではエンドポイントは低下状態のままです。
10. **サービスへのトラフィックの再開**Traffic Manager は、GET 要求を送信し、200 OK ステータス応答を受け取ります。これは、サービスが正常な状態に戻ったことを示します。Traffic Manager のネーム サーバーがもう一度更新され、DNS 応答でサービスの DNS 名の配信が開始されます。他のエンドポイントに返されているキャッシュ済みの DNS 応答が期限切れになり、他のエンドポイントとの既存の接続が終了すると、トラフィックは元のエンドポイントに戻ります。

>[AZURE.NOTE] Traffic Manager は DNS レベルで動作するため、いずれかのエンドポイントとの既存の接続に影響を与えることはありません。(プロファイル設定を変更するか、フェールオーバーまたはフェールバックが発生して) エンドポイント間でトラフィックが送信される際、新しい接続は Traffic Manager によって使用可能なエンドポイントに転送されます。ただし、他のエンドポイントは、セッションが終了するまで既存の接続を介してトラフィックを受信し続ける可能性があります。トラフィックが既存の接続に転送されないようにするには、各エンドポイントで使用されるセッション期間をアプリケーションで制限する必要があります。

## 低下状態のエンドポイント

機能低下状態になったエンドポイントは、DNS クエリへの応答で返されなくなります (この規則の例外については、後述の「メモ」を参照してください)。代わりに別のエンドポイントが選択されて返されます。どのエンドポイントが選択されるかは、構成済みのトラフィック ルーティング方法によって異なります。

- **優先度**エンドポイントの優先度リストが使用されます。常に、リスト内で最初に使用可能なエンドポイントが返されます。エンドポイントが低下状態になると、次に使用可能なエンドポイントが返されます。
- **重み付け**使用可能なエンドポイントのいずれもが返される可能性があります。選択は、割り当てられた重みと、他の使用可能なエンドポイントの重みに基づき、ランダムに行われます。エンドポイントが低下状態になると、使用可能なエンドポイントの残りのプールからエンドポイントが選択されます。
- **パフォーマンス**エンドユーザーに最も近いエンドポイントが返されます (無効状態/停止状態のエンドポイントを除く)。エンドポイントが使用不可になると、他の使用可能なすべてのエンドポイントからエンドポイントがランダムに選択されます (この方法によって、次に近いエンドポイントが過負荷になったときに発生するおそれのある連鎖エラーを回避できます。パフォーマンス トラフィック ルーティング方法では、[入れ子になった Traffic Manager プロファイル](traffic-manager-nested-profiles.md#example-4-controlling-performance-traffic-routing-between-multiple-endpoints-in-the-same-region)を使用してフェールオーバー計画を構成することもできます)。

詳細については、「[Traffic Manager のトラフィック ルーティング方法](traffic-manager-routing-methods.md)」を参照してください。

>[AZURE.NOTE] すべての Traffic Manager エンドポイント (無効状態または停止状態のエンドポイントを除く) が正常性チェックに合格せず、低下として指定された場合はどうなるのでしょうか。

>この現象は通常、サービスの構成におけるエラー (例: Traffic Manager の正常性チェックが ACL でブロックされている) や Traffic Manager プロファイルの構成におけるエラー (例: 監視パスが正しくない) が原因で発生します。

>この場合、Traffic Manager は、"ベスト エフォート" として、*低下状態のすべてのエンドポイントが実際にはオンラインであるかのように応答*します。この方法は、DNS 応答でどのエンドポイントも返されないよりは望ましい手段です。

>どのエンドポイントも返されない場合、Traffic Manager の正常性チェックが正しく構成されていなくても、トラフィック ルーティングの観点では Traffic Manager が正常に動作*している*ように見える場合があります。ただし、エンドポイントに障害が発生した場合には、エンドポイントのフェールオーバーが行われず、アプリケーション全体の可用性に影響を与えます。この問題が発生しないようにするために、プロファイルに低下状態ではなくオンライン状態を表示することが重要になります。オンライン状態は、Traffic Manager の正常性チェックが正常に動作していることを示しています。

失敗した正常性チェックのトラブルシューティング方法の詳細については、「[Azure Traffic Manager での機能低下状態のトラブルシューティング](traffic-manager-troubleshooting-degraded.md)」を参照してください。

## FAQ

### Traffic Manager 自体には Azure リージョンの障害に対する回復性がありますか。

Azure Traffic Manager は、正常性チェックとリージョン間の自動フェールオーバーが組み込まれており、Azure で可用性の高いアプリケーションを配信するための重要なコンポーネントの 1 つとなっています。Traffic Manager を使用することで、リージョン全体にわたって Azure に障害が発生した場合でも回復できるアプリケーションを実現できます。

そのため、Traffic Manager は、リージョンの障害に対する回復性を含め、非常に高レベルの可用性を提供する必要があります。

Traffic Manager の構成要素の中には Azure で実行されるものがありますが、これらは複数のリージョンに分散されていて、いずれかの Azure リージョン全体に障害が発生した場合でも回復できるように設計されています。この回復性は Traffic Manager のすべての構成要素、つまり、DNS ネーム サーバー、API、ストレージ層、エンドポイント監視サービスに適用されます。

そのため、万一 Azure リージョン全体が停止した場合でも、Traffic Manager は引き続き通常どおりに機能します。お客様がアプリケーションを複数の Azure リージョンにデプロイされている場合も、Traffic Manager を利用して、アプリケーションの使用可能なインスタンスにトラフィックを転送できます。

### リソース グループの場所の選択は Traffic Manager にどのような影響を与えますか。

Traffic Manager は、1 つのグローバル サービスです。いずれかのリージョンに限定されたものではありません。リソース グループの場所をどこに選択しても、そのリソース グループにデプロイされる Traffic Manager プロファイルに違いはありません。

Azure Resource Manager では、すべてのリソース グループに対して場所を指定することが求められます。これに基づいて、リソース グループにデプロイされたリソースの既定の場所が決定されます。Azure ポータルで Traffic Manager プロファイルを作成する際、リソース グループを新規作成すると、この場所を指定するよう求められます。

Traffic Manager プロファイル自体の場所には、常に **グローバル** が使用されます。これは指定可能な唯一の値であるため、Azure ポータル、PowerShell、または Azure CLI ではアクセスできません。リソース グループの既定値は、この値によって上書きされます。

### 各エンドポイントの現在の正常性を確認するには、どうすればよいですか。

各エンドポイントとプロファイル全体の現在の監視状態は、Azure ポータルに表示されます。この情報は、Traffic Manager の [REST API](https://msdn.microsoft.com/library/azure/mt163667.aspx)、[PowerShell コマンドレット](https://msdn.microsoft.com/library/mt125941.aspx)、および [クロスプラットフォームの Azure CLI](../xplat-cli-install.md) を使用して取得することもできます。

エンドポイントの過去の正常性に関する履歴情報を表示することはできません。また、エンドポイントの正常性の変化に関する通知を構成することもできません。

### HTTPS エンドポイントを監視できますか。

はい。Traffic Manager は、HTTPS 経由のプローブをサポートしています。監視構成でプロトコルに **HTTPS** を指定するだけで済みます。以下の点に注意してください。

- サーバー側の証明書は検証されません。
- SNI サーバー側証明書はサポートされていません。
- クライアント証明書はサポートされていません。

### エンドポイントの正常性チェックには、どのようなホストヘッダーが使用されますか。

HTTP/S 正常性チェックで使用されるホストヘッダーは、各エンドポイントに関連付けられている DNS 名です。Azure ポータル、[PowerShell コマンドレット](https://msdn.microsoft.com/library/mt125941.aspx)、[クロスプラットフォームの Azure CLI](../xplat-cli-install.md)、および [REST API](https://msdn.microsoft.com/library/azure/mt163667.aspx) では、これがエンドポイントのターゲットとして表示されます。

この値は、エンドポイント構成に含まれています。ホスト ヘッダーで使用される値を、ターゲットプロパティとは別に指定することはできません。


## 次のステップ

[Traffic Manager のしくみ](traffic-manager-how-traffic-manager-works.md)を確認する。

Traffic Manager でサポートされている[トラフィック ルーティング方法](traffic-manager-routing-methods.md)の詳細を確認する。

[Traffic Manager プロファイルの作成](traffic-manager-manage-profiles.md)方法を確認する。

Traffic Manager エンドポイントの [低下状態に関するトラブルシューティング](traffic-manager-troubleshooting-degraded.md)を行う。

<!---HONumber=AcomDC_0706_2016-->