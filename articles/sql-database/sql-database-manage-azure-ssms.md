<properties 
	pageTitle="SSMS を使用した SQL データベースの管理 | Microsoft Azure" 
	description="SQL Server Management Studio を使用して SQL データベース サーバーとデータベースを管理する方法について説明します。" 
	services="sql-database" 
	documentationCenter=".net" 
	authors="stevestein" 
	manager="jhubbard" 
	editor="tysonn"/>

<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/09/2016" 
	ms.author="sstein"/>


# SQL Server Management Studio を使用した Azure SQL Database の管理 


> [AZURE.SELECTOR]
- [Azure ポータル](sql-database-manage-portal.md)
- [SSMS](sql-database-manage-azure-ssms.md)
- [PowerShell](sql-database-command-line-tools.md)

SQL Server Management Studio (SSMS) を使用して Azure SQL Database の論理サーバーとデータベースを管理できます。このトピックでは、SSMS を使用した一般的なタスクの手順について説明します。始める前に、Azure SQL Database 論理サーバーとデータベースが既にある必要があります。接続してから単純な SELECT クエリを実行する方法については、[最初の Azure SQL Database の作成](sql-database-get-started.md)に関する記事と [SSMS を使用して接続しクエリを実行する](sql-database-connect-query-ssms.md)方法に関する記事をご覧ください。

Azure SQL Database を操作する際は、常に最新バージョンの SSMS を使用することをお勧めします。

> [AZURE.IMPORTANT] 最新バージョンの SQL Server Management Studio (SSMS) を使用して、Microsoft Azure と SQL Database の更新プログラムの同期を維持する必要があります。SQL Database で前のバージョンの SSMS が正常に動作しない場合は、「[SQL Server Management Studio (SSMS) のダウンロード](https://msdn.microsoft.com/library/mt238290.aspx)」を参照してください。

## Azure SQL データベースの作成と管理

**master** データベースに接続している間に、サーバーに新しいデータベースを作成したり、既存のデータベースを変更または削除したりできます。以下の手順では、Management Studio を使用して一般的なデータベース管理タスクを実行する方法について説明します。これらのタスクを実行するには、サーバーのセットアップ時に作成したサーバー レベルのプリンシパル ログインを使用して **master** データベースに接続している必要があります。

Management Studio でクエリ ウィンドウを開くには、[データベース] フォルダーを開き、**[システム データベース]** フォルダーを展開します。**[master]** を右クリックし、**[新しいクエリ]** をクリックします。

-   新しいデータベースを作成するには、**CREATE DATABASE** ステートメントを使用します。詳細については、「[CREATE DATABASE (Azure SQL データベース)](https://msdn.microsoft.com/library/dn268335.aspx)」を参照してください。次のステートメントは、**myTestDB** という名前の新しいデータベースを作成し、既定の最大サイズが 250 GB の Standard S0 エディションのデータベースとして指定します。

        CREATE DATABASE myTestDB
        (EDITION='Standard',
         SERVICE_OBJECTIVE='S0');

**[実行]** をクリックしてクエリを実行します。

-   データベースの名前とエディションを変更する場合など、既存のデータベースを変更するには、**ALTER DATABASE** ステートメントを使用します。詳細については、「[ALTER DATABASE (Microsoft Azure SQL データベース)](https://msdn.microsoft.com/library/ms174269.aspx)」を参照してください。次のステートメントは、前の手順で作成したデータベースのエディションを Standard S1 に変更します。

        ALTER DATABASE myTestDB
        MODIFY
        (SERVICE_OBJECTIVE='S1');

-   既存のデータベースを削除するには、**DROP DATABASE** ステートメントを使用します。詳細については、「[DROP DATABASE (Transact-SQL)](https://msdn.microsoft.com/library/ms178613.aspx)」をご覧ください。次のステートメントは、**myTestDB** データベースを削除します。ただし、次の手順でログインを作成する際に使用するので、ここでは削除しないでください。

        DROP DATABASE myTestBase;

-   master データベースには、すべてのデータベースに関する詳細を表示する際に使用できる **sys.databases** ビューがあります。既存のデータベースをすべて表示するには、次のステートメントを実行します。

        SELECT * FROM sys.databases;

-   SQL Database では、**USE** ステートメントを使用してデータベースを切り替えることはできません。代わりに、ターゲット データベースに直接接続する必要があります。

>[AZURE.NOTE] データベースを作成または変更する Transact-SQL ステートメントの多くは、それ自体のバッチ内で実行する必要があり、他の Transact-SQL ステートメントと共にグループ化することはできません。詳細については、上記のリンクでステートメント固有の情報を参照してください。

## ログインを作成と管理する

**master** データベースでは、ログインと、どのログインにデータベースや他のログインを作成するためのアクセス許可があるかが追跡されます。ログインを管理するには、サーバーのセットアップ時に作成したサーバー レベルのプリンシパル ログインを使用して **master** データベースに接続します。**CREATE LOGIN**、**ALTER LOGIN**、または **DROP LOGIN** ステートメントを使用すると、サーバー全体でログインを管理する master データベースに対してクエリを実行できます。詳細については、「[Microsoft Azure SQL データベースにおけるデータベースとログインの管理](http://msdn.microsoft.com/library/azure/ee336235.aspx)」を参照してください。


-   新しいサーバー レベル ログインを作成するには、**CREATE LOGIN** ステートメントを使用します。詳細については、「[CREATE LOGIN (Microsoft Azure SQL データベース)](https://msdn.microsoft.com/library/ms189751.aspx)」を参照してください。次のステートメントは、**login1** という新しいログインを作成します。**password1** を適切なパスワードに置き換えてください。

        CREATE LOGIN login1 WITH password='password1';

-   データベース レベルのアクセス許可を付与するには、**CREATE USER** ステートメントを使用します。ログインは必ず **master** データベースに作成する必要がありますが、別のデータベースに接続するためのログインに対しては、そのデータベースで **CREATE USER** ステートメントを使用してデータベース レベルのアクセス許可を付与する必要があります。詳細については、「[CREATE USER (Azure SQL Database)](https://msdn.microsoft.com/library/ms173463.aspx)」を参照してください。

-   login1 に **myTestDB** というデータベースに対するアクセス許可を付与するには、次の手順に従います。

 1.  オブジェクト エクスプローラーを更新して、作成した **myTestDB** データベースを表示させるには、オブジェクト エクスプローラーのサーバー名を右クリックして、**[最新の情報に更新]** をクリックします。

     接続を閉じてある場合は、[ファイル] メニューの **[オブジェクト エクスプローラーを接続]** を選択して再接続できます。

 2. **myTestDB** データベースを右クリックし、**[新しいクエリ]** を選択します。

    3.  myTestDB データベースに対して次のステートメントを実行して、サーバー レベルのログイン **login1** に対応する** login1User** という名前のデータベース ユーザーを作成します。

            CREATE USER login1User FROM LOGIN login1;

-   このデータベースに対する適切なレベルのアクセス許可をユーザー アカウントに付与するには、**sp\_addrolemember** ストアド プロシージャを使用します。詳細については、「[sp\_addrolemember (Transact-SQL)](http://msdn.microsoft.com/library/ms187750.aspx)」をご覧ください。次のステートメントは、**login1User** を **db\_datareader** ロールに追加することで、データベースに対する読み取り専用アクセス許可を **login1User** に付与します。

        exec sp_addrolemember 'db_datareader', 'login1User';    

-   ログインのパスワードなど、既存のログインを必要に応じて変更するには、**ALTER LOGIN** ステートメントを使用します。詳細については、「[ALTER LOGIN (Transact-SQL)](https://msdn.microsoft.com/library/ms189828.aspx)」をご覧ください。**ALTER LOGIN** ステートメントは、**master** データベースに対して実行する必要があります。そのデータベースに接続されているクエリ ウィンドウに切り替えてください。

    次のステートメントは、ログイン **login1** を変更してパスワードをリセットします。**newPassword** を適切なパスワードに、**oldPassword** をログインの現在のパスワードに置き換えます。

        ALTER LOGIN login1
        WITH PASSWORD = 'newPassword'
        OLD_PASSWORD = 'oldPassword';

-   既存のログインを削除するには、**DROP LOGIN** ステートメントを使用します。サーバー レベルでログインを削除すると、それに関連付けられているデータベース ユーザー アカウントも削除されます。詳細については、「[DROP DATABASE (Transact-SQL)](https://msdn.microsoft.com/library/ms178613.aspx)」をご覧ください。**DROP LOGIN** ステートメントは、**master** データベースに対して実行する必要があります。次のステートメントは、ログイン **login1** を削除します。

        DROP LOGIN login1;

-   master データベースには、ログインを表示する際に使用できる **sys.sql\_logins** ビューがあります。既存のログインをすべて表示するには、次のステートメントを実行します。

        SELECT * FROM sys.sql_logins;

## 動的管理ビューを使用して SQL データベースを監視する</h2>

SQL データベースでは、個々のデータベースを監視するのに使用できる動的管理ビューがいくつかサポートされています。以下に、これらのビューで取得できるモニター データの種類の例を示します。すべての種類と詳しい使用例については、「[動的管理ビューを使用した Azure SQL Database の監視](https://msdn.microsoft.com/library/azure/ff394114.aspx)」をご覧ください。

-   動的管理ビューに対してクエリを実行するには、**VIEW DATABASE STATE** アクセス許可が必要です。**VIEW DATABASE STATE** アクセス許可を特定のデータベース ユーザーに付与するには、サーバー レベルのプリンシプル ログインで管理するデータベースに接続し、データベースに対して次のステートメントを実行します。

        GRANT VIEW DATABASE STATE TO login1User;

-   **sys.dm\_db\_partition\_stats** ビューを使用して、データベース サイズを計算します。**sys.dm\_db\_partition\_stats** ビューは、データベース内のすべてのパーティションのページと行数情報を返します。これらを使ってデータベース サイズを計算できます。次のクエリはデータベース サイズを MB 単位で返します。

        SELECT SUM(reserved_page_count)*8.0/1024
        FROM sys.dm_db_partition_stats;   

-   データベースに関連付けられている内部タスクと現在のユーザー接続に関する情報を取得するには、**sys.dm\_exec\_connections** ビューと **sys.dm\_exec\_sessions** ビューを使用します。次のクエリは現在の接続に関する情報を返します。

        SELECT
            e.connection_id,
            s.session_id,
            s.login_name,
            s.last_request_end_time,
            s.cpu_time
        FROM
            sys.dm_exec_sessions s
            INNER JOIN sys.dm_exec_connections e
              ON s.session_id = e.session_id;

-   キャッシュされたクエリ プランの集計パフォーマンス統計を取得するには、**sys.dm\_exec\_query\_stats** ビューを使用します。次のクエリは、平均 CPU 時間の上位 5 クエリに関する情報を返します。

        SELECT TOP 5 query_stats.query_hash AS "Query Hash",
            SUM(query_stats.total_worker_time), SUM(query_stats.execution_count) AS "Avg CPU Time",
            MIN(query_stats.statement_text) AS "Statement Text"
        FROM
            (SELECT QS.*,
            SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
            ((CASE statement_end_offset
                WHEN -1 THEN DATALENGTH(ST.text)
                ELSE QS.statement_end_offset END
                    - QS.statement_start_offset)/2) + 1) AS statement_text
             FROM sys.dm_exec_query_stats AS QS
             CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
        GROUP BY query_stats.query_hash
        ORDER BY 2 DESC;
 
 

<!---HONumber=AcomDC_0706_2016-->