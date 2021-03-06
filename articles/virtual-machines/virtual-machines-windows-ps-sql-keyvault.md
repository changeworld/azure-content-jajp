<properties
	pageTitle="Azure VM で SQL Server 用に Azure Key Vault 統合を構成する (リソース マネージャー)"
	description="Azure Key Vault で使用するために SQL Server 暗号化の構成を自動化する方法について説明します。このトピックでは、リソース マネージャーで作成される SQL Server 仮想マシンで Azure Key Vault 統合を使用する方法について説明します。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="rothja"
	manager="jhubbard"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="07/14/2016"
	ms.author="jroth"/>

# Azure VM で SQL Server 用に Azure Key Vault 統合を構成する (リソース マネージャー)

> [AZURE.SELECTOR]
- [リソース マネージャー](virtual-machines-windows-ps-sql-keyvault.md)
- [クラシック](virtual-machines-windows-classic-ps-sql-keyvault.md)

## 概要
[透過的なデータ暗号化 (TDE)](https://msdn.microsoft.com/library/bb934049.aspx)、[列レベルの暗号化 (CLE)](https://msdn.microsoft.com/library/ms173744.aspx)、[バックアップ暗号化](https://msdn.microsoft.com/library/dn449489.aspx) など、SQL Server 暗号化機能が複数存在します。これらの形態の暗号化では、暗号化に利用する暗号鍵を管理し、保存する必要があります。Azure Key Vault (AKV) サービスは、セキュリティを強化し、安全かつ可用性の高い場所で鍵を管理できるように設計されています。[SQL Server コネクタ](http://www.microsoft.com/download/details.aspx?id=45344)を利用すると、SQL Server は Azure Key Vault にある鍵を利用できます。

SQL Server をオンプレミス コンピューターで実行している場合、[いくつかの手順を踏んでオンプレミスの SQL Server コンピューターから Azure Key Vault にアクセスできます](https://msdn.microsoft.com/library/dn198405.aspx)。ただし、Azure VM の SQL Server の場合、*Azure Key Vault 統合*機能を利用し、時間を節約できます。

この機能が有効になっていると、SQL Server コネクタが自動的にインストールされ、Azure Key Vault にアクセスするように EKM プロバイダーが構成され、Vault へのアクセスを許可する資格情報が作成されます。前述のオンプレミス文書の手順を見れば、この機能で手順 2 と 3 が自動化されることがわかります。手動でしなければならないことは、Key Vault と鍵を作成することだけです。そこから先は、SQL VM の設定全体が自動化されます。この機能でこの設定が完了したら、T-SQL ステートメントを実行し、通常どおり、データベースやバックアップの暗号化を開始できます。

[AZURE.INCLUDE [AKV 統合の準備](../../includes/virtual-machines-sql-server-akv-prepare.md)]

## AKV 統合の有効化と構成
AKV 統合はプロビジョニング時に有効にできます。または、既存の VM 用に構成できます。

### 新しい VM
リソース マネージャーで新しい SQL Server 仮想マシンをプロビジョニングしている場合は、Azure のポータルで Azure Key Vault 統合を有効にする手順を提供します。

![SQL Azure Key Vault Integration](./media/virtual-machines-windows-ps-sql-keyvault/azure-sql-arm-akv.png)

プロビジョニングの詳細なチュートリアルについては、「[Azure ポータルでの SQL Server 仮想マシンのプロビジョニング](virtual-machines-windows-portal-sql-server-provision.md)」をご覧ください。

### 既存の VM
既存の SQL Server 仮想マシンの場合は、ご使用の SQL Server 仮想マシンを選択します。**[設定]** ブレードの **[SQL Server の構成]** セクションを選択します。

![既存の VM 用の SQL AKV 統合](./media/virtual-machines-windows-ps-sql-keyvault/azure-sql-rm-akv-existing-vms.png)

**[SQL Server の構成]** ブレードの [Automated Key Vault integration (Key Vault の自動統合)] セクションで **[編集]** ボタンをクリックします。

![既存の VM 用の SQL AKV 統合を構成する](./media/virtual-machines-windows-ps-sql-keyvault/azure-sql-rm-akv-configuration.png)

終了したら、**[SQL Server の構成]** ブレードの下部にある **[OK]** ボタンをクリックして変更を保存します。

>[AZURE.NOTE] テンプレートを使用して AKV 統合を構成することもできます。詳細については、[Azure Key Vault 統合用の Azure クイックスタート テンプレート](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-sql-existing-keyvault-update)に関するページをご覧ください。

[AZURE.INCLUDE [AKV 統合の次のステップ](../../includes/virtual-machines-sql-server-akv-next-steps.md)]

<!---HONumber=AcomDC_0720_2016-->