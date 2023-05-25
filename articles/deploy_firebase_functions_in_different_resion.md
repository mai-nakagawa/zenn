---
title: "PythonのFirebase FunctionsでRegionを指定する方法" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["firebase", "python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
publication_name: "singularity"
---
# これは何？

PythonのFirebase Functionsはbetaということもあり、Regionを指定する方法が2023年5月25日時点では公式ドキュメントに記載されていない。Pythonのfirebase_functionsのソースコードを見て調べる必要があった。

# 方法

以下のようにして指定できることが分かった。
```python
options.set_global_options(region=options.SupportedRegion.ASIA_NORTHEAST1)
```

以下のようなpythonファイルを作ってデプロイすれば良い:
```python
from firebase_admin import initialize_app
from firebase_functions import https_fn, options, storage_fn


initialize_app()
options.set_global_options(region=options.SupportedRegion.ASIA_NORTHEAST1)


@https_fn.on_request()
def on_request_example(_: https_fn.Request) -> https_fn.Response:
    return https_fn.Response("Hello world!")
```
