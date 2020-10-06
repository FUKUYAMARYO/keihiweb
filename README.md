# KeihiWeb

## 必須
 - Java 11
 - Docker Desktop　https://www.remotemeeting.com/reservation/share/2c908ad674c41ef80174f75dd3d8254d
 
## データベースの接続情報設定
Docker で作成される MySQL サーバー および JDBC に用いる情報の設定を行います。

i. `KeihiWeb/pom.xml` を開きます。以下の情報に従い、設定を行ってください。  

| 項目        | 説明              |
|-------------|--------------------------|
| db.name     | 使用するスキーマ（データベース）名       |
| db.host     | データベースのホスト名       |
| db.port     | データベースのポート番号           |
| db.username | データベースのログインユーザー名 |
| db.password | データベースのパスワード |

## MySQL サーバー構築
i. ご自身の環境を確認してください。
 - オペレーションシステム
   - macOS
   - Windows 10 Pro
   - Windows 10 Home※
 - Docker Desktop がインストールされている

※WSL2 (Ubuntu) をインストールし、開発者モードの場合 Docker Desktop がご利用できます。

ii. 以下のコマンドでデータベースをセットアップします。
```
# Windows
mvnw -Drun=init-container

# macOS or Git Bash
./mvnw -Drun=init-container
```

iii. STATUS が `healthy` になるまで待機します。確認コマンドは以下の通りです。
```
docker ps
```
 
## 一括セットアップ
i. ご自身の環境に合わせて `KeihiWeb/pom.xml` を編集してください。  

ii. 以下のコマンドで、次項以降のセットアップを一括で行います。
また、依存関係のダウンロードやデータベースのテーブル構築も行います。
```
# Windows
mvnw

# macOS or Git Bash
./mvnw
```

それでは、それぞれの個別セットアップの方法を記載します。

## テーブルの初期化
i. 以下のコマンドで、テーブルの初期化※を行います。  
テーブル定義の SQL を変更した場合にも、以下のコマンドを実行すると即座に反映されます。
```
# Windows
mvnw -f KeihiMigration -P migrate

# macOS or Git Bash
./mvnw -f KeihiMigration -P migrate
```

***※テーブルの初期化時に実行される SQL について***  
`KeihiMigration/src/main/resources/db/migration` 配下に [Flyway の仕様](https://flywaydb.org/documentation/command/migrate) に基づいて配置されている SQL が実行されます。

## エンティティのリバースエンジニアリング
i. 以下のコマンドで、データベースにアクセスし、テーブル定義に基づいたエンティティ※を自動生成します。
```
# Windows
mvnw -f KeihiEntity -P reveng

# macOS or Git Bash
./mvnw -f KeihiEntity -P reveng
```

ii. `KeihiWeb/KeihiEntity/target/generated-sources/mybatis-generator` にエンティティが作成されていることを確認します。  

iii. 特に何もせず、作成されたエンティティは Java 上で参照できます。

***※テーブル定義に基づいたエンティティ について***  
エンティティを生成するテーブルは明示的に記述する必要があります。  

`KeihiEntity/src/mybatis/resources/generatorConfig.xml` に以下の記述を追記します。
```
<table schema="${db.name}" tableName="エンティティ化するテーブル名"/>
```

## インストール
```
# Windows
mvnw install -Dmaven.test.skip

# macOS or Git Bash
./mvnw install -Dmaven.test.skip
```

## Spring アプリケーションの起動
### デバッグする場合
`CommonWeb/src/main/java/keihiweb/CommonWebApplication` の main メソッドを IDE 上で実行します。
  
### 通常起動する場合
以下コマンドを実行します。
```
# Windows
mvnw -f CommonWeb spring-boot:run

# macOS or Git Bash
./mvnw -f CommonWeb spring-boot:run
```

## ユースケース
### 経費申請
i. [http://localhost:8080/expense/](http://localhost:8080/expense/) にアクセスします。  
ii. ログインします。以下の社員が初期登録されています。

| 社員 ID | パスワード       | 役割      |
|-------------|----------------| --------- |
| 0           | demo           | 管理者   |
| 1           | demo           | ユーザー      |

iii. 以下のユースケースに従い操作を行います。
```
・画面①：経費登録画面  
　　＜社員用＞  
　　画面A：ログイン画面  
　　　⇒社員番号でログインできる 

　　画面B：経費申請画面  
　　　⇒利用日、利用者、申請者（自分）、利用科目（何の経費か） 

　　画面C：経費一覧画面  
　　　⇒自分の申請した経費のみの一覧が見れる。  
　　　　承認状況もステータス表示される。  
　　　　利用月を指定して表示される。 
```

### 管理
i. [http://localhost:8080/manage/](http://localhost:8080/manage/) にアクセスします。  
ii. ログインします。以下の管理者が初期登録されています。

| 社員 ID | パスワード       | 役割      |
|-------------|----------------| --------- |
| 0           | demo           | 管理者   |

iii. 以下のユースケースに従い操作を行います。
```
・画面②：経費管理画面  
　　＜管理部用＞  
　　画面A：ログイン画面  
　　　⇒管理部の利用ユーザIDでログインできる  

　　画面B：社員管理画面  
　　　⇒社員を登録・編集・削除できる。  
　　　　ここで登録した社員が画面①を利用できる。  

　　画面C：社員経費一覧画面  
　　　⇒画面①で申請された経費が一覧表示できる。  
　　　　利用月を指定して表示される。  
　　　　全社員の経費が表示される。  
　　　　アクションD（経費承認・否決）がボタンアクションで実行できる。  
　　　　検索フィルターがかけられる（月フィルター、社員フィルター）  

　　アクションD：経費承認・否決  
　　　⇒画面②-C（社員経費一覧画面）から、承認・否決ができる。  
```

