# flutterでのPostgreSQL利用手順

## 前提

- 「②PostgreSQLの使い方」を終了している事

## ①flutterプロジェクト作成

プロジェクトを作成します。

PostgreSQLとの接続の確認をするだけなので以下に雛形のコードをコピーして`main.dart`へ貼り付けてください。

このコードで主に編集するのは60行目付近にある  
ElevatedButtonのonPressedプロパティの{}の中だけです。  
お手数ですが１ページ目のコードをコピーして貼り付け後、  
続けて２ページ目のコードも続いてコピペして下さい
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});
  @override
  State<MyHomePage> createState() => _MyHomePageState();
}
class _MyHomePageState extends State<MyHomePage> {
  final TextEditingController _firstNameController = TextEditingController();
  final TextEditingController _lastNameController = TextEditingController();
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              SizedBox(
                height: 40,
                width: 150,
                child: TextField(
                  controller: _firstNameController,
                  decoration: InputDecoration(
                    labelText: 'first name',
                    border: OutlineInputBorder(),
                  ),),),
              const SizedBox(height: 10,),
              SizedBox(
                height: 40,
                width: 150,
                child: TextField(controller: _lastNameController,
                  decoration: InputDecoration(
                    labelText: 'last name',
                    border: OutlineInputBorder(),
                  ),), ),
              ElevatedButton(
                  onPressed: () {
                    // 以下に実行文を記述してください
                  },
                  child: const Text('data send')),
              ElevatedButton(
                  onPressed: () {
                    // 以下に実行文を記述してください
                  },
                  child: const Text('data get'))
            ],
          ),
        ));
  }
}

```

コピーができたら一旦ビルドをしてみましょう。  
以下のような画像になればここまではOKです。

<img src="image/スクリーンショット 2023-09-20 11.14.53.png" width="38%">

## ②PostgreSQL用のパッケージ追加

一旦、pgAdmin4を開いてください。

PostgreSQLの使い方の説明のところで以下のテーブルを作成したかと思います。

前回講義で作成した「test_db」を選択して「Query Tool」で以下のコードを実行して下さい。

```sql
SELECT * FROM human;
```

以下のような出力がされていればOKです。

<img src="スクリーンショット 2023-10-17 13.34.47.png" width="50%">

ここからflutterのテキストフィールドに入力した文字を追加するアプリを作っていきます。

ではflutterで使えるようにしていくのでAndroid Studioを開いてください。

先ほど作成したプロジェクトを開いてAndroid Studioのターミナルを開きます。

<img src="image/スクリーンショット 2023-09-14 11.57.11.png" width="50%">


そこのターミナルに以下のコマンドを入力して下さい。

```bash
flutter pub add postgres
```

このコマンドはflutterで使用するパッケージを追加するコマンドです。

flutter pub add postgres は、Flutterアプリケーションに  
PostgreSQLデータベースを操作するためのパッケージを追加するためのコマンドです。  

以下で、このコマンドの各部分について説明します。

flutter → これはFlutterコマンドラインツールの名前です。  

pub　→ pub は、Flutterアプリケーションの依存関係を管理するためのパッケージマネージャーです。  
Flutterアプリケーションには、さまざまなパッケージ（ライブラリやツール）を追加できます。

add　→ pub コマンドのサブコマンドの1つで、新しいパッケージをアプリケーションに追加します。

postgres　→ これは追加しようとしているパッケージの名前です。  
具体的には、`PostgreSQLデータベースとの通信をサポートする`Flutter用のパッケージです。

## ③PostgreSQLへデータ送信

ではflutterで扱うPostreSQLのコードを記述していきます。  

コードは[dart packageの公式サイト](https://pub.dev/packages/postgres)を参考にしています。  

参考にして少し改変したコードが以下になります。

60行目付近の以下の所に以下のコードを貼り付けて下さい。 

```dart
              ElevatedButton(
                  onPressed: () {
                    // ここにコードを貼り付ける
                  },
                  child: const Text('data send')),
```

以下のコードではパスワードのところをインストール時に設定したパスワードに  
置き換える必要がるので忘れずに書き換えるようにして下さい。  

```dart

                    final connection = PostgreSQLConnection(
                      'localhost', // ホスト名
                      5432, // ポート番号
                      'test_db', // データベース名
                      username: 'postgres', // ユーザー名
                      password: '!!!!!your_password!!!!!',
                    );

                    try {
                      await connection.open();
                      final insertResult = await connection.execute(
                        'INSERT INTO human (first_name, last_name) VALUES (@first_name, @last_name)',
                        substitutionValues: {
                          'first_name': firstName,
                          'last_name': lastName,
                        },
                      );

                      if (insertResult > 0) {
                        print('success!');
                      } else {
                        print('failed...');
                      }
                    } catch (e) {
                      print('Error: $e');
                    } finally {
                      await connection.close();
                    }
```

以上のコードを貼り付けると赤線が表示されます。

エラーを解消していきます。  

※注意点  
コピペの時に人によっては表示崩れが起きるかもしれません。  

もし以下の画像のようになっていれば  
<img src="image/スクリーンショット 2023-09-20 10.56.03.png" width="50%">

下のように改行を無くすようにお願いします。  

<img src="image/スクリーンショット 2023-09-20 10.56.17.png" width="50%">

以下のコードの「PostgreSQLConnection」に赤線があるかと思います。

```dart
final connection = PostgreSQLConnection(
```

PostgreSQLConnectionの所をクリックしてから

「option」＋「Enter（Return）キー」を押します。

すると以下のような画像のようにimportをする画面が出るのでクリックで選択します。

これは先ほど「flutter pub add postgres」でインストールした
パッケージを読み込むために行っています。

<img src="image/スクリーンショット 2023-09-14 13.16.23.png" width="50%">

まだ他にも「await」の所でエラーが出ていると思います。

以下のように「async」を追加します。

```diff
- ElevatedButton(
-               onPressed: () {
+ ElevatedButton(
+               onPressed: () async {
```

次は80行目あたりのエラーを解消します。

ここでは一旦「firstName」と「lastName」を書いているので以下のように書き換えます。

```diff
- substitutionValues: {
-                          'first_name': firstName,
-                          'last_name': lastName,
-                        },

+  substitutionValues: {
+                          'first_name': _firstNameController.text,
+                          'last_name': _lastNameController.text,
+                        },

```

以上のコードは何を変えているかと言うとテキストフィールドに入力した文字を送信するようにしています。

一旦接続できているか確認する為にビルドします。

ここで注意点ですがAndroid Studioでビルドする時は「iPhone」のエミュレーターを選択して下さい。

なぜかAndroid端末を選択するとエラーが出ます（原因調査中）

ビルドが終わればテキストフィールドに文字を入力して送信します。

<img src="image/スクリーンショット 2023-09-20 11.16.36.png" width="50%">

サンプルコードの中にデータが送信すれば「success!」 と 
出力するようにしているのでConsoleに出力されていれば成功です。

念の為pgAdmin4側でもデータが取得できているか確認します。

```sql
SELECT * FROM human;
```

以下のように出力されればデータの追加は完了です。


```bash
test_db=# select * from human;
 id | first_name | last_name 
----+------------+-----------
  5 | 太郎       | 大発
  6 | hanako     | daihatsu
```

一旦ここでコードの解説を挟みます。

```dart
  final connection = PostgreSQLConnection(
                      'localhost', // ホスト名
                      5432, // ポート番号
                      'test_db', // データベース名
                      username: 'postgres', // ユーザー名
                      password: '!!!!!your_password!!!!!', 
                    );
```

以上のコードはflutter pub add postresで追加したパッケージの
関数を実行して、PostgreSQLと通信を行いオブジェクトを生成しています。

`localhost`  
データベースサーバーのホスト名を指定しています。  
この例ではローカルホスト（アプリケーションが実行されているコンピューター自体）を指定しています。  
`5432`  
データベースサーバーのポート番号を指定しています。  
PostgreSQLのデフォルトポート番号は通常5432です。  
`test_db`  
データベース名を指定しています。この接続は 'test_db' という名前のデータベースに接続しようとしています。  
`username: 'postgres'`  
データベースへの接続に使用するユーザー名を指定しています。  
この例では 'postgres' というユーザー名を使用しています。  
`password: 'ここに設定したパスワードを入力'`  
データベースへの接続に使用するパスワードを指定しています。ユーザーのパスワードをここに入力する必要があります。  
セキュリティのためにパスワードを平文で**直接コードに記述するのは避けるべき**です。  
通常、環境変数やセキュアな方法でパスワードを設定することが推奨されます。
一旦お試しなのでここでは直接入力しますがgithubなどにコードをあげるときは絶対にこのまま上げないようにして下さい。（APIキーなども同様）

***

```dart
await connection.open();
```

以上のコードはPostgreSQLデータベースへの接続を開始します。  
具体的には、データベースサーバーに接続し、接続が確立されるのを待機します。  
このメソッドは非同期メソッドであるため、接続が開かれるまで待機し、  
接続が確立されたら次のステップに進みます。

```dart
 final insertResult = await connection.execute(
                        'INSERT INTO human (first_name, last_name) VALUES (@first_name, @last_name)',
                        substitutionValues: {
                          'first_name': _firstNameController.text,
                          'last_name': _lastNameController.text,
                        },
                      );
```

`insertResult 変数`  
この変数は、データベースへの挿入操作の結果を格納するために使用されます。  
操作が成功したかどうか、またはエラーが発生したかどうかを確認するために使用できます。  

`connection.execute()`  
このメソッドは、PostgreSQLデータベースに対してクエリを実行するために使用されます。  
この場合、以下のINSERT文が実行されます。  

```sql
INSERT INTO human (first_name, last_name) VALUES (@first_name, @last_name)

```

ここで、human テーブルに新しい行が挿入され、first_name と last_name 列に値が挿入されます。  
@first_name および @last_name はプレースホルダーであり、後で置換されます。

`substitutionValues`   
このパラメータは、プレースホルダーで@first_name と @last_name の値を置換するためのデータを指定します。  
具体的な値は _firstNameController.text と _lastNameController.text から取得されます。

```dart
  if (insertResult > 0) {
                        print('success!');
                      } else {
                        print('failed...');
                      }
```

insertResultには変動のあった行数の数がリターンされています。  
よくわからないと思うのでpostgresパッケージのコードを見てみます。 
以下のコードはパッケージのコードなので今は理解する必要ありませんので見るだけで結構です。 

```dart
 @override
  Future<int> execute(String fmtString,
      {Map<String, dynamic>? substitutionValues = const {},
      int? timeoutInSeconds}) async {
    final result = await _execute(
      fmtString,
      substitutionValues: substitutionValues,
      timeoutInSeconds: _connection.queryTimeoutInSeconds,
      onlyReturnAffectedRows: true,
    );
    // ここでリターンが実行
    return result.affectedRowCount;
  }
```

つまり上のif文のコードはinsertResultが0ならPostgreSQLに何も変動はないため  
PostgreSQL自体に問題があると推測されます。

本番運用にはあまり意味がありませんが開発時には重要なので入れておきましょう。

コードの中にtryやcatchを見かけたと思います。  
これはエラーの確認やエラー時の動作の挙動をコントロールするのに重要です。

```dart

try {
    // エラーがなければここが実行
} catch (e) {
    // エラーが出ればeという変数にエラー文を格納して出力
    print('Error: $e');
} finally {
    // ここは正常に動いてもエラーが出ても必ず実行されます。
    // データベースの接続を停止
    await connection.close();
}
```

ここまでがflutterにてPostgreSQLへデータを送信する流れになります。

## ⑤PostgreSQLのデータを取得

データ受信ボタンのonChangedのところに以下のコードを貼り付けます。

ここでもパスワードの書き換えを忘れないようにして下さい。

```dart
final connection = PostgreSQLConnection(
    'localhost', // ホスト名
    5432, // ポート番号
    'test_db', // データベース名
    username: 'postgres', // ユーザー名
    password: '!!!!!your_passwaord!!!!!', // データベースパスワード
  );

  try {
    await connection.open();

    final results = await connection.query('SELECT * FROM human');

    for (final row in results) {
      final id = row[0];
      final firstName = row[1];
      final lastName = row[2];
      print('id is $id : first name is $firstName : last name is $lastName');
    }
  } catch (e) {
    print('Error: $e');
  } finally {
    await connection.close();
  }
```

送信コードの時と同じようにasncyをつけます。

```diff
- ElevatedButton(
-               onPressed: () {
+ ElevatedButton(
+               onPressed: () async {
```

一旦ビルドして、エミュレータ上の「データ受信」のボタンを押します。

ここではprint文で出力するようにしているのでconsoleを確認します。  

consoleにPostgreSQLのデータが表示されていれば取得成功です。

ではコードの解説を行なっていきます。

```dart
final results = await connection.query('SELECT * FROM human');
```

`connection.query()`   
このメソッドは、PostgreSQLデータベースへのクエリを実行するために使用されます。  
引数としてクエリ文字列を受け取り、データベースに対してクエリを実行し、その結果を取得します。

次は取得した結果の扱いについてです。

```dart
for (final row in results) {
  final id = row[0];
  final firstName = row[1];
  final lastName = row[2];
  print('idは$idです。名前は $firstNameです。苗字は $lastNameです');
}
```

ここではfor文でデータを分割しています。  
for文について忘れた方はUdemyをもう一度見直すことをお勧めします。

少しデータの構造がわかりづらいので間にprint(results);を挟み中身の確認をします。  

```bash
flutter: [[5, 太郎, 大発], [6, hanako, daihatsu], [7, hanako, daihatsu]]
```

リスト型の中にリスト型が3つ並んでいます。  
ここはデータを送信した内容によって異なるので全く同じじゃなくても安心して下さい。  

なので

```dart
print(results[0]);
```

を実行すると  
`[5, 太郎, 大発]`が出力されます。

```dart
print(results[1]);
```
を実行すると `[6, hanako, daihatsu]`が出力されます。

このresultsという変数を分解して活用します。

```dart
for (final row in results) {
```

変数の「row」にはresultsが持つ`[5, 太郎, 大発]`などのデータが入っています。

```dart
  final id = row[0];
  final firstName = row[1];
  final lastName = row[2];
```

つまりfor文の繰り返し処理の中で`「5」「太郎」「大発」`を順番に代入していくができます。

以上のように必要なデータのみを指定してアプリに活用していきます。

