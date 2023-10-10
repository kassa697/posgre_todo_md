# flutter firebaseプロジェクト移行手順

1. 移行対象のflutterプロジェクトを開く
2. 以下の画像にあるlib/firebase_options.dartを削除する。

<img src="image/スクリーンショット 2023-10-05 9.04.18.png" width="50%">

3. 移行先のアカウントでfirebaseコンソールへログインする
4. firebaseコンソールでアプリの登録を再度行うと移行が完了

firebaseとflutterアプリの接続を忘れてしまった方は以下の記事を参考にして下さい。

- [flutter cloud firestoreとの接続入門](https://zenn.dev/articles/4f48c4c928fa64/edit)