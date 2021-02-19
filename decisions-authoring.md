
# 4. ビジネスルールとデシジョンの設計

このセクションでは、以下のことを学習します。

1. Business Object Model を使って、意思決定や業務方針を自動化する方法
2. 様々な種類のオーサリングツールの使用方法
3. 自動化された判断やルールの使用方法

## 概要

チャージバック申請の承認は、いくつかの情報を元に判断をする必要があります。

- 顧客の情報
- チャージバック申請の金額

これらのルールや判断をどのように適用するかという知識は、暗黙のうちに他のドメインエキスパートの頭の中に存在するものになってしまっています。
プロセスを自動化するためには、まず、チャージバック申請の処理方法を決定する業務知識をルールという形で表現する必要があります。

このワークショップのケースでは、プロセス上の異なるステージに対して2つのルールが定義されています。

## リスクの計算

現状の業務においては、チャージバック申請の処理手続きはすべて手作業で実施しています。
チャージバックのデータに基づいて、意思決定を行う専門の部署ががあります。
これにはコストがかかることに加え、エラーや不整合が発生しやすいという問題があります。

Pecunia Corp. のチャージバック申請の処理には、チャージバック金額とは別に、高額の費用がかかっています。
だからこそ、処理コストと時間を削減する柔軟なルールを持つことが非常に重要なのです。
コストの削減以外にも、顧客体験の向上にもつながります。

プロセスのために定義されたルールは以下の通りです。

- Platinum、Gold のクレジットカードのお客様に対しては、チャージバック申請を自動承認する

- トランザクションのリスクを、お客様のカード種別とチャージバック申請の金額によって決定します

    - カード種別が **Standard** で、チャージバック申請金額が **100 未満** の場合、リスクは **low**
    - カード種別が **Standard** で、チャージバック申請金額が **100 以上、 500 未満** の場合、リスクは **medium**
    - カード種別が **Standard** で、チャージバック申請金額が **500 以上** の場合、リスクは **high**
    - カード種別が **Silver** で、チャージバック申請金額が **250 未満** の場合、リスクは **low**
    - カード種別が **Silver** で、チャージバック申請金額が **250 以上、 500 未満** の場合、リスクは **medium**
    - カード種別が **Silver** で、チャージバック申請金額が **500 以上** の場合、リスクは **high**
    - カード種別が **Gold** で、チャージバック申請金額が **500 未満** の場合、リスクは **low**
    - カード種別が **Gold** で、チャージバック申請金額が **500 以上** の場合、リスクは **medium**

## オーサリングツール

前のセクションで Business Object Model を定義しました。
前のセクションの最後のステップでは、完全な Business Object Model を持つプロジェクトをインポートしました。
このプロジェクトをこのラボのベースとして使用します。

### デシジョンテーブル

#### デシジョンテーブルの作成

リスクの算出の背後にあるロジックを定義する非常に一般的な方法は、この情報をスプレッドシートに保存することです。
Red Hat Process Automation Manager を使用すると、同じスプレッドシートのアプローチを使用して、エンジン内で実行可能な資産 (すなわち、ルールのセット) にすることができます。
このセクションでは、リスク算出のルールを自動化するために _デシジョンテーブル_ を作成します。

1. まず、ライブラリビューに戻り、右上の青い `アセットの追加` ボタンをクリックします。

    ![Business Central Decision Table Add Asset]({% image_path business-central-decision-table-add-asset.png %}){:width="800px"}

2. アセットのカタログから`ガイド付きデシジョンテーブル`を選択します。 <br> 
(画面左上のフィルタと入力ボックスを利用して、タイプ別にフィルタリングすることができます。デシジョンアセットをフィルタリングするには、`デシジョン`を選択します)

    ![Business Central Decision Table Add Asset Guided]({% image_path business-central-decision-table-add-asset-guided.png %}){:width="800px"}

1. `新規作成 ガイド付きデシジョンテーブル` のウィンドウで、以下の値を入力します。

    - ガイド付きデシジョンテーブル (Name): `risk-evaluation`
    - パッケージ: `com.myspace.ccd_project`

    それ以外の情報はそのままにして、 _OK_ をクリックします。

2. `ガイド付きデシジョンテーブル` エディタに空のテーブルが表示されているはずです。

    ![Business Central Decision Table New]({% image_path business-central-decision-table-new.png %}){:width="800px"}

    エディタには5つのタブがあります:

    - _モデル_: デシジョンテーブルのモデル図
    - _列_: テーブルに列を追加、編集、または削除します。カラムは、属性、メタデータ、 Business Object Model のプロパティ上の制約（ルール左側またはLHS）、またはアクション（ルール右側またはRHS）を表すことができます。
    - _概要_: アセットのメタ情報（バージョン、説明、最終更新日など）が含まれます。
    - _ソース_: デシジョンテーブルモデルから生成される実際のソースコードです。ルールエンジンでは、デシジョンテーブルhzDRL（Drools Rule Language）に変換され、テーブルの各行はルールに変換されます。
    - _データオブジェクト_: エディタで条件やアクションとして使用できるデータオブジェクトをリストアップします。

今回のシステムにおいて、リスク算出をするための必要な情報は、以下の通りです。

    - 顧客のクレジットカードの種別
    - クレジットカード不正利用の対象となっている金額の合計

#### デシジョンテーブルの列の設定

`Credit Card Holder` の条件列を追加していきます。

1. `列`タブに移動し、`Insert Column`ボタンをクリックし、`条件を追加`を選択して`次へ`をクリックします。

    ![Business Central Add Condition]({% image_path business-central-add-condition.png %}){:width="800px"}

2. `新規ファクトパターンを作成`をクリックします。_ファクトタイプ_ として `CreditCardHolder` を選択し、_バインディング_ として `holder` という変数を定義します。`OK` をクリックして新規ファクトパターン作成ウィンドウを閉じ、`Next` をクリックします。

    ![Business Central Create Pattern]({% image_path business-central-create-pattern.png %}){:width="800px"}

3. 計算タイプは、適用する評価のタイプです。この場合は固定値に対する評価になります。`固定値` を選択し、`次へ` をクリックします。

4. カード所持者のステータス（カード種別）に条件を定義する必要があるので、`status`フィールドを選択して、`次へ` をクリックします。

    ![Business Central Create Pattern Field]({% image_path business-central-create-pattern-field.png %}){:width="800px"}

5. 次に、オペレーター（制約の演算子）を選択します。ドロップダウンメニューから `は次の値と等しい` を選択し、`次へ` をクリックします。

    ![Business Central Create Pattern Field Operator]({% image_path business-central-create-pattern-field-operator.png %}){:width="800px"}

6.  このデシジョンテーブルで扱うステータスは3つしかありませんので、以下の値でフィールドを設定し、`次へ` をクリックします。

    - 値リスト (オプション): `Standard,Silver,Gold`
    - デフォルト値: `Standard`

    ![Business Central Create Pattern Field Values]({% image_path business-central-create-pattern-field-values.png %}){:width="800px"}

7.  ヘッダー（説明）: `Status` と入力してください。

    ![Business Central Create Pattern Field Header]({% image_path business-central-create-pattern-field-header.png %}){:width="800px"}

8.  `完了` をクリックし、エディタの `Model` タブを選択します。新しく作成された列が表示されているはずです。

9.  同じ手順を繰り返して、さらに2つの列を追加します。

    - パターン:
        - ファクトタイプ: `FraudData`
        - バインディング: `data`
    - 計算タイプ: `固定値`
    - フィールド: `totalFraudAmount`
    - 値のオプション: 空欄
    - オペレーター:
        - 1つ目の列: `は次の値以上` 、ヘッダー（説明）: `Minimum Amount`
        - 2つ目の列: `は次の値よりも小さい` 、ヘッダー（説明）: `Maximum Amount`

    2つ目の列については、新しいファクト・パターンを作成する必要はなく、既存のパターンを再利用できることに注意してください。

    最後に、このデシジョンテーブルは次のようになります。

    ![Business Central Decision Table Columns]({% image_path business-central-decision-table-columns.png %}){:width="800px"}

10. `保存` をクリックし、デシジョンテーブルを保存します。

11. 次に `列` タブに移動し、`Insert Column` をクリックします。今回はルールの右側にアクション列を追加します。このアクションは条件が満たされたときに実行されます。`フィールド値をセット`を選択して、`次へ`をクリックします。

12. `FraudData` オブジェクトのリスク評価を設定します。パターンのドロップダウンメニューで、変数 `data` にバインドされたオブジェクト `FraudData` を選択します。

    ![Business Central Decision Table Columns Action Data]({% image_path business-central-decision-table-columns-action-data.png %}){:width="800px"}

13. フィールド `disputeRiskRating` を選択し、`次へ`をクリックします。値のリストは空欄で、そのまま `次へ` をクリックします。列のヘッダー（説明）に `Risk Scoring` と入力し、`完了`をクリックします。

    ![Business Central Decision Table Columns Action Data Finish]({% image_path business-central-decision-table-columns-action-data-finish.png %}){:width="800px"}

14. `保存` をクリックし、デシジョンテーブルを保存します。

#### ビジネスルールの書き方

1. `モデル`タブに戻ります。ここで実際の制約条件とアクション、つまり実際のルールを追加します。業務要件を見ると、最初の制約条件は次のように定義されています。

    _カード種別が **Standard** で、チャージバック申請金額が **100 未満** の場合、リスクは **low**_

    リスクには、low, medium, high, very-high の4つのレベルがあります。これらのリスクレベルを0,1,2,3の整数で定義します。

2. `挿入` ボタンをクリックし、ドロップダウンメニューから`行の追加`を選択します。

     ![Business Central Decision Table Columns Action Data Finish Model]({% image_path business-central-decision-table-append-row.png %}){:width="800px"}

3. 新しい行の `説明` セルをクリックして、`Standard customer low risk` と入力します。他の列には以下の値を使用します。

     - 説明:`Standard customer low risk`{{copy}}  
     - Status:`Standard`{{copy}}  
     - Minimum Amount:`0`{{copy}}  
     - Max Amount:`100`{{copy}}  
     - Risk Scoring:`0`{{copy}}

    デシジョンテーブルは以下のようになります。`保存` をクリックします。

    ![Business Central Decision Table First Row]({% image_path business-central-decision-table-first-row.png %}){:width="800px"}

4. 業務要件に従い、それ以外の部分についても同様の手順を適用します。

    - Standard 100未満: low risk (Risk scoring = 0)
    - Standard 100-500: medium risk (Risk scoring = 1)
    - Standard 500以上: high risk (Risk scoring = 2)
    - Silver 250未満: low risk (Risk scoring = 0)
    - Silver 250-500: medium risk (Risk scoring = 1)
    - Silver 500以上: high risk (Risk scoring = 2)
    - Gold 500未満: low risk (Risk scoring = 0)
    - Gold 500以上: medium risk (Risk scoring = 1)

    最後に、あなたのデシジョンテーブルは次のようになります。

    ​	![Business Central Decision Complete]({% image_path business-central-decision-table-complete.png %}){:width="800px"}

5. 入力が終わったら、テーブルを保存します。


## Guided Rules

Guided Rules are another type of rules that you can create in Business Central. Once you have defined the Business Object Model, you can create rules that check conditions on the properties of these objects, rules that define conditions on combinations of objects, etc. For example, you can define a rule with a constraint on a Credit Card Holder's age, his/her status, riskRating, etc. If the condition or conditions are met, the rule is set to be _matched_ and becomes eligible for _firing_. When the rule fires, it executes the action defined in the rule. The action is the _THEN_ part of the rule, or what is also called the rule's consequence, or Right-Hand-Side (RHS).

In the previous section you have created the rules, in the form a decision table, that determine the credit risk scoring. In this section, you create the rules that determine whether the credit card dispute is eligible for automatic chargeback. You will create this rule in the form of _Guided Rule_.

In the case of the rules for automatic chargeback you are evaluating only the Credit Card Holder. So let's create the rule.

First, tell the rule what object or collection of objects is going to be evaluated. Rules have a very basic syntax, and basically consist of 3 parts:

- the _When_ part defines the constraints of the rule. I.e. these are the discrimination criteria or conditions which, if they are met, cause the rule to fire.
- the _Then_ part defines the actions the rule will execute when it fires. This can be for example setting specific data on a fact (in the Business Object Model), but this can also be inserting new, inferred, data into the rules engine. For example, based on a card holder's age, you can infer that he/she is an adult.
- the _properties_ or _attributes_ part. Here you set additional characteristics of the rule, for example the group of rules it belongs to.

To create the rule:

1. At the top of the screen under the main heading, click the _ccd-project_ to bring you back to the homepage for the project

    ![Business Central Breadcrumb bar ccd project]({% image_path business-central-breadcrumb-bar-ccd-project.png %}){:width="800px"}

2. Click on the blue button `Add Asset` on the right upper corner of the Library View.

    ![Business Central CCD BOM Project]({% image_path business-central-ccd-bom-project.png %}){:width="800px"}

3. In the _Add Asset_ screen, select _Decision_ from the drop-down filter menu to filter on decision assets.

    ![Business Central Add Assets Filter]({% image_path business-central-add-assets-filter.png %}){:width="800px"}

4. Select `Guided Rule` from the filtered catalog of Wizards.

5. Set the following data in the creation wizard:

    - Name: `automated-chargeback`
    - Package: `com.myspace.ccd_project`

    ![Business Central Guided Rule New]({% image_path business-central-guided-rule-new.png %}){:width="800px"}

6. Click ok. You should see a banner in green telling you that the asset was success fully created. The UI will display the editor that allows you to author your rule.

    ![Business Central Guided Rule New Wizard]({% image_path business-central-guided-rule-new-wizard.png %}){:width="800px"}

7. You will see 4 tabs in the editor panel. Select the tab that says "Data Objects"

    ![Business Central Guided Rule Import Data Object]({% image_path business-central-guided-rule-import-data-object.png %}){:width="800px"}

8. You should see 4 items listed: `AdditionalInformation`, `CreditCardHolder`, `FraudData`, and `Number`. These are shown by default as the rule is created in the same folder/package as these data objects. If `CreditCardHolder` is not listed, click on the blue _New Item_ button to import it.

    ![Business Central Guided Rule Import Data Object New]({% image_path business-central-guided-rule-import-data-object-new.png %}){:width="800px"}

9. Return to the _Model_ tab and Click on the green plus-sign to the right of the word _WHEN_.

    ![Business Central Guided Rule New Fact]({% image_path business-central-guided-rule-new-fact.png %}){:width="800px"}

10. Select the object `CreditCardHolder`, and click ok. You are now telling the rule engine that every time there is a CreditCardHolder this rule needs to be activated.

    ![Business Central Guided Rule New Fact Select]({% image_path business-central-guided-rule-new-fact-select.png %}){:width="800px"}

    In order to match the criteria of the functional requirement, you need to add a restriction on one of the card holder's properties. Automated chargeback is only approved for CC Holders that have the `status` _Gold_ or _Platinum_.

11. Click on the condition `There is a Credit Card Holder`, a new wizard will open. You now _Add a restriction on a field_. From the dropdown box select the `status` field of the CC Holder.

    ![Business Central Guided Rule New Property Select]({% image_path business-central-guided-rule-new-property-select.png %}){:width="800px"}

12. From the dropdown box select that the status `is contained in the (comma separated) list`. Click on the pencil icon and add the literal values  _Gold_ and _Platinum_, separated by a comma. TIP: You can also add enumerations containing these values to have them pre-populated for you. Click on the _Save_ button to save the asset.

    ![Business Central Guided Rule New Property Select Values]({% image_path business-central-guided-rule-new-property-select-values.png %}){:width="800px"}

13. Go back to the _Data Objects_ tab. If the `FraudData` data object has not been imported yet, complete the same procedure, to import it. Go back to the _Model_ tab and add a constraint on the `FraudData` object the same way as you did before. You don't need to put a constraint on any property of the `FraudData`. Instead, you just need to make sure that it's there.

    ![Business Central Guided Rule Check Fraud Data]({% image_path business-central-guided-rule-check-fraud-data.png %}){:width="800px"}

14. When you want to modify the data in the objects of the Business Model or facts, you need to be able to reference the matched object from within the rule. To allow this, the object needs to be bound to a variable inside the rule. This makes the object accessible in both the left-hand-side (LHS) and  right-hand-side (RHS) through the variable. Click on the fact declaration `There is FraudData`, the wizard to modify the constraints will open.

15. In the _Variable name_ field at the bottom of the form, type `data` as the name of the variable that you want to bind the `FraudData` object to. Click on the _Set_ button. Save the asset.

    ![Business Central Guided Rule Modify Fraud Data]({% image_path business-central-guided-rule-modify-fraud-data.png %}){:width="800px"}
    
      Now set the property of automated chargeback to true on the `FraudData` object, so the dispute can be processed accordingly. Since this is the decision you are making, and thus the _action_ of the rule, you define this as the THEN clause,  also known as the Right Hand Side (RHS) or Action section of our rule.
    
      All of the information of the CC dispute is stored in facts. These facts can live in a session that the engine will keep in memory. So every time you evaluate a new fact, or change something to an existing fact, you will have all of the Objects in the session available in the process of decision making. In the RHS, or action, part of the rule you can change the values of any property on the objects that you can reference via the variables, or even create and add new objects/facts to the session (this is usually referred to as _inferring_ new data or information). Every time a property in an object changes, all of the decisions in which this property is used will be reevaluated to make sure that no other rule needs to be applied.
    
16. Click on the green plus-sign next to the _THEN_ keyword.

       ![Business Central Guided Rule New Then Condition]({% image_path business-central-guided-rule-new-then-condition.png %}){:width="800px"}

17. When the `Add new action` wizard opens select `Change field values of data` and click on _OK_. This will automatically select the `FraudData` object, as this is the only object that has been bound to a variable.

       ![Business Central Guided Rule Modify Fraud Data Wizard]({% image_path business-central-guided-rule-modify-fraud-data-wizard.png %}){:width="800px"}

18. Now set the value of the property `automated` to `true`, indicating that an automatic chargeback applies. Click on the action `Set value of FraudData [data]` and select the field `automated`. Click on the pencil icon to the right and assign a literal value to the property.

     ​	![Business Central Guided Rule Modify Fraud Automated]({% image_path business-central-guided-rule-modify-fraud-automated.png %}){:width="800px"}

19. Select `true` as the value for the automated property (this is the default value for booleans, so the property is probably already set to `true`). Note that since the type of data is `boolean`, you can only choose between `true` and `false`.

     ​	![Business Central Guided Rule Modify Fraud Automated True]({% image_path business-central-guided-rule-modify-fraud-automated-true.png %}){:width="800px"}

20. To validate that everything is correct, click on the _Validate_ button on the top navigation bar and you should see a green "Item successfully validated!" message.

     ​	![Business Central Guided Rule Validate]({% image_path business-central-guided-rule-validate.png %}){:width="800px"}

21. Finally, click on _Save_ to save the rule.

You have created your first Business Rule using the Guided editor
<!-- // TODO Update to business central DMN editor -->

## Decision Model & Notation (DMN)

Red Hat Process Automation Manager 7 supports the Decision Model & Notation (DMN) v1.2 standard. This means that models created in the DMN v1.1 or v1.2 specification can be imported into, and executed on, RHPAM. Apart from using Red Hat Process Automation Manager's and Red Hat Decision Manager's DMN editor, this also allows users to create DMN models using Business Central DMN editor, or even third-party editors like for example Trisotech's Digital Enterprise Suite, and execute them in RHPAM. In the following image you can see some examples of the types of diagrams you can create to define, in this case, the rules to calculate risk.

![Business Central Trisotech DMN]({% image_path business-central-dmn.png %}){:width="600px"}

DMN uses a business friendly language called FEEL or Friendly Enough Expression Language.

![Business Central DMN FEEL]({% image_path business-central-dmn-feel.png %}){:width="800px"}

For now, DMN is out of scope for this workshop. However, it is supported by RHPAM and the specification provides an additional, interesting, and standard way to model and execute decisions in your business applications.
