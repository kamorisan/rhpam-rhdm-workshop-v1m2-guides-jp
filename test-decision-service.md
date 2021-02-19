# 6. デシジョンサービスのテスト

デシジョンサービス を作成し、Red Hat Process Automation Manager 実行サーバーにデプロイしたので、それをテストする方法を学びましょう。

このセクションでは、以下のことを学習します:

1. ルールの実装を検証するためのテストシナリオの作成
2. テストシナリオの実行
3. デプロイした業務アプリケーションを外部アプリを使用してテストする

# テストシナリオの紹介

デシジョンサービスをテストするには、さまざまな方法があります。
個々のルール、ルールのグループ、そしてサービス全体のテストケースを書くのは重要です。

これらのテストケースは、サービスのコードとルールがコンパイルされてパッケージ化されたときに自動的に実行されます。
これにより、本番環境にデプロイする前にサービスが適切にテストされていることを保証すると共に、実行されるロジックと決定が仕様に沿って正しく行われていることが保証されます。
Red Hat Process Automation Manager は、この種のテストシナリオを完全にサポートします。

Red Hat Process Automation Manager には、洗練された _テスト シナリオ_ の機能が含まれており、ルールのための様々なテストシナリオを構築することができます。

## Test Scenario

1. プロジェクトの `アセットライブラリ` ビューに戻ります。青い `アセットの追加` ボタンをクリックします。画面左上のアセットフィルタで `デシジョン` を選択し、デシジョンアセットをフィルタリングします。

2. `テストシナリオ`を選択します。シナリオの名前を `risk-evaluation-tests` とし、パッケージを `com.myspace.cd_project` に設定します。ソースタイプを `RULE` に設定します。

    ![Test Scenario Create]({% image_path test-scenario-create.png %}){:width="800px"}

3. シナリオテストツールは、特定の入力データを _Given_ し、特定の結果を _Expect_ するという概念を使用しています。テストシナリオを実装するためには、ルールの入力データ( _Given_ )と、与えられた入力データに対してルールが生成することを期待する結果( _EXPECT_ )を指定する必要があります。

4. エディタで `データオブジェクト` タブをクリックします。すべてが正しく表示されていれば、`AdditionalInformation`, `CreditCardHolder`, `FraudData` の3つのデータ型がリストアップされているはずです。

5. `モデル` タブに戻ります。ルールをテストするために、入力データと期待される出力を提供する必要があります。デシジョンテーブルは2つのデータ型、`CreditCardHolder` と `FraudData` で動作します。そこで、シナリオテストテーブルの_Given_ 部分に `CreditCardHolder` の列を作成してみましょう。テーブルの _INSTANCE 1_ セルをクリックします。エディタの右側の _テストツール_ パネルで、_データオブジェクト_ `CreditCardHolder` を展開し、`status` フィールドを選択し、`データオブジェクトの挿入` ボタンをクリックします。

    ![Test Scenario Add Given CCH Status]({% image_path test-scenario-add-given-cch-status.png %}){:width="800px"}

    _Given_ セクションの列は `CreditCardHolder` オブジェクトの `status` フィールドを表すように設定されます。

6. テーブルの _Given_ セクションに列を追加するには、_Given_ 列を右クリックして `右に列を挿入` を選択します。

    ![Test Scenario Given Insert Column Right]({% image_path test-scenario-given-insert-column-right.png %}){:width="800px"}

7. `INSTANCE` または `PROPERTY` と書かれたセルをクリックし、_テストツール_ パネルで、_データオブジェクト_ `FraudData`を展開し、フィールド `totalFraudAmount` を選択し、`データオブジェクトの挿入` ボタンをクリックします。

8. 入力データを定義する2つの列を設定したので、期待される結果を設定する列を設定します。`EXPECT` セル内の `INSTANCE` または `PROPERTY` セルをクリックします。右側の _テストツール_ パネルで、`FraudData` オブジェクトを展開し、`disputeRiskRating` フィールドを選択し、`データオブジェクトの挿入` ボタンをクリックします。

    ![Test Scenario Table Configured]({% image_path test-scenario-table-configured.png %}){:width="800px"}

9. テーブルが設定されたので、テストケースの追加を開始します。以下の値を持つ行を追加します。
    - Given:
        - CreditCardHolder.status: `Standard`
        - FraudData.totalFraudAmount: `42`
    - Expect:
        - FraudData.disputeRiskRating: `0`

    ![Test Scenario First Test]({% image_path test-scenario-first-test.png %}){:width="800px"}

10. メニューの `実行` ボタン（ `検証` ボタンの隣）をクリックしてテストを実行します。すべてが正しい場合、テストが実行され、結果が表示されます。

11. シナリオテストを完成させるために、例えば、いくつかの追加テストを追加します。
    - Given:
        - CreditCardHolder.status: `Gold`
        - FraudData.totalFraudAmount: `863`
    - Expect:
        - FraudData.disputeRiskRating: `1`

    ![Test Scenario Two Test]({% image_path test-scenario-two-tests.png %}){:width="800px"}

ツールの動作を確認するために、期待される列に間違った値を自由に試してみてください。

## Webアプリケーションからのテスト

前の演習では、_テストシナリオ_ ツールを使用して、チャージバック申請処理プロジェクトのルールをテストしました。今度は、それが公開する REST API を介してデプロイされたサービスをテストします。

そのためには、用意されているシンプルなWebアプリケーションを使ってみましょう。
このアプリケーションでは、クレジットカード所持者のデータと、トランザクションのデータを入力することができます。
そのデータはデシジョンサーバーに送信され、トランザクションのリスクを計算して自動処理できるかどうかを判断してくれます。

​	![ReactJS App]({% image_path reactjs-app.png %}){:width="800px"}

1. アプリケーションにアクセスするには、OpenShiftコンソールに戻り、`react-web-app` Route をクリックして新しいタブで開きます。

   ![ReactJS App Route]({% image_path openshift-react-app-route.png %}){:width="800px"}

2. ReactのWebアプリケーションを開いたら、以下の詳細を入力します:

    - **Name:** Jim
    - **Age:** 52
    - **Status:** Gold
    - **Description:** Delta Airlines
    - **Amount:** 1000

3. 次に、`Submit`ボタンをクリックします。アプリケーションが デシジョンサーバー に RESTful リクエストを送信します。すべてが正常に動作していれば、デシジョンサーバー はアプリケーションに表示される結果を送信します。

    ![ReactJS App Request Response]({% image_path reactjs-app-request-response.png %}){:width="800px"}

`riskRating` が **1** に設定されており、トランザクションが自動処理の対象となっていることがわかります。
ルールに実装したユースケースがすべてカバーされているかどうかを確認するために、さまざまな値を使って デシジョンサービス をテストしてみてください。

これでデシジョンサービスのテストに成功しました。
次のセクションでは、サービスによって公開されているRESTful APIを詳しく見ていき、APIのドキュメントページからサービスをテストする方法を見ていきます。
