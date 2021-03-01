Red Hat Process Automation Manager / Decision Manager Workshop Module 2
===
Red Hat Process Automation Manager と Red Hat Decision Manager の ハンズオンワークショップの `モジュール 2` です。
このワークショップでは、開発者や業務エキスパートに、最新のクラウドネイティブアーキテクチャのコンテキストでのルールおよびプロセス駆動型アプリケーションとマイクロサービスの紹介を行います。

Agenda
===
* イントロダクション
* ユースケースの紹介
* Business Object Model
* ビジネスルールとデシジョンの設計
* デシジョンサービス
* デシジョンサービスのテスト
* デシジョンサービスの実行
* 終わりに

Run locally
=== 

```
$ git clone https://github.com/kamorisan/rhpam-rhdm-workshop-v1m2-guides-jp.git
$ cd rhpam-rhdm-workshop-v1m2-guides-jp
$ docker run -it --rm -p 8080:8080 -v $(pwd):/app-data -e CONTENT_URL_PREFIX="file:///app-data" -e WORKSHOPS_URLS="file:///app-data/_rhpam-rhdm-workshop-module2.yml" -e LOG_TO_STDOUT=true quay.io/osevg/workshopper 
```

Lab Instructions on OpenShift
===

APB経由で labs-infra をインストールした場合、ラボインストラクションはすでにデプロイされていることに注意してください。

ここでは、ラボインストラクションを OpenShift クラスタに手動でデプロイするための Ansible プレイブックの例を示します。
```
- name: Create Guides Module 2
  hosts: localhost
  tasks:
  - import_role:
      name: siamaksade.openshift_workshopper
    vars:
      project_name: "guide-m2"
      workshopper_name: "RHPAM / RHDM Workshop V1 Module-2"
      project_suffix: "-XX"
      workshopper_content_url_prefix: https://raw.githubusercontent.com/kamorisan/rhpam-rhdm-workshop-v1m2-guides-jp/main
      workshopper_workshop_urls: https://raw.githubusercontent.com/kamorisan/rhpam-rhdm-workshop-v1m2-guides-jp/main/_rhpam-rhdm-workshop-module2.yml
      workshopper_env_vars:
        PROJECT_SUFFIX: "-XX"
        COOLSTORE_PROJECT: coolstore{{ project_suffix }}
        OPENSHIFT_CONSOLE_URL: "https://master.seoul-2922.openshiftworkshop.com"
        ECLIPSE_CHE_URL: "http://che-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        GIT_URL: "http://gogs-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        NEXUS_URL: "http://nexus-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        LABS_DOWNLOAD_URL: "http://gogs-labs-infra.apps.seoul-2922.openshiftworkshop.com"

      openshift_cli: "/Users/doh/cloud-native-app-dev/oc --server https://master.seoul-2922.openshiftworkshop.com"
```
