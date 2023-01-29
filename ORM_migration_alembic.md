## GOAL：PythonのDB操作まわりの解像度を上げよう  

### 解決したいこと
SQLAlchemyとalembicが並列と思ったら親子関係っぽかった（全然違った。記事の誤読。）
全く理解できていないのでこの機に整理する。  

### SQLAlchemyとは  
公式サイト：  https://www.sqlalchemy.org/
>SQLAlchemy is the Python SQL toolkit and Object Relational Mapper that gives application developers the full power and flexibility of SQL.

Pythonでよく使われるORM
	ORMとは：ObjectRelationalMapper／オブジェクト関係マッピング
		オブジェクト＝プログラムで扱うオブジェクト→プログラムにおけるクラス
			※クラス＝メソッドが記述されてる（要するにPythonで書かれてるという話）
		関係＝関係データベース（RDB）→DBのテーブル
		マッピング：一対一で紐づける
		→→つまり：DBのテーブルとプログラムのクラスを紐づけて、PythonでDB操作できる	
**【発見】** FastAPIのチュートリアルにSQLAlchemyの使い方記載あり（英語）  

### alembicとは
公式サイト：  https://alembic.sqlalchemy.org/
> Alembic is a lightweight database migration tool for usage with the SQLAlchemy Database Toolkit for Python.

SQLAlchemyで使われるDBマイグレーションツールのこと。  

### マイグレーションとは
マイグレーション（migration）とは、英語で「移動」「移行」を意味する。
IT分野では**システム環境の移行**のことを指す。
マイグレーションにはいくつか種類がある。
 - データマイグレーション
 - サーバマイグレーション
 - クラウドマイグレーション
 - レガシーマイグレーション
よく混同される概念としてリプレイスがある。マイグレーションの種類やリプレイス等との違いは省略。
データベースマイグレーションとは文字通り「データベース移行」という意味で現在のDBシステムを**そのまま**別のDBシステムに移行することを指す。それだけではなく（おそらく、転じて）、DB内のデータを維持したままテーブル構成を変更する作業のことをマイグレーションと呼ぶ。（DBを作り直す際に中身のデータが削除されては困るので・・・？）
	よく分からない：今あるものを移すことがマイグレーションなのに0からのテーブル作成もマイグレーションによりDB構築するっぽいこと（できるならそれでいいけど・・・）
	→参考：マイグレーションファイル
### 要するに
SQLAlchemy＝PythonでDB操作する。alembic＝SQLAlchemyで使うマイグレーションツール。
ということは、Pythonアプリ開発でのマイグレーションとは、PythonでDB構築すること。  

### マイグレーションファイル
DBの設計図のこと。
alembicの環境設定とSQLAlchemyのモデル作成（つまりmodels.py）を行うことでマイグレーションファイルは自動生成できるらしい。

### alembicの環境設定
alembicを使うために必要なファイルはコマンドで作成される。
```
$ alembic init "マイグレーション環境名"
```
こんな感じ。（環境名：migrations）
```
$ tree
.
├── alembic.ini
└── migrations
    ├── README
    ├── env.py
    ├── script.py.mako
    └── versions
```
 - alembic.ini
   alembicのスクリプトが実行される時に読み込まれる構成ファイル。実行時の設定を記述。
 - README.md
   どのような環境でmigration環境を作成したかの記述
 - env.py
   alembic起動の度に実行されるPythonスクリプト。SQLAlchemyのengineを設定・生成してマイグレーションが実行できるようになっている。カスタマイズできる。
 - script.py.mako
   新しいmigrationスクリプトを生成するために使われるMakoテンプレートファイル。
	   Mako：Pythonのテンプレートエンジン。情報がなさすぎるので省略。
   versions内の新しいファイルを生成するために使われる。
 - versionsディレクトリ
   migrationスクリプトが保存されるディレクトリ
### alembic.iniを編集
SQLAlchemyが接続するためのURLを実行環境に応じて修正しておく。  

### SQLAlchemyの設定
1. setting.pyの作成
   DB設定を行う。トップディレクトリに作成。
2. models.pyの作成
   マイグレーションファイル生成のために作る。つまり設計図のためにある。ER図はここに落とし込むっぽい。
### alembicの環境設定ファイルenv.pyを修正
SQLAlchemyの設定に応じてenv.pyを修正する。
おそらくこれでSQLAlchemyとalembicがいい感じに紐づく。

### マイグレーションファイルを自動生成
コマンドで自動生成できる。
```
$ alembic revision --autogenerate -m "create tables"
```

### DBにマイグレーションファイルの内容を反映
これもコマンドでできる。
```
$ alembic upgrade head
```

### 気づき（課題）
 - SQLAlchemyの使い方・engineについて要調査
 - alembicはコマンドでいろいろできるらしい
   （現在の状態を確認したり、履歴を見たり、巻き戻したり）Gitっぽい。
 - マイグレーション内でSQL文を直接書くこともできるらしい