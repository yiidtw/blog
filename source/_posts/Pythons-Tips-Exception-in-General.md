---
title: Python Tips 通用的例外處理
tags:
  - Python Tips
  - Exception
date: 2019-01-26 16:51:36
---


今天稍微小結了一下 Python 中的例外處理，想找出一個通用好用的 exception 寫法

## sample1.py

最簡單的一種如下
```=python
#!/usr/bin/python

try:
    raise NotImplementedError("Excpetion Testing")
except:
    raise
```

會列出造成例外的 stack 如下
```
Traceback (most recent call last):
  File "sampel1.py", line 4, in <module>
    raise NotImplementedError("Exception Testing")
NotImplementedError: Exception Testing
```

## sample2.py

然後是一樣效果，可是可以自訂 exit code 的例子，這也蠻常用的，尤其是跟 Automation 有關的 script
```=python
#!/usr/bin/python
import sys, traceback

try:
    raise NotImplementedError("Exception Testing")
except:
    traceback.print_exc()
    sys.exit(2)
```

輸出跟上面一樣，然後它的 exit code 是 2
```
$ echo $?
2
```

都 import 了 traceback package ，其實就有很多花招可以玩了，但我個人比較喜歡下面這種

## sample3.py

一樣可以自訂 exit code ，但這次我們 format 一下輸出，好處是打印到 log file 的時候，可以一眼看出哪些部分是自己捕捉到的 exception

```=python
#!/usr/bin/python
import sys, os

try:
    raise NotImplementedError("Exception Testing")
except:
    exc_type, exc_value, exc_traceback = sys.exc_info()
    exc_file = os.path.split(exc_traceback.tb_frame.f_code.co_filename)[1]
    exc_lineno = exc_traceback.tb_lineno
    print(
"""
 %s
 File: %s 
 Line Number: %d
 Error Message: %s
""" % (exc_type.__name__, exc_file, exc_lineno, exc_value)) 
    sys.exit(2)
```

輸出長這樣，高度客製化
```

 NotImplementedError
 File: customize_exc.py 
 Line Number: 5
 Error Message: Exception Testing

```
不過每個 Exception 都要寫是麻煩了點，就自行寫另一個 function 來使用吧

## 結語
- 我個人習慣 error 和輸出都 print 在 stdout ，如果要寫到 log file 再統一由 stdout redirect ，所以就不討論直接輸出到 file 的寫法
- 這邊都預設發生 exception 的時候要被 terminated ，所以也不討論 pass 的情況

## Reference
- [exception - Generic catch for python - Stack Overflow](https://stackoverflow.com/questions/442343/generic-catch-for-python)
- [Python Traceback详解 - 简书](https://www.jianshu.com/p/a8cb5375171a)
- [28.10. traceback — Print or retrieve a stack traceback — Python 2.7.15 documentation](https://docs.python.org/2/library/traceback.html#traceback-examples)
- [Python When I catch an exception, how do I get the type, file, and line number? - Stack Overflow](https://stackoverflow.com/questions/1278705/python-when-i-catch-an-exception-how-do-i-get-the-type-file-and-line-number)
