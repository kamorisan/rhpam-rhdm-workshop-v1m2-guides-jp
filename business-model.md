# 3. Business Object Model

このセクションでは、以下のことを学習します。

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

5. Select _Design_ from the main menu. You will be redirected to your working space. You will see a list of _Spaces_, with a single space named _MySpace_.
6. メインメニューから _設計_ を選択します。ワークスペースにリダイレクトされます。スペースのリストが表示され、MySpaceという名前の1つのスペースが表示されます。

7. Click on _MySpace_. This is the sandbox in which you'll define your projects, and within those projects, your projects assets.

8. Since this project is going to be used to deliver a case management implementation, we need to add a new `Case Project`. In order to do so, click on the arrow right next to the _Add Project_ button and select the option `Case Project`.

    ![Business Central Asset CCD Project]({% image_path add-new-case-project.png %}){:width="800px"}

9. When the _Add Project_ wizard opens up, type in

      *  `ccd-project` as the name of the project, and
      * `Credit card dispute business automation project` as the description of your project.

    This project is your business boundary that encapsulates your business capability. Once the creation of your project is complete, you will see it in your space:

    ![Business Central Asset CCD Project]({% image_path business-central-asset-ccd-project.png %}){:width="800px"}

10. Select the `ccd-project`. You should see the following page with the project content.

    ![Business Central Asset Empty Project]({% image_path business-central-asset-empty-project.png %}){:width="800px"}

11. Notice there's a blue button called `Add Asset`.  Click on the `Add Asset` button and you will be presented with a catalog of the wizards to create assets.An asset is a business resource of the project like Rules, Processes, Decision Tables, Data Objects, Data Forms, etc._

    ![Business Central Asset Catalog]({% image_path business-central-asset-catalog.png %}){:width="800px"}

12. Select the wizard for _Data Object_ from the catalog to create your business object model for the _Credit Card Holder_ enter the following and Click OK:

    * type `CreditCardHolder`{{copy}} as the name of the object sandbox
    * select `com.myspace.ccd_project` as the Package.

    ![Business Central CCD Object]({% image_path business-central-CCD-object-new.png %}){:width="800px"}

13. You will see the new created object with no properties, lets click on the `+add field` button to start adding the properties to our CreditCardHolder data object.

    ![Business Central CCD Object New Empty]({% image_path business-central-CCD-object-new-empty.png %}){:width="800px"}

14. In the _New Field_ window, enter the following values and click on _Create_:

    - Id: `age`
    - Label: `Age`
    - Type: `Integer`

    ![Business Central CCD Object New Properties]({% image_path business-central-CCD-object-new-properties.png %}){:width="800px"}

The first step to automate a process or decision is to define and specify the Business Object Model. In this exercise you've created the Entity `CreditCardHolder` and defined it's `age` field. These entities will be used to store the information that you need to make decisions and drive process execution.

## Managing projects in Business Central

We learned how easy it is to create a new project and new data objects. Let's now import an existing project with more Business Object Models.

In order to do this, let's delete the `ccd-project` project and learn how to import an existing project from a git repository.

### Deleting the ccd-project project

  1. Delete the current project
  2. At the top of the screen under the main heading, click the _ccd-project_ to bring you back to the homepage for the project
    ![Business Central Breadcrumb bar ccd project]({% image_path business-central-breadcrumb-bar-ccd-project.png %}){:width="800px"}

  3. Delete the project by clicking the hamburger menu & selecting _Delete Project_
    ![Business Central Delete CCD Project]({% image_path business-central-delete-ccd-project.png %}){:width="800px"}

  4. Type in _ccd-project_ and click `Delete Project`
  5. If asked you can `Discard unsaved changes and proceed`.

### Importing a project from external git repository

Let's import the project with all the Data Objects relative to the Domain Model:

  1. Click the `Import Project` button;
  2. On the pop-up, enter [https://github.com/RedHat-Middleware-Workshops/rhpam-rhdm-workshop-v1m2-labs-step-2.git](https://github.com/RedHat-Middleware-Workshops/rhpam-rhdm-workshop-v1m2-labs-step-2.git) as the _Repository URL_ and click `Import`
  3. On the _Import Projects_ screen, select the _ccd-project_ and click `Ok`
    ![Business Central Delete CCD Project]({% image_path business-central-import-ccd-project.png %}){:width="800px"}

  4. Examine the other newly-imported entities

Congratulations, now you that we've seen how to define Data Objects, we can now start working with the automation of our business rules and decisions!
