ドメイン名のレコードが反映されると、カスタム ドメイン名を使用して対象の Azure App Services の Web アプリにアクセスできることをブラウザーで確認できます。

> [AZURE.NOTE] CNAME が DNS に反映されるまで多少の時間がかかります。<a href="http://www.digwebinterface.com/">http://www.digwebinterface.com/</a> などのサービスを使用すると、CNAME を利用できるかどうかを確認できます。

カスタム ドメイン名がトラフィック マネージャーにルーティングされているため、Web アプリがトラフィック マネージャーのエンドポイントとしてまだ追加されていない場合は、名前を適切に解決するためにエンドポイントとして追加する必要があります。これにより、トラフィック マネージャーによって対象の Web アプリにルーティングされます。トラフィック マネージャーのプロファイルに Web アプリをエンドポイントとして追加する方法については、「[エンドポイントの追加と削除](../articles/traffic-manager/traffic-manager-endpoints.md)」を参照してください。

> [AZURE.NOTE] エンドポイントを追加する際に Web アプリが表示されない場合は、Web サイトが **Standard** App Service プラン モードに構成されていることを確認します。Web アプリをトラフィック マネージャーで操作するには、**Standard** モードを使用する必要があります。

1. ブラウザーで、[Azure ポータル](https://portal.azure.com)を開きます。

1. **[Web Apps]** タブで、Web アプリの名前をクリックし、**[設定]**、**[カスタム ドメインと SSL]** の順に選択します。

	![](./media/custom-dns-web-site/dncmntask-cname-6.png)

1. **[カスタム ドメインと SSL]** ブレードで **[外部ドメインの使用]** をクリックします。

	![](./media/custom-dns-web-site/dncmntask-cname-7.png)

1. **[ドメイン名]** ボックスに、この Web アプリと関連付ける Traffic Manager ドメイン名を入力します。

	![](./media/custom-dns-web-site/dncmntask-cname-8.png)

1. **[保存]** をクリックしてドメイン名の構成を保存します。

	構成が完了すると、カスタム ドメイン名が Web アプリの **[ドメイン名]** セクションに表示されます。

この時点でブラウザーに Traffic Manager ドメイン名を入力し、対象の Web アプリに正常にアクセスできることを確認できます。

<!---HONumber=AcomDC_0720_2016-->