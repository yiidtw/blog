---
title: 在 Mac book 上設置 Golang 的開發環境
tags:
- Golang
- Mac book
---

最近要來用 Golang 開發一些東西了，先寫個 setup

## 跟 Golang 有關的重要環境變量

### GOROOT
- golang 安裝路徑，我直接用官方的 installer (.pkg) 安裝，預設會在 /usr/local/go，若客製化需要可更改
- 如果 shell 的 auto complete 抓不到 go ，例如我某台 mac 上用 zsh ，那可以在 .bash_profile 新增 alias go="/usr/local/go/bin/go"，記得 source

### GOPATH
- 預設在 $HOME/go ，若客製化需要可更改
- 通常會有三個 dir ，分別是 src、pkg、bin ，我們只需要手動創建 src ，像是 `$ mkdir -p $HOME/go/src` ，其他兩個會在 go install 有需要時自動創建

### GOBIN
- 基本上不設，可為空，若客製化需要可更改

### 其他
- 可用 `$ go env` 查看 golang 用到相關的環境變數
- project 要用到的版本管理 a.k.a. git ，應該在 src 下，一個 project 一個 dir，詳見下段工程目錄結構

---

## 工程目錄結構

---

## 編譯選項

### 推薦用這個

- go install: 把 可執行檔放在 $GOPATH 下的 /bin，如果沒有會建對應的目錄
```
$ cd ~/go/src/hello
$ go install

$ ls ../../bin
hello
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