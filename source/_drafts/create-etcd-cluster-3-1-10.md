---
title: create etcd cluster 3.1.10
tags:
---

### 產生 ETCD cert
在哪台機器上產生 cert 都可以，這裡借用 jumpy 的使用者(centos) 的家目錄，之後再把 cert 丟到各個 etcd server 上
```
$ mkdir ~/bin 
$ curl -o ~/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 >/dev/null 2>&1
$ curl -o ~/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 >/dev/null 2>&1
$ chmod +x ~/bin/cfssl*
