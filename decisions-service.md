
# 5. デシジョンサービス

このセクションでは、以下のことを学習します。

1. 前のセクションで作成したルールの使用方法
2. デシジョンをサービスとして公開するための方法。
3. 承認の自動化とそのルールについて

これまでのラボでは、データオブジェクトのモデルと、そのモデル上で動作するルールとデシジョンを定義してきました。前のセクションでラボを完了した場合は、既存のプロジェクトを使用することができます。

NOTE: _最初からやり直したい場合は、`ccd-project` プロジェクトを削除して、この URL を使用して再インポートすることができます。[https://github.com/kamorisan/rhpam-rhdm-workshop-v1m2-labs-step-3.git](https://github.com/kamorisan/rhpam-rhdm-workshop-v1m2-labs-step-3.git)._

まず、Business Centralを使ったアセットのバージョン管理の概要を説明します。

## アセットのバージョン管理

下の図で示すように、Red Hat Process Automation Manager は、意思決定やプロセスを開発・実行するためのモジュール式のプラットフォームです。
これまでは、ルールやビジネスモデルを開発するために **Business Central** ワークベンチを主に使用してきました。
Business Central を使用して作成または変更したアセットはすべて、リポジトリ（Git ベースのバージョン管理システム）でバージョン管理されています。

​	![RHPAM 7 Architecture]({% image_path rhpam-7-architecture.png %}){:width="800px"}

Business Central は git ベースのリポジトリ内のコードを自動的にバージョン管理し、必要に応じて変更点を確認したり、以前のバージョンにロールバックしたりすることができます。

1. Go to your project library view and select the automated-chargeback rule. Once the editor opens click on the button Latest Version. (**NOTE:** If you re-imported the project then there is probably only 1 version listed).プロジェクト・ライブラリ・ビューに移動し、`automated-chargeback` ルールを選択します。エディタが開いたら、`最新バージョン` ボタンをクリックしてください。(**注:**プロジェクトを再インポートした場合は、おそらく1つのバージョンしか表示されません)。

     ![Business Central Chargeback Versions]({% image_path business-central-chargeback-versions.png %}){:width="800px"}

2. リポジトリに保存されているアセットについては、アセットを作成したユーザーの名前や、アセットが最終的に修正された時刻やデータなど、より多くのメタデータがあります。

     ![Business Central Chargeback Versions Detail]({% image_path business-central-chargeback-versions-detail.png %}){:width="800px"}

3. ドロップダウンボックス内のバージョンのいずれかをクリックすると、タグ付けされたアセットのバージョンが表示されます。

     ![Business Central Chargeback Version]({% image_path business-central-chargeback-version.png %}){:width="800px"}

## デプロイメント・プロセスを理解する

After your project is developed, you can build the project in Business Central and deploy it to a configured Process Server. The Process Server is the engine that executes rules and processes. It also allows distributed execution so if, for example, the execution of the case rules will be performed at bank branches, you can deploy as many Process Servers as you need all connected to your Business Central Authoring console.

The Process Server supports the capabilities configured in its server configuration. A server configuration is the template that defines the configuration of a group of Execution Servers (a group can contain zero or more execution servers).

There are 2 things that you can configure through the template:

  - Capabilities: What can you execute in your Process Server (Process, Decisions, Planner rules). They are not mutually exclusive. I.e. an execution server can be enabled with zero or more of these capabilities enabled.
  - Deployment Unit: what package of assets (project, Knowledge JAR) you want to deploy on the server to make available for execution.


Let's have a glance of what happens under the covers during a deployment in a managed Process Server:

1. When you _Build and Deploy_ a project in Business Central, the _Deployment Unit_ (KJAR) is created and pushed to the artifact repository (Maven);
2. After this, a deployment request is sent to the Execution Server;
3. The managed Execution Server receives a deployment request for a specific _Deployment Unit_;
4. The Execution Server fetches the _Deployment Unit_ and tries to initialize it.

Once this tasks are completed, you can start, stop, or remove deployment units using Business Central as needed. You can also create additional _Deployment Units_ from previously built projects and start them on existing or new Process Servers configured in Business Central.

## Deploying your project

Let's check your Process Server using Business Central.

1. Return to the Home screen of the Business Central workbench (by clicking on the _Home_ icon in the upper left screen).

2. Click on _Deploy_. This will open the _Server Configurations_ perspective. Notice which capabilities you have enabled for your Process Server.

   ![Business Central Process Server Server Configurations]({% image_path business-central-server-configuration.png %}){:width="800px"}

3. Return to the Home screen and select _Design_. Select `MySpace`, next, select your Credit Card Dispute project (`ccd-project`). The Library view should open with a list of all your assets. These assets will be compiled and packaged inside a _KJAR_, a _Deployment Unit_.

4. Click on the _Deploy_ button in the top right corner.

    ​	![Business Central Deploy]({% image_path business-central-deploy.png %}){:width="800px"}

You will see that the project is first built, meaning the assets are compiled and packaged, and then deployed to a Execution Server container. Go back to the Home screen and select Deploy. You will now see a container running with your newly created decisions.

## Execution

Let's check if the service you deployed is available.

1. Go to the Main Menu and select `Deploy`>`Execution Services`.

2. Click on the URL of the container, and a new tab should open:

     ![Business Central Execution Services Detail]({% image_path business-central-execution-services-detail.png %}){:width="800px"}

3. You may also be prompted for credentials. Use the same credentials you used to log into the Business Central console.

    - user: `pamAdmin`
    - password: `redhatpam1!`

    ![Business Central Execution Services Info]({% image_path business-central-execution-services-info.png %}){:width="800px"}

4. Notice the Process Service responds with details about the `Kie Container` where your `Deployment Unit` is running.

Congratulations! Now that you have deployed your first business application within the engine, let's learn how about how to automate tests of the rules you created in the Credit Card Dispute project.
