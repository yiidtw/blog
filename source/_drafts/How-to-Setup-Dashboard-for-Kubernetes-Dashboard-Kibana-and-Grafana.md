---
title: 如何在 Kubernetes 裡設置 Kubernetes Dashboard ， Kibana 和 Grafana 的 SubPath
date: 2018-09-11 18:35:23
tags:
- Kubernetes
- Grafana
- Kibana
- DevOpsDays 2018
---

當我們設置好 Kubernetes Observability 三本柱(我自己給的稱號)後，可能會想要在同一個 domain 下，使用相同的 hostname 並設置 SubPath，例如：
- https://mydemoservice.example.com/dashboard
- https://mydemoservice.example.com/kibana
- https://mydemoservice.example.com/grafana

跟下面這種例子比較，可能更方便一些
- https://mydemodashboard.example.com
- https://mydemokibana.example.com
- https://mydemografana.example.com

三種設置方式略有不同，跟安裝方式也有關，不過都只是小技巧，稍微紀錄一下

---

---

---
