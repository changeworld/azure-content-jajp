<properties
   pageTitle="SQL Data Warehouse への SQL コードの移行 | Microsoft Azure"
   description="ソリューション開発のための Azure SQL Data Warehouse への SQL コードの移行に関するヒント"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="06/30/2016"
   ms.author="lodipalm;barbkess;sonyama;jrj"/>

# SQL Data Warehouse への SQL コードの移行

他のデータベースから SQL Data Warehouse にコードを移行するときは、多くの場合、コード ベースに変更を加える必要があります。一部の SQL Data Warehouse 機能は分散環境で機能するように設計されているため、大幅にパフォーマンスを向上できます。ただし、パフォーマンスと拡張性を維持するには、一部の機能が使用できなくなる場合もあります。

## 一般的な T-SQL の制限事項

Azure SQL Data Warehouse でサポートされていない一般的な機能を次に示します。リンクをクリックすると、サポートされていない機能に対する解決策が表示されます。

- [更新での ANSI の JOIN][]
- [削除での ANSI の JOIN][]
- [MERGE ステートメント][]
- 複数データベースの JOIN
- [カーソル][]
- [SELECT..INTO][]
- [INSERT..EXEC][]
- OUTPUT 句
- インライン ユーザー定義関数
- 複数ステートメント関数
- [共通テーブル式](#Common-table-expressions)
- [再帰共通テーブル式 (CTE)](#Recursive-common-table-expressions-(CTE)
- CLR 関数およびプロシージャ
- $partition 関数
- テーブル変数
- テーブル値パラメーター
- 分散トランザクション
- コミット/ロールバック処理
- トランザクションの保存
- 実行コンテキスト (EXECUTE AS)
- [rollup / cube / grouping セット オプションによる句ごとのグループ化][]
- [8 を超えるの入れ子のレベル][]
- [ビューを使用した更新][]
- [変数代入のための SELECT の使用][]
- [動的 SQL 文字列の MAX 以外のデータ型][]

こうした制限の大部分は回避できます。上記の関連する開発記事に説明が記載されています。

## サポートされる CTE 機能

SQL Data Warehouse では、共通テーブル式 (CTE) が部分的にサポートされています。現時点でサポートされている CTE 機能を次に示します。

- CTE は、SELECT ステートメントで指定できます。
- CTE は、CREATE VIEW ステートメントで指定できます。
- CTE は、CREATE TABLE AS SELECT (CTAS) ステートメントで指定できます。
- CTE は、CREATE REMOTE TABLE AS SELECT (CRTAS) ステートメントで指定できます。
- CTE は、CREATE EXTERNAL TABLE AS SELECT (CETAS) ステートメントで指定できます。
- リモート テーブルは、CTE から参照できます。
- 外部テーブルは、CTE から参照できます。
- 複数の CTE クエリ定義は、CTE で定義できます。

## CTE の制限事項

SQL Data Warehouse での共通テーブル式の制限事項を次に示します。

- CTE の後ろには単一の SELECT ステートメントを続ける必要があります。INSERT、UPDATE、DELETE、MERGE ステートメントはサポートされていません。
- 自己参照を含む共通テーブル式 (再帰共通テーブル式) は、サポートされていません (以下のセクションを参照してください)。
- CTE で、複数の WITH 句を指定することはできません。たとえば、CTE\_query\_definition にサブクエリが含まれている場合、そのサブクエリに、別の CTE を定義する入れ子になった WITH 句を含めることはできません。
- TOP 句が指定されている場合を除き、ORDER BY 句は、CTE\_query\_definition で使用できません。
- バッチの一部であるステートメントで CTE を使用する場合は、前のステートメント末尾にセミコロンを付ける必要があります。
- Sp\_prepare によって作成されるステートメントで使用される場合、CTE は PDW の他の SELECT ステートメントと同様に動作します。ただし、CTE が sp\_prepare で準備される CETAS の一部として使用される場合、バインドを sp\_prepare に対して実装する方法によって、動作が SQL Server および他の PDW ステートメントとは異なる場合があります。CTE を参照する SELECT が CTE に存在しない間違った列を使用している場合、sp\_prepare はエラーを検出せずに渡されますが、代わりに sp\_execute でエラーがスローされます。

## 再帰 CTE

再帰 CTE は、SQL Data Warehouse ではサポートされていません。再帰 CTE の移行は複雑であるため、複数の手順に分けて実行することをお勧めします。通常、再帰的な中間クエリの反復処理時に、ループを使用したり、一時テーブルに値を取り込んだりできます。一時テーブルに値が取り込まれたら、単一の結果セットとしてデータを戻すことができます。[group by 句と rollup / cube / grouping sets オプション][]に関する記事でも `GROUP BY WITH CUBE` の解決に同様のアプローチを採用しています。

## システム関数

また、サポートされていないシステム関数もいくつかあります。データ ウェアハウジングで一般的に使用されている主なものを次に示します。

- NEWSEQUENTIALID()
- @@NESTLEVEL()
- @@IDENTITY()
- @@ROWCOUNT()
- ROWCOUNT\_BIG
- ERROR\_LINE()

これらの問題の大部分も回避できます。

たとえば、次のコードは、@@ROWCOUNT 情報を取得するための代替ソリューションです。

```sql
SELECT  SUM(row_count) AS row_count
FROM    sys.dm_pdw_sql_requests
WHERE   row_count <> -1
AND     request_id IN
                    (   SELECT TOP 1    request_id
                        FROM            sys.dm_pdw_exec_requests
                        WHERE           session_id = SESSION_ID()
                        AND             resource_class IS NOT NULL
                        ORDER BY end_time DESC
                    )
;
```

## 次のステップ
サポートされているすべての T-SQL ステートメントの一覧については、「[Transact-SQL トピック][]」をご覧ください。

<!--Image references-->

<!--Article references-->
[更新での ANSI の JOIN]: ./sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-update-statements
[削除での ANSI の JOIN]: ./sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-delete-statements
[MERGE ステートメント]: ./sql-data-warehouse-develop-ctas.md#replace-merge-statements
[INSERT..EXEC]: ./sql-data-warehouse-tables-temporary.md#modularizing-code
[Transact-SQL トピック]: ./sql-data-warehouse-reference-tsql-statements.md

[カーソル]: ./sql-data-warehouse-develop-loops.md
[SELECT..INTO]: ./sql-data-warehouse-develop-ctas.md#selectinto
[group by 句と rollup / cube / grouping sets オプション]: ./sql-data-warehouse-develop-group-by-options.md
[rollup / cube / grouping セット オプションによる句ごとのグループ化]: ./sql-data-warehouse-develop-group-by-options.md
[8 を超えるの入れ子のレベル]: ./sql-data-warehouse-develop-transactions.md
[ビューを使用した更新]: ./sql-data-warehouse-develop-views.md
[変数代入のための SELECT の使用]: ./sql-data-warehouse-develop-variable-assignment.md
[動的 SQL 文字列の MAX 以外のデータ型]: ./sql-data-warehouse-develop-dynamic-sql.md

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=AcomDC_0706_2016-->