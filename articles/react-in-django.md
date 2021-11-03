---
title: "DjangoのtemplateにReactを埋め込んで使う"
emoji: "😱"
type: "tech"
topics: ["django", "react"]
published: false
---

DjangoにReactを埋め込む記事が全然無かったので、記録として書いておく。Webpackの勉強になった。

# 方針
`django/`と`frontend/`でディレクトリを分け、webpackが生成するファイルをDjangoのstaticディレクトリ内で共有する。