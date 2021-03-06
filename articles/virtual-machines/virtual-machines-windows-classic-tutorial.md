<properties
	pageTitle="Windows を実行する VM をクラシック ポータルで作成する | Microsoft Azure"
	description="Azure クラシック ポータルで Windows 仮想マシンを作成します。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/12/2016"
	ms.author="cynthn"/>

# Windows を実行する仮想マシンを Azure クラシック ポータルで作成する

> [AZURE.SELECTOR]
- [Azure クラシック ポータル](virtual-machines-windows-classic-tutorial.md)
- [PowerShell: クラシック デプロイ](virtual-machines-windows-classic-create-powershell.md)

<br>

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)] [Resource Manager デプロイメント モデルを使用してこれらの手順を実行する](virtual-machines-windows-hero-tutorial.md)方法について説明します。

このチュートリアルでは、Windows を実行する Azure 仮想マシン (VM) を Azure クラシック ポータルで簡単に作成する方法を説明します。例として Windows Server イメージを使用しますが、Azure では他にも使用可能なイメージが多数あります。イメージの選択肢は、サブスクリプションによって異なります。たとえば、Windows デスクトップ イメージは MSDN サブスクライバーが使用できます。

このセクションでは、Azure 旧ポータルの **[ギャラリーから]** オプションを使用して仮想マシンを作成する方法について説明します。このオプションは、**[簡易作成]** オプションよりも多数の構成の選択肢があります。たとえば、仮想マシンを仮想ネットワークに参加させる場合、**[ギャラリーから]** オプションを使用する必要があります。

また、[独自のイメージ](virtual-machines-windows-classic-createupload-vhd.md)を使用して VM を作成することもできます。この方法や他の方法の詳細については、「[Windows 仮想マシンを作成するさまざまな方法](virtual-machines-windows-creation-choices.md)」を参照してください。



## ビデオ チュートリアル

次は、このチュートリアルのガイドです。

[AZURE.VIDEO creating-a-windows-vm-on-microsoft-azure-classic-portal]

## <a id="createvirtualmachine"> </a>仮想マシンの作成

[AZURE.INCLUDE [virtual-machines-create-WindowsVM](../../includes/virtual-machines-create-windowsvm.md)]

## 次のステップ

- 仮想マシンにログオンします。手順については、[Windows Server が実行されている仮想マシンへのログオン](virtual-machines-windows-classic-connect-logon.md)に関する記事をご覧ください。

- データを格納するディスクを接続します。空のディスクと、データが含まれているディスクのどちらも接続できます。手順については、「[クラシック デプロイメント モデルを使用して作成された Windows 仮想マシンにデータ ディスクをアタッチする](virtual-machines-windows-classic-attach-disk.md)」をご覧ください。

<!---HONumber=AcomDC_0713_2016-->