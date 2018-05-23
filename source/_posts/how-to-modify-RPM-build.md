---
title: 如何修改 RPM build，以 unzip 套件為例
tags:
  - RPM
  - SRPM
  - unzip
date: 2018-05-23 14:40:45
---


今天工作的時候，因為 unzip 有個我想要的 flag 沒有提供，於是就想改個 source code 測試一下，順便學了怎麼上 RPM 的 patch，以下的範例是去修改 unzip 的 warning-error 的 return code ，我希望忽略 warning-error，也就是雖然會寫 warning-error 的 message ，但 return code 是 0，達到忽略 warning 的效果，但其他 error 仍然保留 

## 應用場景

常常在 CentOS 上工作，常常壓 yum 的指令，其實這些包可以透過 yumdownloader 下載源碼，修改上 patch ，再重包，重包完可依需求、licence 送到自己團隊的 yum 源、或是打完補釘貢獻回社區，以下範例雖然以開源的 unzip 套件為範例，可自行替換成其他的 yum package

## Prerequisite

```
$ sudo yum install -y yum-utils rpm-build 
```

確定一下要用的套件名稱，不確的定的話用關鍵字找一下
```
$ yum --showduplicates list unzip
```

然後裝個 dependencies
```
$ sudo yum-builddep -y unzip
```

如果已經裝過該套件，通常會裝 dependencies ，因為我們要改的是 unzip 這個套件，希望只移除 unzip ，不要砍掉 dependencies ，可以這麼做：先找到已安裝套件精確的套件名，然後再刪掉 + --nodeps 的 flag
```
$ rpm -qa | grep unzip
unzip-6.0-19.el7.centos.x86_64

$ sudo rpm -e --nodeps "unzip-6.0-19.el7.x86_64"
```

## 下載並安裝源碼

有關 RPM、SRPM、YUM 的關係，可以參考[鳥哥的 Linux 私房菜 -- 第二十二章、軟體安裝 RPM, SRPM 與 YUM](http://linux.vbird.org/linux_basic/0520rpm_and_srpm.php)，總之，可以知道我們要的是 SRPM，也就 suffix 是 src.rpm 的檔案，剛剛有裝 yum-utils ，應該就有 yumdownloader 了
```
$ yumdownloader --source unzip
```

會自動在家目錄下拉取 latest 的 unzip src.rpm，目前為止我都是在有 sudoer 權限的一般用戶身份下執行
```
$ ls ~/ | grep unzip
unzip-6.0-19.el7.src.rpm
```

安裝 src.rpm ，會在家目錄底下出現 ~/rpmbuild 這個 dir
```
$ rpm -ivh unzip-6.0-19.el7.src.rpm
```

## 編譯源碼

這時候 ~/rpmbuild 下應該只有 SOURCE 和 SPECS ，我們需要從頭編一次
```
$ cd ~/rpmbuild
$ rpmbuild -ba ./SPECS/unzip.spec
```

然後就會多出 BUILD  BUILDROOT  RPMS  SRPMS 這幾個目錄
- BUILD: 放 source code
- BUILDROOT: 空的
- RPMS: 放編好的 rpm
- SRPMS: 放 src.rpm
- SOURCES: 放各種 patch ，基本上就是每次新增 diff 的部分
- SPECS:  編 rpm 所需要的各種設定

## 修改源碼與上 patch

剛剛說到 ~/rpmbuild/BUILD 下面會有 source code ，unzip 套件的 source code 放在 ~/rpmbuild/BUILD/unzip60，由於上 patch 的時候會需要提供新、舊 code 的 diff ，所以我直接 `$ cp -r unzip60 unzip60.new` ，然後在 .new 做完修改，再將 diff 的結果 redirect 到 SOURCES dir 底下。(我要改的 code 只有一行，就是 unzip.h 的 PK_WARN 由 1 改成 0)

```
$ ls BUILD
unzip60  unzip60.new

$ diff -uNr ./BUILD/unzip60 ./BUILD/unzip60.new > ./SOURCE/unzip-6.0-pk_warn.patch  
// patch name 的 unzip-6.0- 跟著大版號，pk_warn 自己取的

$ cat ./SOURCE/unzip-6.0-pk_warn.patch
--- unzip60/unzip.h	2009-02-16 02:12:54.000000000 +0800
+++ unzip60.new/unzip.h	2018-05-22 19:30:27.489247715 +0800
@@ -634,7 +634,7 @@
 /* external return codes */
 #define PK_OK              0   /* no error */
 #define PK_COOL            0   /* no error */
-#define PK_WARN            1   /* warning error */
+#define PK_WARN            0   /* warning error, 1->0 in order to ignore warning */
 #define PK_ERR             2   /* error in zipfile */
 #define PK_BADERR          3   /* severe error in zipfile */
 #define PK_MEM             4   /* insufficient memory (during initialization) */
```

用 diff 的方式我們已經製作好 patch ，接下來就要修改 spec 然後重編

## 修改 spec 並重編

spec 的位置在 ./SPECS/unzip.spec ，修改前可能會想要備份舊的，新的 spec 直接看完整的範例在這 [unzip.spec example for writing RPM building tutorial](https://gist.github.com/yiidtw/bacbe4d1f6acaf3d789914087620dbff)，我修改了四個部分
- 第 4 行，改小版號
- 第 37 行，指明 patch name 與新增 patch 流水號
- 第 79 行，增加這行，source code 裡被改到的檔案，會保留舊版本的一份並在原檔名加上這邊給的 suffix ，例如這邊 source code 就會多一個 unzip.h.pk_warn 的檔案，而原來的 unzip.h 則會在重編後被更新，所以如果等一下重編成功，可以把 .new 的那份 repo 砍掉
- 第 102-103 行，給 changelog

修改得其實不多，我們來編編看
```
$ rpmbuild -ba ./SPECS/unzip.spec
```

一切順利的話，會在 RPMS 下看到新編出來的 rpm
```
$ ls RPMS/x86_64/
unzip-6.0-20.el7.centos.x86_64.rpm  unzip-debuginfo-6.0-20.el7.centos.x86_64.rpm
```

## 安裝與測試

我們可以直接用 rpm 的指令來安裝與移除我們 customized 的 rpm
```
$ sudo rpm -ivh ./RPMS/x86_64/unzip-6.0-20.el7.centos.x86_64.rpm  // 安裝
$ sudo rpm -e --nodeps unzip-6.0-20.el7.centos.x86_64.rpm         // 移除
```

至於測試的方法，先看版本號
```
$ unzip -v
UnZip 6.00 of 20 April 2009, by Info-ZIP.  Maintained by C. Spieler.  Send
bug reports using http://www.info-zip.org/zip-bug.html; see README for details.
```

然後拿新版本的 unzip 去解壓縮檔案，然後可以用 `$ echo $?` 的指令看 return code ，是否有忽略 warning-error 就大功告成啦

## Reference

- [Centos环境下 helloworld rpm的制作,安装,卸载 - CSDN博客](https://blog.csdn.net/qq_26437925/article/details/73691983)
- [Centos下 rpm 打补丁，patch - CSDN博客](https://blog.csdn.net/qq_26437925/article/details/73864258)
- [RPM 安裝/更新/移除套件指令 – Linux 技術手札](https://www.phpini.com/linux/rpm-install-update-remove-package)
- [How to use yum to download a package without installing it](https://access.redhat.com/solutions/10154)
