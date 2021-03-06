<properties 
	pageTitle="Enterprise Integration Pack での XML 検証の概要 | Microsoft Azure App Service | Microsoft Azure" 
	description="Enterprise Integration Pack と Logic Apps での検証のしくみについて説明します。" 
	services="app-service\logic" 
	documentationCenter=".net,nodejs,java"
	authors="msftman" 
	manager="erikre" 
	editor="cgronlun"/>

<tags 
	ms.service="app-service-logic" 
	ms.workload="integration" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/08/2016" 
	ms.author="deonhe"/>

# XML 検証を使用した Enterprise Integration

## 概要
B2B のシナリオでは多くの場合、契約の対象となるパートナーは、データの処理が開始される前に、パートナーの間で交換されるメッセージが有効であることを検証する必要があります。Enterprise Integration Pack では、XML 検証コネクタを使用して、定義済みのスキーマに対してドキュメントを検証できます。

## XML 検証コネクタを使用してドキュメントを検証する方法
1. ロジック アプリを作成し、XML データの検証に使用するスキーマが含まれた[統合アカウントにリンクします](./app-service-logic-enterprise-integration-accounts.md "ロジック アプリへの統合アカウントの関連付けについての詳細情報")。
2. ロジック アプリに **[Request - When an HTTP request is received (要求 - HTTP 要求を受信したとき)]** トリガーを追加します。![](./media/app-service-logic-enterprise-integration-xml/xml-1.png)
3. **[Add an action (アクションの追加)]** を選択し、**[XML Validation (XML の検証)]** アクションを追加します。
4. 検索ボックスに「*xml*」と入力し、すべてのアクションから使用するアクションだけをフィルター処理します。
5. **[XML Validation (XML の検証)]** を選択します。 ![](./media/app-service-logic-enterprise-integration-xml/xml-2.png)
6. **[コンテンツ]** ボックスを選択します。 ![](./media/app-service-logic-enterprise-integration-xml/xml-1-5.png)
7. 検証する内容として body タグを選択します。 ![](./media/app-service-logic-enterprise-integration-xml/xml-3.png)
8. **[スキーマ名]** リスト ボックスを選択し、上側の *[コンテンツ]* の入力内容を検証するために使用するスキーマを選択します。 ![](./media/app-service-logic-enterprise-integration-xml/xml-4.png)
9. 作業内容を保存します。![](./media/app-service-logic-enterprise-integration-xml/xml-5.png)

この時点で、検証コネクタの設定が終了します。実際のアプリケーションでは、検証されたデータを SalesForce などの LOB アプリケーション内に格納する必要がある場合があります。検証の出力を Salesforce に送信するアクションを簡単に追加できます。

これで、HTTP エンドポイントに要求を送信して、検証のアクションをテストできます。

## 次のステップ

[Enterprise Integration Pack についての詳細情報](./app-service-logic-enterprise-integration-overview.md "Enterprise Integration Pack についての詳細情報")

<!---HONumber=AcomDC_0713_2016-->