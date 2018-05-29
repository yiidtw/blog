---
title: 在 Mac book / Centos 7 上設置 Golang 的開發環境
tags:
  - Golang
  - Mac book
  - centos 7
date: 2018-05-23 15:31:52
---


最近要來用 Golang 開發一些東西了，先寫個 setup

## 跟 Golang 有關的重要環境變量

### GOROOT

![](https://qph.fs.quoracdn.net/main-qimg-9226c86887a176e77eb32e4db96f628e)
[Photo from Quora](https://www.quora.com/What-was-the-significance-of-Groot-saying-We-are-Groot-near-the-end-of-the-movie-when-all-through-he-is-only-capable-of-saying-I-am-Groot) I'm ~~GOROOT~~ GROOT

- GOROOT 就是 golang 安裝路徑，我直接用官方的 installer for MAC (.pkg) 安裝，預設會在 /usr/local/go，若客製化需要可更改
- 裝完記得修改 ~/.profile 或是 ~/.bash_profile，修改完要記得 source
```
export PATH=$PATH:/usr/local/go/bin
```

### GOPATH (最重要)
- 預設在 $HOME/go ，若客製化需要可更改
- 通常會有三個 dir ，分別是 src、pkg、bin ，我們只需要手動創建 ~/go/src ，像是 `$ mkdir -p $HOME/go/src` ，其他兩個會在 go install 有需要時自動創建

### GOBIN
- 基本上不設，可為空，若客製化需要可更改

### 其他
- 可用 `$ go env` 查看 golang 用到相關的環境變數
- project 要用到的版本管理 a.k.a. git ，應該在 src 下，一個 project 一個 dir，詳見下段工程目錄結構

---

## CentOS 7  上安裝 go 開發環境

- 先去官方 source wget 最新的 tar 檔 [Downloads - The Go Programming Language](https://golang.org/dl/)
- 解壓縮
- 設置 Path
```
$ cd /tmp; wget https://dl.google.com/go/go1.10.2.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
```

修改 ~/.bash_profile，export 後加上 `:/usr/local/go/bin`
```
$ source ~/.bash_profile
```


---

## 工程目錄結構

- 上面說過我們需要手動創建的 dir 只有 src ，不過本篇範例還不會用到 go get ，所以 ./pkg 還不會被創建出來，為了先 demo ，我們這邊手動 mkdir 了
- 這邊有兩個專案，分別是 hello 和 foo ，各自都有 main function

```
$ tree ~/go -a

.
├── bin
│   ├── foo
│   └── hello
├── pkg
└── src
    ├── foo
    │   ├── .git
    │   └── foo.go
    └── hello
        ├── .git
        └── hello.go
```

至於 hello.go 程式本身很簡單，就是印 hello, world 而已
```
$ cat ~/go/src/hello/hello.go 
package main

import "fmt"

func main(){
	fmt.Printf("hello, world\n")
}
```

---

## 編譯方法

### 推薦用這個

- go install: 把 可執行檔放在 $GOPATH 下的 /bin，如果沒有會建對應的目錄
```
$ cd ~/go/src/hello
$ go install

$ ls ../../bin
hello

$ ../../bin/hello 
hello, world
```

### 其他可用

- go build: 直接在 .go 所在的 dir 編出可執行檔
```
$ cd ~/go/src/hello
$ go build

$ ls
hello		hello.go
```

- go run: 忽略編譯過程，直接跑出結果
```
$ cd ~/go/src/hello
$ go run hello.go
hello, world!!
```

---

## Reference:
- [golang.org - Getting Started](https://golang.org/doc/install) 
- [初学者没有搞明白的GOROOT,GOPATH,GOBIN,project目录 - python修行路 - 博客园](https://www.cnblogs.com/zhaof/p/7906722.html)
- [关于GOROOT、GOPATH、GOBIN、project目录 - CSDN博客](https://blog.csdn.net/Alsmile/article/details/48290223)
