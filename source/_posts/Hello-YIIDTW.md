---
title: 本站首Po
date: 2018-04-22 22:50:38
tags:
---

用 Hexo 重架了一次 blog ，有幾項舊站就想要修改的設定
- URL 設定：/year-month-date-title
- 用同一個 repo 管理 source(.md) 和 redering result(.html)
- 由於 Next 主題在不同功能環境中會用 plugin 的方式安裝，於是 next 的 _config.yaml 會在一級目錄用 _config.yml.next.example 來維持，並 override 各環境中安裝後 Next 主題的設定檔

---

## 寫草稿再發佈

```
$ hexo new draft "article title" // 建立草稿
$ hexo s --draft // 在本地預覽
$ hexo publish "不含 md 的草稿 file name" // 把草稿從 _draft 移到 _posts
```

---

## 超好用工具推薦

常寫 markdown 的人，一定會遇到要把網址轉成 markdown 格式這件事，推薦使用 tabcopy 的 chrome extension ，[連結在此](https://chrome.google.com/webstore/detail/tabcopy/micdllihgoppmejpecmkilggmaagfdmb?hl=zh-TW)，而且推薦從預設的 FANCY MODE 調成 SIMPLE MODE，快捷鍵是 Alt+C ，可以直接在網頁視窗上一鍵複製網站標題和 URL ，而且是 markdown 格式，再貼到文字編輯器

---

## 常用連結

- [Hexo 集成 Disqus 评论](http://www.cylong.com/blog/2017/03/26/hexo-next-disqus/)
- [为 Next 主题添加文章阅读量统计功能 - LeanCloud](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)
    - LeanCloud Dashboard -> 设置 -> 安全中心-> Web 安全域名：要加上 blog 的 domain ，例如說 https://yiidtw.github.io
    - Next 的 `_config.yml` leancloud 部分 security 設為 false
- [如何使用 Hexo 寫草稿 ? | 點燈坊](http://oomusou.io/hexo/draft/)

TODO:
- 測試 gitment(懶)
