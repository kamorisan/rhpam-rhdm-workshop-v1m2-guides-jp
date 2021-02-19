# 3. Business Object Model

このセクションでは、以下のことを学習します:

1. Business Object Models とは？
2. Business Central で、Business Object Models を作成する方法
3. プロジェクトをスペースにインポートする方法

## ビジネスドメインのコンテキスト

ビジネスドメインのエキスパートであるあなたは、自動化しようとしている業務のドメインモデルが何であるかを定義する必要があります。
Eric Evans氏によって提唱された [_Domain Driven Design_](https://en.wikipedia.org/wiki/Domain-driven_design){:target="_blank"} には3つの主要な指針があります。

- コアドメインに焦点を当てる。
- ドメインエキスパートと開発者の創造的なコラボレーションの中でモデルを探求する。
- 明示的に境界づけられたコンテキストの中でユビキタス言語を話す。

この演習における、コアドメインの業務を自動化する際の最初の非常に重要なタスクは、チャージバックのドメインのコンテキスト内でビジネスエンティティの定義を作成することです。
このケースでは、最初のエンティティは `Credit Card Holder` です。私たちのユースケースのコンテキストでの定義は、他の組織内での定義とは全く異なるかもしれません。
しかし、私たちのユースケース内では、それはドメインエキスパートと開発者のチームの間で共通の定義となり、私たちの _ユビキタス言語_ の一部となるでしょう。

## Credit Card Holder エンティティの作成

1. **Business Central**にアクセスするには、ブラウザで [Openshift Console]({{ OPENSHIFT_CONSOLE_URL }}){:target="_blank"} を開きます。すでにログインしている場合は、Step2へジャンプします。まだログインしていない場合、またはログアウトされている場合は、以下の認証情報を使用してログインしてください。

    - user: `userX`{{copy}}
    - password: `openshift`{{copy}}

    ![OpenShift Console]({% image_path openshift-console.png %}){:width="800px"}

2. `Developer` パースペクティブを選択し、左側のメニューから `Topology` を選択します。`rhpam-userX` プロジェクトが選択されていることを確認してください。

3. `rhpam7-rhpamcentr`、`rhpam7-kieserver`、`react-web-app` の3つのコンポーネントが表示されています。このページの `rhpam7-rhpamcentr` に、Business Centralにアクセスするためのリンクが表示されています。これをクリックすると、別のタブでBusiness Centralを開くことができます。

    ![PAM Project]({% image_path topology-details.png %}){:width="800px"}

4. Business Central へのログインは、以下の認証情報を使用してください。

    - user: `pamAdmin`{{copy}}
    - password: `redhatpam1!`{{copy}}

    ![Business Central Console]({% image_path business-central-console.png %}){:width="800px"}

5. メインメニューから _設計_ を選択します。ワークスペースにリダイレクトされます。スペースのリストが表示され、MySpaceという名前のスペースが1つ表示されます。

6. _MySpace_ をクリックします。 このスペースでプロジェクトを定義し、その中でプロジェクトアセットを定義します。

7. このプロジェクトはケースマネジメントの実装に使われる予定なので、新しい `ケースプロジェクト` を追加する必要があります。これを行うには、_プロジェクトの追加_ ボタンの右隣にある矢印をクリックして `ケースプロジェクト` オプションを選択します。

    ![Business Central Asset CCD Project]({% image_path add-new-case-project.png %}){:width="800px"}

8.  プロジェクトの追加のウィンドウが開いたら、次のように入力します。

      * プロジェクト名（Name）: `ccd-project`
      * 説明: `Credit card dispute business automation project`

    プロジェクトの作成が完了すると、MySpaceにプロジェクトが表示されます。

    ![Business Central Asset CCD Project]({% image_path business-central-asset-ccd-project.png %}){:width="800px"}

9.  `ccd-project` を選択します。プロジェクトの内容が記載された以下のページが表示されるはずです。

    ![Business Central Asset Empty Project]({% image_path business-central-asset-empty-project.png %}){:width="800px"}

10. `アセットの追加` という青いボタンをクリックして下さい。アセットとは、_ルール、プロセス、デシジョンテーブル、データオブジェクト、データフォームなど_ のプロジェクトのビジネスリソースのことです。

    ![Business Central Asset Catalog]({% image_path business-central-asset-catalog.png %}){:width="800px"}

11. カタログから _データオブジェクト_ のウィザードを選択し、_CreditCardHolder_ のビジネスオブジェクトモデルを作成します。

    * データオブジェクト: `CreditCardHolder`{{copy}}
    * パッケージ: `com.myspace.ccd_project` を、リストから選択

    ![Business Central CCD Object]({% image_path business-central-CCD-object-new.png %}){:width="800px"}

12. 新しく作成されたオブジェクトにはプロパティがありませんので、`+フィールドを追加` ボタンをクリックして、CreditCardHolderデータオブジェクトにプロパティを追加します。

    ![Business Central CCD Object New Empty]({% image_path business-central-CCD-object-new-empty.png %}){:width="800px"}

13. _新規フィールド_ ウィンドウで、以下の値を入力し、_作成_ をクリックします。

    - ID: `age`
    - ラベル: `Age`
    - タイプ: `Integer`

    ![Business Central CCD Object New Properties]({% image_path business-central-CCD-object-new-properties.png %}){:width="800px"}

プロセスや意思決定を自動化するための最初のステップは、Business Object Model を定義して指定することです。
この演習では、エンティティ `CreditCardHolder` を作成し、その `age` フィールドを定義しました。
これらのエンティティは、意思決定やプロセスの実行に必要な情報を格納するために使用されます。

## Business Central でのプロジェクト管理

新しいプロジェクトと新しいデータオブジェクトを作成することがいかに簡単かを学びました。
それでは、より多くのビジネスオブジェクトモデルが定義されている、既存のプロジェクトをインポートしてみましょう。

そのためには、`ccd-project` プロジェクトを一旦削除して git リポジトリから既存のプロジェクトをインポートする方法を学びましょう。


### ccd-project プロジェクトを削除

  1. 現在のプロジェクトを削除します。
  2. 画面上部のメインメニューの下にあるリストから _ccd-project_ をクリックすると、プロジェクトのホームページに戻ります。
    ![Business Central Breadcrumb bar ccd project]({% image_path business-central-breadcrumb-bar-ccd-project.png %}){:width="800px"}

  3. 右上にある3点記号をクリックして、_プロジェクトの削除_ を選択します。
    ![Business Central Delete CCD Project]({% image_path business-central-delete-ccd-project.png %}){:width="800px"}

  4. _ccd-project_ と入力し、`プロジェクトの削除` をクリックします。
  5. もし削除する際に確認の画面が表示された場合は、 `保存されていない変更を破棄して続行する` を選択します。

### 外部の git リポジトリからプロジェクトをインポートする

Let's import the project with all the Data Objects relative to the Domain Model:

  1. `プロジェクトのインポート` をクリックします。
  2. ウィンドウが開いたら、 _リポジトリー URL_ に [https://github.com/kamorisan/rhpam-rhdm-workshop-v1m2-labs-step-2.git](https://github.com/kamorisan/rhpam-rhdm-workshop-v1m2-labs-step-2.git) と入力し、 `インポート` をクリックします。
  3. 次に、_プロジェクトのインポート_ 画面で、 _ccd-project_ を選択し, `OK` をクリックします。
    ![Business Central Delete CCD Project]({% image_path business-central-import-ccd-project.png %}){:width="800px"}

  4. インポートしたプロジェクトのデータオブジェクトの内容について、確認をして下さい。

おめでとうございます！
このセクションでは、データオブジェクトの定義方法を確認しました。
これでビジネスルールと意思決定の自動化に取り組むことができるようになります！

