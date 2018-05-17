---
title: 本站首Po
date: 2018-04-22 22:50:38
tags:
---

用 Hexo 重架了一次 blog ，有幾項舊站就想要修改的設定
- URL 設定：/year-month-date-title
- 用同一個 repo 管理 source(.md) 和 redering result(.html)
- 由於 Next 主題在不同功能環境中會用 plugin 的方式安裝，於是 next 的 _config.yaml 會在一級目錄用 _config.yml.next.example 來維持，並 override 各環境中安裝後 Next 主題的設定檔

補個常用連結
- [Hexo 集成 Disqus 评论](http://www.cylong.com/blog/2017/03/26/hexo-next-disqus/)
- [为 Next 主题添加文章阅读量统计功能 - LeanCloud](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)
    - LeanCloud Dashboard -> 设置 -> 安全中心-> Web 安全域名：要加上 blog 的 domain ，例如說 https://yiidtw.github.io
    - Next 的 _config.yml leancloud 部分 security 設為 false

TODO:
- 測試 gitment(懶)
