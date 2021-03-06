<properties
	pageTitle="Azure Storage と Visual Studio 接続済みサービスの概要 (Web ジョブ プロジェクト)"
	description="Visual Studio 接続済みサービスを使用してストレージ アカウントに接続した後、Visual Studio の Azure Web ジョブ プロジェクトで Azure テーブル ストレージの使用を開始する方法について説明します。"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="storage"
	ms.workload="web"
	ms.tgt_pltfrm="vs-getting-started"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/18/2016"
	ms.author="tarcher"/>

# Azure Storage の使用 (Azure Web ジョブ プロジェクト)

[AZURE.INCLUDE [storage-try-azure-tools](../../includes/storage-try-azure-tools.md)]

## 概要

この記事では、Azure テーブル ストレージ サービスを使用して Azure Web ジョブ SDK バージョン 1.x を操作する方法について説明します。コード サンプルでは [Web ジョブ SDK](../app-service-web/websites-dotnet-webjobs-sdk.md) Version 1.x を使用しています。

Azure テーブル ストレージ サービスを使用すると、大量の構造化データを格納できるようになります。このサービスは、Azure クラウドの内部および外部からの認証された呼び出しを受け付ける NoSQL データストアです。Azure のテーブルは、構造化された非リレーショナル データを格納するのに最適です。詳細については、「[.NET を使用して Azure Table Storage を使用する](storage-dotnet-how-to-use-tables.md#create-a-table)」をご覧ください。

一部のコード スニペットは、手動で呼び出される関数で使用される **Table** 属性を表示します。つまり、トリガー属性を使用して呼び出すのではありません。

## エンティティをテーブルに追加する方法

テーブルにエンティティを追加するには、**Table** 属性を **ICollector<T>** または **IAsyncCollector<T>** パラメーターに指定します (**T** は、追加するエンティティのスキーマです)。この属性のコンストラクターは、テーブルの名前を指定する文字列パラメーターを受け取ります。

次のコード サンプルでは、*Ingress* という名前のテーブルに **Person** エンティティを追加しています。

		[NoAutomaticTrigger]
		public static void IngressDemo(
		    [Table("Ingress")] ICollector<Person> tableBinding)
		{
		    for (int i = 0; i < 100000; i++)
		    {
		        tableBinding.Add(
		            new Person() {
		                PartitionKey = "Test",
		                RowKey = i.ToString(),
		                Name = "Name" }
		            );
		    }
		}

**ICollector** には、**TableEntity** を継承する型または **ITableEntity** を実装する型を用いるのが一般的ですが、必須ではありません。先ほど **Ingress** メソッドに示したコードは、次の 2 つの **Person** クラスのどちらでも正しく動作します。

		public class Person : TableEntity
		{
		    public string Name { get; set; }
		}

		public class Person
		{
		    public string PartitionKey { get; set; }
		    public string RowKey { get; set; }
		    public string Name { get; set; }
		}

Azure Storage API を直接操作する場合は、メソッド シグネチャに **CloudStorageAccount** パラメーターを追加することもできます。

## リアルタイム監視

多くの場合、データの受信関数は大量のデータを処理するため、Web ジョブ SDK のダッシュボードはリアルタイムの監視データを提供します。**[Invocation Log]** セクションは、関数がまだ実行であるかどうかを示します。

![実行中の受信関数](./media/vs-storage-webjobs-getting-started-tables/ingressrunning.png)

**[Invocation Details]** ページには、実行中の関数の進行状況 (書き込まれたエンティティの数) が表示されます。このページで関数の実行を中止することもできます。

![実行中の受信関数](./media/vs-storage-webjobs-getting-started-tables/ingressprogress.png)

関数が終了すると、書き込まれた行の数が **[Invocation Details]** ページに表示されます。

![完了した受信関数](./media/vs-storage-webjobs-getting-started-tables/ingresssuccess.png)

## テーブルから複数のエンティティを読み取る方法

テーブルの読み取りを行うには、**Table** 属性を **IQueryable<T>** パラメーター (**T** は、**TableEntity** を継承する型か、**ITableEntity** を実装する型) に指定します。

次のコード サンプルは、**Ingress** テーブルからすべての行を読み取り、ログに記録します。

		public static void ReadTable(
		    [Table("Ingress")] IQueryable<Person> tableBinding,
		    TextWriter logger)
		{
		    var query = from p in tableBinding select p;
		    foreach (Person person in query)
		    {
		        logger.WriteLine("PK:{0}, RK:{1}, Name:{2}",
		            person.PartitionKey, person.RowKey, person.Name);
		    }
		}

### テーブルから 1 つのエンティティを読み取る方法

1 つのテーブル エンティティにバインドする際に、パーティション キーと行キーを指定できる追加の 2 つのパラメーターを持つ **Table** 属性コンストラクターがあります。

次のコード サンプルは、キュー メッセージで受信したパーティション キー値と行キー値に基づいた **Person** エンティティのテーブル行を読み取ります。

		public static void ReadTableEntity(
		    [QueueTrigger("inputqueue")] Person personInQueue,
		    [Table("persontable","{PartitionKey}", "{RowKey}")] Person personInTable,
		    TextWriter logger)
		{
		    if (personInTable == null)
		    {
		        logger.WriteLine("Person not found: PK:{0}, RK:{1}",
		                personInQueue.PartitionKey, personInQueue.RowKey);
		    }
		    else
		    {
		        logger.WriteLine("Person found: PK:{0}, RK:{1}, Name:{2}",
		                personInTable.PartitionKey, personInTable.RowKey, personInTable.Name);
		    }
		}


この例の **Person** クラスは、**ITableEntity** を実装する必要はありません。

## .NET ストレージ API を直接使用して、テーブルを操作する方法

テーブルを操作する際に、**CloudTable** オブジェクトを使用して、より柔軟に **Table** 属性を使用することもできます。

次のコード サンプルは、**CloudTable** オブジェクトを使用して、1 つのエンティティを *Ingress* テーブルに追加します。

		public static void UseStorageAPI(
		    [Table("Ingress")] CloudTable tableBinding,
		    TextWriter logger)
		{
		    var person = new Person()
		        {
		            PartitionKey = "Test",
		            RowKey = "100",
		            Name = "Name"
		        };
		    TableOperation insertOperation = TableOperation.Insert(person);
		    tableBinding.Execute(insertOperation);
		}

**CloudTable** オブジェクトの使用方法の詳細については、「[.NET を使用して Azure Table Storage を使用する](storage-dotnet-how-to-use-tables.md)」をご覧ください。

## キューのハウツー記事で紹介されている関連トピック

キュー メッセージによってトリガーされるテーブルの処理方法、テーブル処理に固有ではない WebJobs SDK のシナリオの詳細については、「[Azure キュー ストレージと Visual Studio 接続済みサービスの概要 (Web ジョブ プロジェクト)](vs-storage-webjobs-getting-started-queues.md)」をご覧ください。



## 次のステップ

この記事では、Azure テーブルを操作するための一般的なシナリオの処理方法を示すコードのサンプルを提供しました。Azure WebJobs および WebJobs SDK の使用方法の詳細については、「[Azure WebJobs のドキュメント リソース](http://go.microsoft.com/fwlink/?linkid=390226)」をご覧ください。

<!---HONumber=AcomDC_0720_2016-->