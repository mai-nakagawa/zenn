---
title: "PythonのFirebase Functionsで複数Pythonファイルを利用する方法" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["firebase", "python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
publication_name: "singularity"
---
# これは何？

Python版のFirebase Functionsで、複数のPythonファイルを利用する方法。

# 現象

以下のようなディレクトリ構成になっているとする:
```
myproject
+- .firebaserc
|
|
+- firebase.json
|
+- functions/
      |
      +- main.py
      |
      +- import_me.py  # このpythonファイルをmain.pyからimportしたい
      |
      +- requirements.txt
      |
      +- venv/
```

以下のようにmain.pyからimport_me.pyをimportする。
```python
import import_me

from firebase_admin import initialize_app
from firebase_functions import https_fn, options, storage_fn


initialize_app()


@https_fn.on_request()
def on_request_example(_: https_fn.Request) -> https_fn.Response:
    return https_fn.Response("Hello world!")
```

すると、`firebase deploy` コマンド実行時にエラーになる:
```
ModuleNotFoundError: No module named 'import_me'
```

# 原因

firebase/firebase-functions-pythonリポジトリのコードを読むと、`firebase deploy`コマンドによりFunctionをデプロイするときに、main.pyしかimportlibで読み込んでいない。そのため、他のファイルがimportできなくてエラーになっている（[該当箇所](https://github.com/firebase/firebase-functions-python/blob/81487233e639c572f1614e2a1cbed26f78badf55/src/firebase_functions/private/serving.py#L35)。

# 暫定対策

私たちができるワークアラウンドは、以下のように関数の中でimportする方法が考えられる。こうすれば `import import_me` は関数が呼ばれるまで実行されないため、デプロイ時にエラーにならない。
```python
from firebase_admin import initialize_app
from firebase_functions import https_fn, options, storage_fn


initialize_app()


@https_fn.on_request()
def on_request_example(_: https_fn.Request) -> https_fn.Response:
    import import_me  # これがワークアラウンド
    return https_fn.Response("Hello world!")
```

# 恒久対策

あるべき姿は、firebase/firebase-functions-pythonリポジトリ側でこれを直してもらうこと。リポジトリを見たら下記のPull Requestが挙がっていた。しかし5ヶ月間、レビューされていない:
https://github.com/firebase/firebase-functions-python/pull/12
