# PostgreSQLでのtodoアプリの作り方

## データベースの準備

アプリを作成する前にデータベースを準備していきます。

作成するデータベースは以下のようなイメージです。

ID : データを識別  
タイトル : todoタイトル  
説明 : todoの詳細  

![Alt text](<スクリーンショット 2023-10-04 12.23.11.png>)

pgAdmin4でpsqlを使って準備をします。

データベースの作成

```sql
CREATE DATABASE todo;
```

作成したデータベースへ接続

```sql
\c todo
```

テーブルの作成

```sql
CREATE TABLE memo (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    description TEXT
);
```

これでデータベースの準備は完了です。

後々にデータベース名とテーブル名はflutterで使用するので  
頭の片隅に置いておいて下さい。

## flutter project作成

- flutterプロジェクトを作成します。


<img src="スクリーンショット 2023-10-04 11.12.30.png" width="80%">

プロジェクト名は好きな名前で結構です。

ではデフォルトで生成されたコードを整えていきます。

1. 39行目から下を全て削除
2. 英語のコメントも邪魔なので削除

すると以下の様になります。

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

```

タイトル名を送る必要はないので以下の様に編集します。

```diff
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
-      home: const MyHomePage(title: 'Flutter Demo Home Page'),
+      home: const MyHomePage(),
    );
  }
}

```

そのまま22行目から下に続けてMyHomePageを作成していきます。
「stf」と入力すると予測変換が出てきます。

<img src="スクリーンショット 2023-10-04 11.32.01.png" width="50%">

「stful」を選択すると雛形が生成されます。

<img src="スクリーンショット 2023-10-04 11.33.02.png" width="50%">

ここで「MyHomePage」と入力して下さい。
以下の様になればOKです。

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return const Placeholder();
  }
}
```

では書き換えていきます。

```diff
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
-    return const Placeholder();
+    return Scaffold(
+      appBar: AppBar(
+        title: const Text('todoアプリ'),
+      ),
+      body: const Center(
+        child: Text('test'),
+      ),
+    );
  }
}
```

一旦ビルドして動作確認をしてみましょう。

エラーなくビルドができればOKです。

ここからまた書き換えます。

```diff
class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('todoアプリ'),
      ),
-      body: const Center(
-        child: Text('test'),
-      ),
+body: ListView.builder(
+        itemCount: 3,
+          itemBuilder: (context, index) => Card(
+                color: Colors.lightGreen,
+                margin: const EdgeInsets.all(15),
+                child: ListTile(
+                  title: Text('仮タイトル'),
+                  subtitle: Text('仮説明欄'),
+                  trailing: SizedBox(
+                    width: 100,
+                    child: Row(
+                      children: [
+                        IconButton(
+                            onPressed: () {}, icon: const Icon(Icons.edit)),
+                        IconButton(
+                            onPressed: () {},
+                            icon: const Icon(Icons.delete_forever_rounded))
+                      ],
+                    ),
+                  ),
+                ),
+              )),
    );
  }
}
```


一旦ビルドして以下の画像の様になればOKです。

<img src="スクリーンショット 2023-10-04 12.45.26.png" width="50%">

ではフローティングアクションボタンを押すとデータを追加するようにしたいので  
一旦ボタンだけ配置します


少しわかりづらいですが以下のところに挿入します。

<img src="スクリーンショット 2023-10-04 12.53.41.png" width="50%">

```dart
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        onPressed: () => _showForm(),
      ),
```

エラーが出てますが後で実装するので一旦は放置します。

次はtodoの内容を追加するUIを実装していきます。

実装イメージはフローティングアクションボタンを押すと  
以下の様に入力フォームが下から出てくる様にします。  

<img src="スクリーンショット 2023-10-04 13.03.07.png" width="50%">

```dart
class _MyHomePageState extends State<MyHomePage> {
  // この間にボトムシートを表示するコードを記述します。
  // ...............
  @override
  Widget build(BuildContext context) {
```

フローティングアクションボタンを押すと表示させる  
ボトムシートのコードは以下になります。

```dart
  Future<void> _showForm() async {
    await showModalBottomSheet(
        context: context,
        elevation: 5,
        isScrollControlled: true,
        builder: (_) => Container(
              padding: EdgeInsets.only(
                top: 15,
                left: 15,
                right: 15,
                bottom: MediaQuery.of(context).viewInsets.bottom + 100,
              ),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                crossAxisAlignment: CrossAxisAlignment.end,
                children: [
                  TextField(),
                  TextField(),
                  const SizedBox(
                    height: 20,
                  ),
                  ElevatedButton(onPressed: () {}, child: Text('新規追加'))
                ],
              ),
            ));
  }
```

ここで一旦ビルドしてみましょう。

以下の画像の様になればOKです。
<img src="スクリーンショット 2023-10-04 13.17.09.png" width="50%">

しかしこれでは入力欄の昨日や何を入力するか分からないので  
コードを補完していきます。

```diff
+  final TextEditingController _titleController = TextEditingController();
+  final TextEditingController _descriptionController = TextEditingController();
  Future<void> _showForm() async {
    await showModalBottomSheet(
        context: context,
        elevation: 5,
        isScrollControlled: true,
        builder: (_) => Container(
              padding: EdgeInsets.only(
                top: 15,
                left: 15,
                right: 15,
                bottom: MediaQuery.of(context).viewInsets.bottom + 100,
              ),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                crossAxisAlignment: CrossAxisAlignment.end,
                children: [
                  TextField(
+                    controller: _titleController,
+                    decoration: const InputDecoration(hintText: 'タイトル'),
                  ),
                  TextField(
+                    controller: _descriptionController,
+                    decoration: const InputDecoration(hintText: '内容'),
                  ),
                  const SizedBox(
                    height: 20,
                  ),
                  ElevatedButton(onPressed: () {}, child: Text('新規追加'))
                ],
              ),
            ));
  }
```

UI作りは一旦ここまでにしてPostgreSQLを操作するクラスを設計していきます。

## PostgreSQL操作クラス作成

ここのコードの中身は他の教育資料やUdemy講座と被る所があるので説明は省略します。 

libフォルダ直下に「postgres_model.dart」というファイルを作成します。

挿入するコードは以下になります。

エラーが出ますが一旦無視して貼り付けて下さい。

```dart

class Postgres {

  static Future<void> insert(
      {required String title, required String description}) async {
    final connection = PostgreSQLConnection(
      'localhost', // ホスト名
      5432, // ポート番号
      dbName, // データベース名
      username: userName, // \lコマンドでOwnerのところを入力
      password: password,
    );

    try {
      await connection.open();
      final insertResult = await connection.execute(
        'INSERT INTO $tableName (title, description) VALUES (@title, @description)',
        substitutionValues: {
          'title': title,
          'description': description,
        },
      );

      if (insertResult > 0) {
        print('データの追加に成功しました');
      } else {
        print('データの追加に失敗しました');
      }
    } catch (e) {
      print('Error: $e');
    } finally {
      await connection.close();
    }
  }

  static Future<PostgreSQLResult?> getItem() async {
    try {
      final connection = PostgreSQLConnection(
        'localhost', // ホスト名
        5432, // ポート番号
        dbName, // データベース名
        username: userName,
        password: password,
      );
      await connection.open();
      final results = await connection.query('SELECT * FROM $tableName');
      return results;
    } catch (e) {
      print('Error: $e');
    } finally {
      final connection = PostgreSQLConnection(
        'localhost', // ホスト名
        5432, // ポート番号
        dbName, // データベース名
        username: userName,
        password: password,
      );
      await connection.close();
    }
    return null;
  }

  static Future<void> update(
      {required int id,
        required String updateTitle,
        required String updateDescription}) async {
    final connection = PostgreSQLConnection(
      'localhost', // ホスト名
      5432, // ポート番号
      dbName, // データベース名
      username: userName,
      password: password,
    );
    await connection.open();

    final updateResult = await connection.execute(
      'UPDATE $tableName SET title = @title, description = @description WHERE id = @id',
      substitutionValues: {
        'id': id,
        'title': updateTitle,
        'description': updateDescription,
      },
    );

    if (updateResult > 0) {
      print('データの更新に成功しました');
    } else {
      print('データの更新に失敗しました');
    }
  }

  static Future<void> delete({required int id}) async {
    final connection = PostgreSQLConnection(
      'localhost', // ホスト名
      5432, // ポート番号
      dbName, // データベース名
      username: userName,
      password: password,
    );
    await connection.open();

    final deleteResult = await connection.execute(
      'DELETE FROM $tableName WHERE id = @id',
      substitutionValues: {
        'id': id,
      },
    );

    if (deleteResult > 0) {
      print('データの削除に成功しました');
    } else {
      print('データの削除に失敗しました');
    }
  }
}

```

ではエラーを解消していきます。

まずは一番大事なPostgreSQLを操作するdartパッケージをインストールします。

ターミナルで  

```bash
flutter pub add postgres
```

コマンドがエラーなく終わればインポートしましょう。

```diff
+ import 'package:postgres/postgres.dart';
```

他の箇所もたくさんエラーが出ています。

これは他のファイルで定数として使い回しをしている為です。

なので定数を設定するファイルを作成します。

これもlib直下に「setting.dart」として作成します。

このファイルにはPosrgreSQLのパスワード等を設定する必要があるので適時書き換えて下さい

```dart
const String password = 'ここにパスワードを入力します';
const String dbName = 'todo';
const String userName = 'postgres';
const String tableName = 'memo';
```

ではこのファイルを「postgres_model.dart」でインポートしましょう。

赤線のところで「option + エンター」でインポートして下さい。

一旦SQL接続確認をします。

```diff
-  ElevatedButton(onPressed: () {}, child: Text('新規追加'))
+  ElevatedButton(onPressed: () async {
+    Postgres.insert(title: _titleController.text, description: _descriptionController.text);
+  }, child: Text('新規追加'))
```

上記のコードを追加すると以下の様に確認ができます。

※アプリ側ではSQLの中身を取得するコードはまだ実装していないので  
pgAdmin側でデータが送信できているか一旦確認します。

```sql
SELECT * FROM memo;
```

先ほど入力したデータが送信されていれば接続はOKです。

もしエラーが出た場合は誤字などの恐れがあるのでエラー文をよくみて  
もう一度確認する様にしましょう。

<img src="スクリーンショット 2023-10-04 13.41.32.png" width="50%">

ではアプリ側でSQLのデータを取得するコードを実装していきます。

_showForm関数の時と同様に

```dart
class _MyHomePageState extends State<MyHomePage> {
  // ここに関数を定義します
```

取得したデータを格納する変数とデータを取得するコードです。

```dart
  List<Map<String, dynamic>> _todos = [];
  Future <void> _refreshTodos() async {
    final data = await Postgres.getItem();
    setState(() {
      _todos = data!.map<Map<String, dynamic>>((row) {
        return {
          'id': row[0],
          'title': row[1],
          'description': row[2],
        };
      }).toList();
    });
  }
```

では取得したデータを活用して画面に表示する準備をしていきます。

```diff
 body: ListView.builder(
-          itemCount: 3,
+          itemCount: _todos.length,
          itemBuilder: (context, index) => Card(
                color: Colors.lightGreen,
                margin: const EdgeInsets.all(15),
                child: ListTile(
-                 title: Text('仮タイトル'),
-                 subtitle: Text('仮説明欄'),
+                 title: Text(_todos[index]['title']),
+                 subtitle: Text(_todos[index]['description']),
                  trailing: SizedBox(
                    width: 100,
                    child: Row(
                      children: [
                        IconButton(
                            onPressed: () {}, icon: const Icon(Icons.edit)),
                        IconButton(
                            onPressed: () {},
                            icon: const Icon(Icons.delete_forever_rounded))
                      ],
                    ),
                  ),
                ),
              )),
```

そして画面立ち上げ時にデータ取得の関数を実行する様にしましょう。

_showForm関数の時と同様に

```dart
class _MyHomePageState extends State<MyHomePage> {
  // ここに関数を定義します
```

```dart
  @override
  void initState() {
    super.initState();
    _refreshTodos();
  }
```

以下の様に表示されていればOKです。

<img src="スクリーンショット 2023-10-04 13.54.49.png" width="50%">

ここまでで取得と送信はできたので次は更新と削除機能を実装していきます。

先ほどと同様に位置に関数を挿入します

```dart
// データ送信後画面更新をしたいので以下の関数も追加します。
  Future<void> _addItem() async {
    await Postgres.insert(
        title: _titleController.text, description: _descriptionController.text);
    // データの再取得
    await _refreshTodos();
  }
  // todo項目更新
  Future<void> _updateItem(int id) async {
    await Postgres.update(
        id: id,
        updateTitle: _titleController.text,
        updateDescription: _descriptionController.text);
    await _refreshTodos();
  }
  // todo項目削除
  Future<void> _deleteItem(int id) async {
    await Postgres.delete(id: id);
    // ignore: use_build_context_synchronously
    ScaffoldMessenger.of(context).showSnackBar(const SnackBar(
      content: Text('リストを削除しました。'),
    ));
    await _refreshTodos();
  }
```

以上の関数を実行するようにコードを編集します。

ここではアプリ画面の鉛筆マークを押すと更新のフォームが現れる様にします。

先ほど実装したフォームを再利用するので_showForm関数にIDを渡して更新や削除の関数に活用します。

```diff

                    child: Row(
                      children: [
                        IconButton(
-                            onPressed: () {},
+                            onPressed: () => _showForm(_todos[index]['id']), 
                            icon: const Icon(Icons.edit)),
```

データを新規追加する時はnullを渡して後々条件式で活用していきます。

```diff
floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
-        onPressed: () => _showForm(),
+        onPressed: () => _showForm(null),
      ),
```

次は編集ボタンの鉛筆マークをタップすればフォームに文字が入る様にします。

_showForm関数の真下に以下のコードを挿入します。

また、_showForm関数でIDを受け取るので引数に「int? id」も追記します。

```diff
-  Future<void> _showForm() async {
+  Future<void> _showForm(int? id) async {
+    if (id != null) {
+      final existingTodos = _todos.firstWhere((todo) => todo['id'] == id);
+      _titleController.text = existingTodos['title'];
+      _descriptionController.text = existingTodos['description'];
+    }
```

ここまでで一旦動作確認をします。

鉛筆マークを押すと入力された内容がテキストフィールドに反映されていると思います。

しかしここで問題が一つあります。

そのままフローティングアクションボタンの新規追加ボタンを押すと

前の状態の値が入ってしまうので初期化のコードを追記します。

```diff
  Future<void> _showForm(int? id) async {
+    _titleController.text = "";
+    _descriptionController.text = "";
    if (id != null) {
      final existingTodos = _todos.firstWhere((todo) => todo['id'] == id);
      _titleController.text = existingTodos['title'];
      _descriptionController.text = existingTodos['description'];
    }
```

これで初期化ができました。

しかしまだフォームのボタンの表記がおかしいので修正します。

新規追加時は「新規追加」

更新のフォームの際は「更新」と表示する様にします。

ここは簡単です。

三項演算子を使用すればシンプルに条件式が実装できます。

```diff

    ElevatedButton(onPressed: () async {
      Postgres.insert(title: _titleController.text, description: _descriptionController.text);
-      }, child: Text('新規追加'))
+      }, child: Text(id == null ? '新規追加' : '更新'))
```

これで新規追加する時のフォームと更新する時の用のフォームを表示する事ができました。

次はフォームのボタンを押すとデータの追加、更新を実装します。

先ほどのテスト用のコードは削除します。


```diff

  ElevatedButton(onPressed: () async {
-    Postgres.insert(title: _titleController.text, description: _descriptionController.text);
  }, child: Text('新規追加'))
```

以下が条件式になります。

データがないときは追加のコード実行、データがあればIDを元に更新を実行します。

```diff
                  ElevatedButton(onPressed: () async {
+                    if (id == null) {
+                      await _addItem();
+                    } else {
+                      await _updateItem(id);
+                    }
+                    Navigator.of(context).pop();
                  }, child: Text(id == null ? '新規追加' : '更新'))
```

次はデータの削除の実装です。

```diff
 IconButton(
-        onPressed: () => _showForm(_todos[index]['id']),
+        onPressed: () => _deleteItem(_todos[index]['id']),
        icon: const Icon(Icons.delete_forever_rounded))
```

これでCRUDの実装は完了です。

しかしアプリは様々状況を想定してユーザーに優しい実装をするべきです。

試しにアプリのゴミ箱マークをタップしてデータを全て削除しましょう。

<img src="スクリーンショット 2023-10-04 14.47.41.png" width="50%">

データがない時はアプリがどの様な状態かわかりません。

これではユーザーに優しくないのでデータがない時は「データがありません」と表示します。

この実装も三項演算子を用いれば簡単にできます。

```diff
- body: ListView.builder(
+ body: _todos.isEmpty
+           ? const Center(
+               child: Text('データがありません'),
+             )
+           : ListView.builder(
```

以上でPostgreSQLを用いたアプリ作りは終了です。

## 補足

- 本来はTodoアプリにPostgreSQLは適しません。（sqfliteなどの端末内で完結するパッケージを主に使う）
- 今回はCRUDの処理を理解してもらうために学習をしてもらいました。
- このCRUDという処理はプログラミングにおいて重要な機能なので理解する様にして下さい。

