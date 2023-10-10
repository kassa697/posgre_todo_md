# インストール手順 mongoDB

## 参考サイト

- [mongoDB公式サイト](https://www.mongodb.com)
- [Mac OSにMongoDBをインストールと基本操作](https://reffect.co.jp/windows/mac-mongodb-install)

homebrewで必要なものをインストールします。

```bash
brew tap mongodb/brew
```

```bash
brew install mongodb-community
```

moreコマンドで設定ファイルを確認します。

```bash
more /usr/local/etc/mongod.conf
```

mongoDBのバージョンを見てみます。

```bash
mongod --version
```

mongoDBを手動で起動します。

```bash
brew services start mongodb-community
```

手動で停止

```bash
brew services stop mongodb-community
```

起動中か確認したければ

```bash
brew services list
```

Statusがstartedとなっていれば起動中

再起動しても自動で起動することもできますが割愛します。

mongoDBを起動させます。

```bash
mongosh
```

ターミナルが「test> 」と表示されていればOK
testは最初からあるDBです。

データベースやコレクションの作り方はflutterと連携した後に補足として記述します。


# flutterへ導入準備

導入したいflutterプロジェクトでパッケージを入れます。

```bash
flutter pub add mongo_dart
```

あとはパッケージサイトの「Exeample」を試せば接続できているか確認できます。
しかし少しわかりづらいので少しサンプルを変更してボタンを押すとデータを飛ばすようにします。

今回作成するファイルは「main.dart」と「mongo.dart」の二つのファイルを作成します。

```dart:main.dart
import 'package:flutter/material.dart';
import 'mongo.dart';

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
        primarySwatch: Colors.blue,
      ),
      home: const Home(),
    );
  }
}

class Home extends StatefulWidget {
  const Home({Key? key}) : super(key: key);

  @override
  State<Home> createState() => _HomeState();
}

class _HomeState extends State<Home> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        alignment: Alignment.center,
        child: const ElevatedButton(onPressed: sendData, child: Text('送信')),
      ),
    );
  }
}
```

二つ目

```dart:mongo.dart
import 'package:mongo_dart/mongo_dart.dart';
import 'dart:io' show Platform;

String host = Platform.environment['MONGO_DART_DRIVER_HOST'] ?? '127.0.0.1';
String port = Platform.environment['MONGO_DART_DRIVER_PORT'] ?? '27017';

Future<void> sendData() async {
  var db = Db('mongodb://$host:$port/mongo_dart-blog');
  // Example url for Atlas connection
  /* var db = Db('mongodb://<atlas-user>:<atlas-password>@'
      'cluster0-shard-00-02.xtest.mongodb.net:27017,'
      'cluster0-shard-00-01.xtest.mongodb.net:27017,'
      'cluster0-shard-00-00.xtest.mongodb.net:27017/'
      'mongo_dart-blog?authSource=admin&compressors=disabled'
      '&gssapiServiceName=mongodb&replicaSet=atlas-stcn2i-shard-0'
      '&ssl=true'); */
  var authors = <String, Map>{};
  var users = <String, Map>{};
  await db.open();
  await db.drop();
  print('====================================================================');
  print('>> Adding Authors');
  var collection = db.collection('authors');
  await collection.insertMany([
    {
      'name': 'ckaaskaksaks Shakespeare',
      'email': 'william@shakespeare.com',
      'age': 587
    },
    {
      'name': 'Jorge Luis Borges',
      'email': 'jorge@borges.com',
      'age': 123}
  ]);
  await db.ensureIndex('authors',
      name: 'meta', keys: {'_id': 1, 'name': 1, 'age': 1});
  await collection.find().forEach((v) {
    print(v);
    authors[v['name'].toString()] = v;
  });
  print('====================================================================');
  print('>> Authors ordered by age ascending');
  await collection.find(where.sortBy('age')).forEach(
      (auth) => print("[${auth['name']}]:[${auth['email']}]:[${auth['age']}]"));
  print('====================================================================');
  print('>> Adding Users');
  var usersCollection = db.collection('users');
  await usersCollection.insertMany([
    {'login': 'jdoe', 'name': 'John Doe', 'email': 'john@doe.com'},
    {'login': 'lsmith', 'name': 'Lucy Smith', 'email': 'lucy@smith.com'}
  ]);
  await db.ensureIndex('users', keys: {'login': -1});
  await usersCollection.find().forEach((user) {
    users[user['login'].toString()] = user;
    print(user);
  });
  print('====================================================================');
  print('>> Users ordered by login descending');
  await usersCollection.find(where.sortBy('login', descending: true)).forEach(
      (user) =>
          print("[${user['login']}]:[${user['name']}]:[${user['email']}]"));
  print('====================================================================');
  print('>> Adding articles');
  var articlesCollection = db.collection('articles');
  await articlesCollection.insertMany([
    {
      'title': 'Caminando por Buenos Aires',
      'body': 'Las callecitas de Buenos Aires tienen ese no se que...',
      'author_id': authors['Jorge Luis Borges']?['_id']
    },
    {
      'title': 'I must have seen thy face before',
      'body': 'Thine eyes call me in a new way',
      'author_id': authors['William Shakespeare']?['_id'],
      'comments': [
        {'user_id': users['jdoe']?['_id'], 'body': 'great article!'}
      ]
    }
  ]);
  print('====================================================================');
  print('>> Articles ordered by title ascending');
  await articlesCollection.find(where.sortBy('title')).forEach((article) {
    print("[${article['title']}]:[${article['body']}]:"
        "[${article['author_id'].toHexString()}]");
  });
  await db.close();
}
```

ボタンを押してmongoDBにデータが飛んでいればOKです。

# mongoDBの使い方

mongoDB使い方知らないのにデータの確認するのも大変なので基本的なコマンドを紹介します。

firestoreもmongoDBもNoSQLなので構造のイメージがしやすいと思います。

データベース確認

```bash
show dbs
```

データベース作成

```bash
use hoge
```

useは作成とデータベースの切り替えをしてくれます。

作成してコレクションなどを作成しなければ保存はされません。

コレクション作成

```bash
db.createCollection('moge')
```

コレクション確認

```bash
show collections
```

ドキュメント作成

```bash
db.moge.insertOne({name:'taro'})
```
複数作成する場合はinsertManyを実行

ドキュメント確認

```bash
db.moge.find()
```

ドキュメント出力整形

```bash
db.moge.find().pretty()
```