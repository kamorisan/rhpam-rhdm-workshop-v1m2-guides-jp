
# 4. ビジネスルールとデシジョンの設計

このセクションでは、以下のことを学習します:

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

2. `新規ファクトパターンを作成`をクリックします。_ファクトタイプ_ として `CreditCardHolder` を選択し、_バインディング_ として `holder` という変数を定義します。`OK` をクリックして新規ファクトパターン作成ウィンドウを閉じ、`次へ` をクリックします。

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

    最後に、デシジョンテーブルは次のようになります。

    ​	![Business Central Decision Complete]({% image_path business-central-decision-table-complete.png %}){:width="800px"}

5. 入力が終わったら、テーブルを保存します。


## ガイド付きルール

ガイド付きルールは、Business Central で作成できるもう1つのタイプのルールです。
データオブジェクトを定義したら、そのオブジェクトのプロパティに対する条件をチェックするルールや、オブジェクトの組み合わせに対する条件を定義するルールなどを作成することができます。
例えば、クレジットカード所有者の年齢、カード種別（ステータス）、リスク評価などに制約条件を加えたルールを定義することができます。
条件が満たされた場合、ルールはマッチングしたとみなされ、起動の対象となります。
ルールが起動されると、ルールで定義されたアクションが実行されます。
アクションは、ルールの _THEN_ 部分、またはルールの結果、またはRight-Hand-Side (RHS)とも呼ばれるものです。

先ほど、リスク評価を決定するルールをデシジョンテーブルの形で作成しました。
ここでは、チャージバック申請が自動処理の対象になるかどうかを決定するルールを作成します。
このルールを _ガイド付きルール_ の形で作成します。

チャージバック申請の自動処理のルールの場合は、クレジットカード所有者のカード種別（ステータス）のみを評価しています。
そこで、ルールを作成してみましょう。

まず、どのようなオブジェクトやオブジェクトの集まりが評価されるかをルールに伝えます。ルールは非常に基本的な構文を持っており、基本的には3つの部分から構成されています。

- _When_ の部分はルールの制約条件を定義します。制約条件が満たされた場合、ルールが起動されます。
- _Then_ の部分は、ルールが起動したときに実行されるアクションを定義します。これは、例えば（データオブジェクトの）ファクトに特定のデータを設定することができますが、ルールエンジンに新しいデータを挿入することもできます。例えば、カード保持者の年齢に基づいて、その人が成人であることを推論することができます。
- プロパティまたは属性の部分。ここでは、ルールの追加の特性を設定します。例えば、ルールの属するグループなどです。

ルールの作成:

1. 画面上部のメインメニューの下にある _ccd-project_ をクリックすると、プロジェクトのホームページに戻ります。

    ![Business Central Breadcrumb bar ccd project]({% image_path business-central-breadcrumb-bar-ccd-project.png %}){:width="800px"}

2. ライブラリビューで、右上の青い `アセットの追加` ボタンをクリックします。

    ![Business Central CCD BOM Project]({% image_path business-central-ccd-bom-project.png %}){:width="800px"}

3. ドロップダウンのフィルタリングで `デシジョン` を選択し、デシジョンアセットをフィルタリングします。

    ![Business Central Add Assets Filter]({% image_path business-central-add-assets-filter.png %}){:width="800px"}

4. フィルタリングされたカタログの中から、`ガイド付きルール` を選択します。

5. 新規作成ウィンドウで以下のデータを設定し、`OK` をクリックします。

    - ガイド付きルール（Name）: `automated-chargeback`
    - パッケージ: `com.myspace.ccd_project`

    ![Business Central Guided Rule New]({% image_path business-central-guided-rule-new.png %}){:width="800px"}

6. アセットが作成されたことを示す緑色のバナーが表示され。ルールを作成するためのエディタが表示されます。

    ![Business Central Guided Rule New Wizard]({% image_path business-central-guided-rule-new-wizard.png %}){:width="800px"}

7. エディタパネルに4つのタブが表示されます。`データオブジェクト` と書かれたタブを選択します。

    ![Business Central Guided Rule Import Data Object]({% image_path business-central-guided-rule-import-data-object.png %}){:width="800px"}

8. `AdditionalInformation`, `CreditCardHolder`, `FraudData`, `Number` の4つの項目がリストアップされているはずです。ルールはこれらのデータオブジェクトと同じフォルダ/パッケージに作成されるため、デフォルトではこれらの項目が表示されます。もし `CreditCardHolder` がリストにない場合は、青い `新規アイテム` ボタンをクリックしてインポートします。

    ![Business Central Guided Rule Import Data Object New]({% image_path business-central-guided-rule-import-data-object-new.png %}){:width="800px"}

9.  `モデル` タブに戻り、_WHEN_ の右側にある緑色のプラス記号をクリックします。

    ![Business Central Guided Rule New Fact]({% image_path business-central-guided-rule-new-fact.png %}){:width="800px"}

10. オブジェクト `CreditCardHolder` を選択し、`OK`をクリックします。これで、CreditCardHolderが存在するたびにこのルールを有効にする必要があることをルールエンジンに伝えることができました。

    ![Business Central Guided Rule New Fact Select]({% image_path business-central-guided-rule-new-fact-select.png %}){:width="800px"}

    業務要件の基準を満たすためには、カード所持者のプロパティに制約条件を追加する必要があります。チャージバック申請の自動処理は、`status` が _Gold_ または _Platinum_ のカード所持者のみに適用されます。

11. `CreditCardHolder があります。` という条件をクリックすると、新しいウィザードが開きます。これでフィールドに制約条件を追加することができます。ドロップダウンボックスからカード所持者の `status` フィールドを選択します。

    ![Business Central Guided Rule New Property Select]({% image_path business-central-guided-rule-new-property-select.png %}){:width="800px"}

12. ドロップダウンボックスで、ステータス`はカンマ区切りのリストに含まれる` を選択します。鉛筆のアイコンをクリックして、固定値を選択し、 `Gold, Platinum` を追加します。`保存` ボタンをクリックして、アセットを保存します。

    ![Business Central Guided Rule New Property Select Values]({% image_path business-central-guided-rule-new-property-select-values.png %}){:width="800px"}

13. `データオブジェクト` タブに戻ります。まだ `FraudData` データオブジェクトがインポートされていない場合は、同じ手順でインポートします。`モデル` タブに戻り、先ほどと同じ方法で `FraudData` オブジェクトに制約を追加します。`FraudData` のプロパティに制約条件を加える必要はありません。代わりに、それが存在することを確認する必要があります。

    ![Business Central Guided Rule Check Fraud Data]({% image_path business-central-guided-rule-check-fraud-data.png %}){:width="800px"}

14. データオブジェクトやファクトのオブジェクト内のデータを変更したい場合、ルール内から一致したオブジェクトを参照できるようにする必要があります。これを可能にするには、オブジェクトをルール内の変数にバインドする必要があります。これにより、オブジェクトは、変数を通じて左側（LHS）と右側（RHS）の両方でアクセス可能になります。`FraudDataがあります。` をクリックすると、制約条件を変更するウィンドウが開きます。

15. フォームの下部にある `変数名` フィールドに、`FraudData` オブジェクトをバインドしたい変数の名前として `data` と入力します。`設定` ボタンをクリックします。その後、アセットを保存します。

    ![Business Central Guided Rule Modify Fraud Data]({% image_path business-central-guided-rule-modify-fraud-data.png %}){:width="800px"}
    
      これで、`FraudData` オブジェクトにチャージバック申請自動処理のプロパティをtrueに設定します。これはルールの決定であり、ルールの _action_ なので、これをTHEN句として定義します。（ルールのRight Hand Side (RHS)またはActionセクションとしても知られています。）
    
      チャージバック申請の情報はすべてファクトに保存されています。これらのファクトは、エンジンがメモリに保持するセッションに保存されます。したがって、新しいファクトを評価したり、既存のファクトに何かを変更したりするたびに、セッション内のすべてのオブジェクトが意思決定のプロセスで利用できるようになります。ルールのRHS（アクション）の部分では、変数を介して参照できるオブジェクトのプロパティの値を変更したり、新しいオブジェクト/ファクトを作成してセッションに追加したりすることができます（これは通常、新しいデータや情報の推論と呼ばれています）。オブジェクトのプロパティが変更されるたびに、このプロパティが使用されているすべての決定は、他のルールを適用する必要がないことを確認するために再評価されます。
    
16. _WHEN_ の右側にある緑色のプラス記号をクリックします。

       ![Business Central Guided Rule New Then Condition]({% image_path business-central-guided-rule-new-then-condition.png %}){:width="800px"}

17. `新しいアクションを追加` ウィンドウが開いたら、`dataのフィールド値を変更` を選択し、`OK` をクリックします。これで自動的に `FraudData` オブジェクトが選択されます。

       ![Business Central Guided Rule Modify Fraud Data Wizard]({% image_path business-central-guided-rule-modify-fraud-data-wizard.png %}){:width="800px"}

18. チャージバック申請自動処理が適用されることを示すプロパティ `automated` の値を `true` に設定していきます。アクション `FraudData[data]の値 設定` をクリックし、フィールド `automated` を選します。さらに右側の鉛筆アイコンをクリックして、`固定値` を選択します。

     ​	![Business Central Guided Rule Modify Fraud Automated]({% image_path business-central-guided-rule-modify-fraud-automated.png %}){:width="800px"}

19. `automated` プロパティの値として `true` を選択します (これはbooleanのデフォルト値なので、プロパティはおそらくすでに `true` に設定されています)。データの型は `boolean` なので、`true` と `false` の間でしか選択できないことに注意してください。
     ​	![Business Central Guided Rule Modify Fraud Automated True]({% image_path business-central-guided-rule-modify-fraud-automated-true.png %}){:width="800px"}

20. すべてが正しいことを確認するには、上部のナビゲーションバーにある `検証` ボタンをクリックします。問題がなければ、`アイテムは正常に検証されました！`という緑色のメッセージが表示されます。

     ​	![Business Central Guided Rule Validate]({% image_path business-central-guided-rule-validate.png %}){:width="800px"}

21. 最後に、`保存` をクリックしてルールを保存します。

ガイド付きエディタを使用して、最初のビジネスルールを作成しました。
<!-- // TODO Update to business central DMN editor -->

## Decision Model & Notation (DMN)

Red Hat Process Automation Manager 7 は、Decision Model & Notation (DMN) v1.2 標準をサポートしています。これは、DMN v1.1 または v1.2 仕様で作成されたモデルを RHPAM にインポートして実行できることを意味します。Red Hat Process Automation Manager と Red Hat Decision Manager の DMN エディタを使用するだけでなく、Business Central DMN エディタを使用して DMN モデルを作成したり、Trisotech社 の Digital Enterprise Suite のようなサードパーティ製のエディタを使用して RHPAM で実行することもできます。下記の画像は、リスクを算出するルールを定義するために作成できるダイアグラムの種類の例を示しています。

![Business Central Trisotech DMN]({% image_path business-central-dmn.png %}){:width="600px"}

DMNではFEEL（Friendly Enough Expression Language）と呼ばれる業務に親和性のある言語を使用しています。

![Business Central DMN FEEL]({% image_path business-central-dmn-feel.png %}){:width="800px"}

今のところ、DMNに関してはこのワークショップの対象外です。
しかし、DMNはRHPAMによってサポートされており、この仕様は、ビジネスアプリケーションで意思決定をモデル化して実行するための、もう一つの標準的な方法として提供されています。
