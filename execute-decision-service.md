# 7.デシジョンサービスの実行

前のセクションでは、Webアプリケーションを介して デシジョンサービス と対話しました。
このステップでは、Webアプリケーションと デシジョンサービス 間の通信に使用される RESTful API を詳しく見ていきます。
ここでは、Swagger APIのドキュメントを使用します。

Swagger は、RESTful API を記述し、ドキュメント化するための標準的な方法を提供しています。
PAM実行サーバの RESTful API を利用することで、他のプラットフォーム、他のアプリケーションが実行サーバと簡単に通信できるようになります。


実行サーバーの Swagger ページにアクセスするには、まず実行サーバーのURLを取得する必要があります。

1. OpenShiftコンソールで、`rhpam-userX` プロジェクトの `Topology` ビューを開きます。`rhpam7-kieserver` をクリックして別のタブでエンジンを開きます。

    ![Execution Server Route]({% image_path kie-server-route.png %}){:width="800px"}

2. 新しいブラウザタブが開くはずです。URL の最後に `/docs` を付け加えてください。URL は http://insecure-rhpam7-kieserver-rhpam-user1.apps.cluster-rio-d6c5.rio-d6c5.example.opentlc.com/docs/ のようになります。すると、次のようなページが表示されます。

    ![KIE Server Swagger]({% image_path kie-server-swagger.png %}){:width="800px"}

3. このページでは、**KIE session assets** というセクションに移動し、`POST /server/containers/instances/{containerId}` という緑色のバーをクリックします。このRESTful APIの説明が表示されます。

4. パネルの右にある `Try it out` ボタンをクリックすると、リクエストの値を入力することができます。

5. まず、`Parameter content type` と `Response content type` を `application/xml` から `application/json` に変更します。これはリクエストとレスポンスに使うデータフォーマットを指定します。この場合、これはJSONフォーマットです。

    ![Swaggger Application JSON]({% image_path swagger-application-json.png %}){:width="800px"}

6. 次に、評価したいルールの展開を含むコンテナIDを指定する必要があります。コンテナの名前は `ccd-project_1.0.0-SNAPSHOT`{{copy}}です。

7. 最後に、リクエストのBodyを提供します。本文では、ドメインモデルやビジネスモデルに基づいたデータを渡し、ルールを評価します。パネルの `body` テキストエリアに以下のリクエストボディを貼り付けてください。

    `FraudData` の `totalFraudAmount` の値が *1000.0* です。また、`CreditCardHolder` の`Status` は *Gold* です。

    ```
    {  
       "lookup":"ccd-ksession-stateless",
       "commands":[  
          {  
             "insert":{  
                "object":{  
                   "com.myspace.ccd_project.CreditCardHolder":{  
                      "age":40,
                      "status":"Gold"
                   }
                },
                "out-identifier":"ccholder",
                "return-object":true,
                "entry-point":"DEFAULT",
                "disconnected":false
             }
          },
          {  
             "insert":{  
                "object":{  
                   "com.myspace.ccd_project.FraudData":{  
                      "totalFraudAmount":1000.0
                   }
                },
                "out-identifier":"frauddata",
                "return-object":true,
                "entry-point":"DEFAULT",
                "disconnected":false
             }
          },
          {  
             "fire-all-rules":{  
                "max":-1,
                "out-identifier":null
             }
          }
       ]
    }
    ```

8. 上記のデータを入力したら、青い `Execute` ボタンをクリックしてリクエストを実行します。
    _ブラウザでユーザー名とパスワードを聞かれた場合は、Business Centralにログインしたときと同じユーザー名とパスワードを使用してください。_

  ![Swaggger Request]({% image_path swagger-request.png %}){:width="800px"}

正常に動作すると、デシジョンサービスから以下のような応答が返ってきます。

  ![Swaggger Response]({% image_path swagger-response.png %}){:width="800px"}

このサービスを `CreditCardHolder` や `FraudData` に他の値を設定してテストし、出力を確認してみてください。

これで、エンジンと直接対話するためのREST APIを使用した、デシジョンサービスのテストを終了します。
