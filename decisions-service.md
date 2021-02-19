
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

プロジェクトが開発されたら、Business Centralでプロジェクトを構築し、設定されたプロセスサーバーにデプロイします。
プロセスサーバーは、ルールやプロセスを実行するエンジンです。
また、分散実行も可能なので、例えば、ケースルールの実行が銀行の支店で行われる場合、必要な数だけプロセスサーバーを配置して、すべてのプロセスサーバーをBusiness Centralのオーサリングコンソールに接続することができます。

プロセスサーバーは、サーバー構成で構成された機能をサポートします。サーバ構成は、実行サーバのグループの構成を定義するテンプレートです（グループには複数の実行サーバを含めることができます）。

テンプレートから設定できることは、以下の2つです:

  - 機能: プロセスサーバーで実行できること（プロセス、ディシジョン、プランナーのルール）。これらは相互に排他的なものではありません。つまり、実行サーバは、これらの機能のうち、0つ以上の機能を有効にした状態で有効にすることができます。
  - デプロイメントユニット: どのようなアセットのパッケージ(プロジェクト、知識JAR)をサーバーにデプロイして実行可能にしたいかを指定します。


プロセスサーバーへのデプロイ時に、何が起こるのかを一目見てみましょう。

1. Business Central でプロジェクトを _ビルド ＆ デプロイ_ すると、_デプロイメント ユニット_ (KJAR) が作成され、アーティファクト リポジトリ (Maven) にプッシュされます。
2. この後、デプロイ要求が実行サーバーに送信されます。
3. 実行サーバは、特定の _デプロイメント ユニット_ に対するデプロイメント要求を受信します。
4. 実行サーバは _デプロイメント ユニット_ を取得し、初期化を試みます。

このタスクが完了したら、必要に応じて Business Central を使用してデプロイメントユニットを開始、停止、または削除することができます。
また、以前に構築したプロジェクトから追加のデプロイメントユニットを作成し、Business Central で構成された既存または新規のプロセスサーバー上で起動することもできます。

## プロジェクトのデプロイ

Business Centralを使ってプロセスサーバーをチェックしてみましょう。

1. Business Central ワークベンチのホーム画面に戻ります（左上画面の `ホーム` アイコンをクリック）。

2. `デプロイ` をクリックします。これで、_サーバー設定_の画面が開きます。プロセスサーバーで有効にしている機能に注目してください。

   ![Business Central Process Server Server Configurations]({% image_path business-central-server-configuration.png %}){:width="800px"}

3. ホーム画面に戻り、`設計` を選択します。`MySpace` を選択し、次にチャージバック申請処理プロジェクト(`ccd-project`)を選択します。ライブラリビューが開き、すべてのアセットのリストが表示されるはずです。これらのアセットはコンパイルされ、_KJAR_ ( _デプロイメントユニット_ )の中にパッケージ化されます。

4. 右上の `デプロイ` ボタンをクリックします。

    ​	![Business Central Deploy]({% image_path business-central-deploy.png %}){:width="800px"}

プロジェクトが最初にビルドされ、アセットがコンパイルされてパッケージ化され、実行サーバーのコンテナにデプロイされていることがわかります。
ホーム画面に戻り、`デプロイ` を選択します。これで、新しく作成したディシジョンが実行されているコンテナが表示されます。

## 実行

デプロイしたサービスが利用可能か確認してみましょう。

1. ホーム画面から、`デプロイ` を選択します。

2. コンテナのURLをクリックすると、新しいタブが開きます。

     ![Business Central Execution Services Detail]({% image_path business-central-execution-services-detail.png %}){:width="800px"}

3. また、資格情報の入力を求められることがあります。Business Central コンソールにログインする際に使用したのと同じ資格情報を使用します。

    - user: `pamAdmin`
    - password: `redhatpam1!`

    ![Business Central Execution Services Info]({% image_path business-central-execution-services-info.png %}){:width="800px"}

4. `デプロイメントユニット` が実行されている `コンテナ` の詳細について、プロセスサービスが応答して表示します。

おめでとうございます！
エンジン内に最初のビジネスアプリケーションをデプロイしたので、チャージバック申請処理プロジェクトで作成した、ルールのテストを自動化する方法を学びましょう。
