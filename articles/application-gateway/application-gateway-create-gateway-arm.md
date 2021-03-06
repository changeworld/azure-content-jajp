<properties
   pageTitle="Azure リソース マネージャーを使用した Application Gateway の作成、起動、または削除 | Microsoft Azure"
   description="このページでは、Azure リソース マネージャーを使用して、Azure Application Gateway を作成、構成、起動、および削除する方法について説明します。"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/05/2016"
   ms.author="joaoma"/>


# Azure リソース マネージャーを使用した、Application Gateway の作成、起動、または削除

Azure Application Gateway はレイヤー 7 のロード バランサーです。クラウドでもオンプレミスでも、異なるサーバー間のフェールオーバーと HTTP 要求のパフォーマンス ルーティングを提供します。Application Gateway は、HTTP 負荷分散、Cookie ベースのセッション アフィニティ、Secure Sockets Layer (SSL) オフロードなどのアプリケーション配信機能を備えています。


> [AZURE.SELECTOR]
- [Azure クラシックの PowerShell の手順](application-gateway-create-gateway.md)
- [Azure Resource Manager の PowerShell](application-gateway-create-gateway-arm.md)
- [Azure Resource Manager テンプレート](application-gateway-create-gateway-arm-template.md)


<BR>


この記事では、Application Gateway を作成、構成、起動、および削除する手順について説明します。


>[AZURE.IMPORTANT] Azure リソースを使用する前に、Azure は現在、リソース マネージャーのデプロイメント モデルと従来のデプロイメント モデルの 2 種類を備えていることを理解しておくことが重要です。Azure リソースを使用する前に、必ず[デプロイ モデルとツール](../azure-classic-rm.md)について理解しておいてください。この記事の上部にあるタブをクリックすると、さまざまなツールについてのドキュメントを参照できます。このドキュメントでは、Azure リソース マネージャーを使用した Application Gateway の作成について説明します。クラシック バージョンを使用する場合は、「[Application Gateway の作成、起動、または削除](application-gateway-create-gateway.md)」を参照してください。



## 開始する前に

1. Web Platform Installer を使用して、Azure PowerShell コマンドレットの最新バージョンをインストールします。[ダウンロード ページ](https://azure.microsoft.com/downloads/)の **Windows PowerShell** セクションから最新バージョンをダウンロードしてインストールできます。
2. 既存の仮想ネットワークがある場合は、既存の空のサブネットを選択するか、既存の仮想ネットワークにアプリケーション ゲートウェイ専用の新しいサブネットを作成します。アプリケーション ゲートウェイの背後にデプロイするリソースとは異なる仮想ネットワークにアプリケーション ゲートウェイをデプロイすることはできません。
3. Application Gateway を使用するように構成するサーバーが存在している必要があります。つまり、仮想ネットワーク内、または割り当てられたパブリック IP/VIP を使用してエンドポイントが作成されている必要があります。

## Application Gateway の作成に必要な構成


- **バックエンド サーバー プール:** バックエンド サーバーの IP アドレスの一覧。一覧の IP アドレスは、仮想ネットワークのサブネットに属しているか、パブリック IP/VIP である必要があります。
- **バックエンド サーバー プールの設定:** すべてのプールには、ポート、プロトコル、Cookie ベースのアフィニティなどの設定があります。これらの設定はプールに関連付けられ、プール内のすべてのサーバーに適用されます。
- **フロントエンド ポート:** このポートは、Application Gateway で開かれたパブリック ポートです。このポートにトラフィックがヒットすると、バックエンド サーバーのいずれかにリダイレクトされます。
- **リスナー:** リスナーには、フロントエンド ポート、プロトコル (Http または Https、大文字小文字の区別あり)、および SSL 証明書名 (オフロードの SSL を構成する場合) があります。
- **ルール:** ルールはリスナーとバックエンド サーバー プールを結び付け、トラフィックが特定のリスナーにヒットした際に送られるバックエンド サーバー プールを定義します。



## 新しいアプリケーション ゲートウェイの作成

Azure クラシックと Azure リソース マネージャーの使用方法の違いは、設定が必要な Application Gateway と項目を作成する順番にあります。

リソース マネージャーを使用すると、Application Gateway を作成するすべての項目は個別に構成され、その後結合されて Application Gateway のリソースを作成します。


Application Gateway を作成するために必要な手順を次に示します。

1. リソース マネージャーのリソース グループを作成します。
2. Application Gateway の仮想ネットワーク、サブネット、およびパブリック IP を作成します。
3. Application Gateway 構成オブジェクトを作成します。
4. Application Gateway のリソースを作成します。


## リソース マネージャーのリソース グループの作成

Azure PowerShell の最新バージョンを使用していることを確認します。詳細については、「[Azure Resource Manager での Azure PowerShell の使用](../powershell-azure-resource-manager.md)」を参照してください。

### 手順 1.
Azure Login-AzureRmAccount へのログイン

資格情報を使用して認証を行うように求めるメッセージが表示されます。<BR>
### 手順 2.
アカウントのサブスクリプションを確認します。

		Get-AzureRmSubscription

### 手順 3.
使用する Azure サブスクリプションを選択します。<BR>

		Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### 手順 4.
新しいリソース グループを作成します (既存のリソース グループを使用する場合は、この手順をスキップしてください)。

    New-AzureRmResourceGroup -Name appgw-rg -location "West US"

Azure リソース マネージャーでは、すべてのリソース グループの場所を指定する必要があります。指定した場所は、そのリソース グループ内のリソースの既定の場所として使用されます。Application Gateway を作成するためのすべてのコマンドで、同じリソース グループが使用されていることを確認します。

上記の例では、"appgw-RG" という名前のリソース グループと "West US" という名前の場所を作成しました。

>[AZURE.NOTE] Application Gateway のカスタム プローブを構成する必要がある場合は、「[Azure Resource Manager で PowerShell を使用して Azure Application Gateway のカスタム プローブを作成する](application-gateway-create-probe-ps.md)」を参照してください。詳細については、「[Application Gateway による正常性監視の概要](application-gateway-probe-overview.md)」を参照してください。



## Application Gateway の仮想ネットワークとサブネットの作成

次の例では、リソース マネージャーを使用して仮想ネットワークを作成する方法を示します。

### 手順 1.

アドレス範囲 10.0.0.0/24 を仮想ネットワークの作成に使用するサブネットの変数に割り当てます。

	$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24


### 手順 2.

サブネット 10.0.0.0/24 とプレフィックス 10.0.0.0/16 を使用して、米国西部リージョンのリソース グループ "appgw-rg" に、"appgwvnet" という名前の仮想ネットワークを作成します。

	$vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $subnet


### 手順 3.

Application Gateway を作成する次の手順のために、サブネット変数を割り当てます。

	$subnet=$vnet.Subnets[0]

## フロントエンド構成のパブリック IP アドレスの作成

米国西部リージョンのリソース グループ "appgw-rg" に、パブリック IP リソース "publicIP01" を作成します。

	$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -name publicIP01 -location "West US" -AllocationMethod Dynamic


## Application Gateway 構成オブジェクトの作成

Application Gateway を作成する前に、すべての構成項目を設定する必要があります。次の手順では、Application Gateway のリソースに必要な構成項目を作成します。

### 手順 1.

"gatewayIP01" という名前の Application Gateway の IP 構成を作成します。Application Gateway が起動すると、構成されているサブネットから IP アドレスが取得されて、ネットワーク トラフィックがバックエンド IP プール内の IP アドレスにルーティングされます。各インスタンスは、1 つの IP アドレスを取得することに注意してください。


	$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet


### 手順 2.

この手順は、IP アドレス "134.170.185.46, 134.170.188.221,134.170.185.50" を使用して、"pool01" という名前のバックエンド IP アドレス プールを構成します。これらは、フロントエンド IP エンドポイントから送信されるネットワーク トラフィックを受信する IP アドレスとなります。独自のアプリケーションの IP アドレス エンドポイントを追加するために、上記の IP アドレスを置き換えます。

	$pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50



### 手順 3.

バックエンド プール内の負荷を分散したネットワーク トラフィックに対して、Application Gateway の設定 "poolsetting01" を構成します。

	$poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled


### 手順 4.

パブリック IP エンドポイントに対して、"frontendport01" という名前のフロントエンド IP ポートを構成します。

	$fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 80

### 手順 5.

"fipconfig01" という名前のフロントエンド IP 構成を作成し、このフロントエンド IP 構成にパブリック IP アドレスを関連付けます。

	$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip


### 手順 6.

"listener01" という名前のリスナーを作成し、フロントエンド IP 構成にフロントエンド ポートを関連付けます。

	$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp

### 手順 7.

ロード バランサーの動作を構成する "rule01" という名前のロード バランサーのルーティング規則を作成します。

	$rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

### 手順 8.

Application Gateway のインスタンスのサイズを構成します。

	$sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

>[AZURE.NOTE]  *InstanceCount* の既定値は 2、最大値は 10 です。*GatewaySize* の既定値は Medium です。Standard\_Small、Standard\_Medium、Standard\_Large のいずれかを選択できます。

## New-AzureRmApplicationGateway を使用した Application Gateway の作成

前の手順の構成項目をすべて使用して、Application Gateway を作成します。この例では、Application Gateway は "appgwtest" という名前です。

	$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku

### 手順 9.
アプリケーション ゲートウェイに接続されているパブリック IP リソースから、アプリケーション ゲートウェイの DNS と VIP の詳細を取得します。

	Get-AzureRmPublicIpAddress -Name publicIP01 -ResourceGroupName appgw-rg  

	Name                     : publicIP01
	ResourceGroupName        : appgwtest 
	Location                 : westus
	Id                       : /subscriptions/<sub_id>/resourceGroups/appgw-rg/providers/Microsoft.Network/publicIPAddresses/publicIP01
	Etag                     : W/"12302060-78d6-4a33-942b-a494d6323767"
	ResourceGuid             : ee9gd76a-3gf6-4236-aca4-gc1f4gf14171
	ProvisioningState        : Succeeded
	Tags                     : 
	PublicIpAllocationMethod : Dynamic
	IpAddress                : 137.116.26.16
	IdleTimeoutInMinutes     : 4
	IpConfiguration          : {
	                             "Id": "/subscriptions/<sub_id>/resourceGroups/appgw-rg/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIPConfigurations/fipconfig01"
	                           }
	DnsSettings              : {
	                             "Fqdn": "ee7aca47-4344-4810-a999-2c631b73e3cd.cloudapp.net"
	                           } 



## Application Gateway の削除

Application Gateway を削除するには、次の手順を実行します。

1. **Stop-AzureRmApplicationGateway** コマンドレットを使用して、ゲートウェイを停止します。
2. **Remove-AzureRmApplicationGateway** コマンドレットを使用して、ゲートウェイを削除します。
3. **Get-AzureRmApplicationGateway** コマンドレットを使用して、ゲートウェイが削除されたことを確認します。

### 手順 1.

Application Gateway オブジェクトを取得し、変数 "$getgw" に関連付けます。

	$getgw =  Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg

### 手順 2.

**Stop-AzureRmApplicationGateway** を使用して、アプリケーション ゲートウェイを停止します。

	Stop-AzureRmApplicationGateway -ApplicationGateway $getgw  


アプリケーション ゲートウェイが停止状態になったら、**Remove-AzureRmApplicationGateway** コマンドレットを使用してサービスを削除します。


	Remove-AzureRmApplicationGateway -Name $appgwtest -ResourceGroupName appgw-rg -Force



>[AZURE.NOTE] **-force** スイッチを使用すると、削除の確認メッセージを表示しないように設定できます。


サービスが削除されたことを確認するには、**Get-AzureRmApplicationGateway** コマンドレットを使用します。この手順は必須ではありません。


	Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg


## 次のステップ

SSL オフロードを構成する場合は、「[クラシック デプロイ モデルを使用して SSL オフロード用にアプリケーション ゲートウェイを構成する](application-gateway-ssl.md)」を参照してください。

内部ロード バランサーと共に使用するようにアプリケーション ゲートウェイを構成する場合は、「[内部ロード バランサー (ILB) を使用したアプリケーション ゲートウェイの作成](application-gateway-ilb.md)」を参照してください。

負荷分散のオプション全般の詳細については、次を参照してください。

- [Azure Load Balancer](https://azure.microsoft.com/documentation/services/load-balancer/)
- [Azure の Traffic Manager](https://azure.microsoft.com/documentation/services/traffic-manager/)

<!---HONumber=AcomDC_0713_2016-->