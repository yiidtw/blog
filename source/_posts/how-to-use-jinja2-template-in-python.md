---
title: 使用 Python 的 Jinja2 進行模板替換 
date: 2018-06-05 14:18:04
tags:
- Python2
- DevOps
- Jinja2
- CentOS7
---

## 思路

jinja2 之前看別人是用在 Flask 的網頁模板，但在運維上也蠻實用的。例如今天的工作，需要在不同環境佈署 container ，映象檔(image)不動，可是 config 需要 mapping 到不同環境，這時候也可以使用模板，下面舉一個很簡單的栗子

考慮到 CentOS 7 預設還是給 python 2.7，所以這裡還是使用 python 2 作為運維語言， shell script 也很常用，但沒有好用的 yaml 模組讓我苦手…

## 安裝與文件架構

```
$ pip install jinja2

$ tree .
.
├── convert.py
└── template.yml
```

## 操作

先看模板
```
$ cat ./template.yml
MyProject:
    ProjectName: {{ name }}
    ProjectItems: {% for i in items %}
        - {{ i }}{% endfor %}
```

這裡要換掉兩個變數
- name 變數
- iterator 遍歷一個 list 叫做 items，然後用 yaml 的型式(前面帶 - )呈現

簡單的 convert.py，使用 python2 的語法，並將結果寫到 foo.yml
```
$ cat convert.py 
#!/usr/bin/python

import jinja2
templateLoader = jinja2.FileSystemLoader(searchpath="./")
templateEnv = jinja2.Environment(loader=templateLoader)
template = templateEnv.get_template("template.yml")
result = template.render(name="foo", items=["a", "b", "c"])

with open("foo.yml", "wb") as f:
    f.write(result)
```

看結果
```
$ cat foo.yml 
MyProject:
    ProjectName: foo
    ProjectItems: 
        - a
        - b
        - c
```

以上，這個簡單的栗子可以擴展並滿足許多我們部署上的需求與各種 workaround

## Reference
- [python - how to load jinja template directly from filesystem - Stack Overflow](https://stackoverflow.com/questions/38642557/how-to-load-jinja-template-directly-from-filesystem)
- [介绍 — Jinja2 2.7 documentation](http://docs.jinkan.org/docs/jinja2/intro.html#api)
