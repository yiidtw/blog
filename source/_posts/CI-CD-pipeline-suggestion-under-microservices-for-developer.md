---
title: 微服務下 RD 的 CI/CD 流水線
date: 2018-06-12 16:47:59
tags:
- Continous Integration
- Continous Delivery
- CI/CD
- Kubernetes
- Jenkins
- Github flow
- Git flow
---

最近要 refactor 組上的 CI/CD pipeline ，花了一些時間研究一下，目前是針對開發人員的部分，流水線的端到端，分別是開發者，與交付有質量保證的 docker build 與相應能在 Kubernetes 跑進來的 config。某種程度上 Kubernetes 成為事實上的標準 CD 環境，於是 Git(版本管理) + Jenkins(CI server) + Kubernetes(CD target env) 的組合，目前應該是個很熱門的組合

---

## Mindset

對 best practices 有興趣的同學可以參考這篇文章 [An Introduction to CI/CD Best Practices | DigitalOcean](https://www.digitalocean.com/community/tutorials/an-introduction-to-ci-cd-best-practices) ，對我們來說，有以下的總結
- 盡早測試，速度快的測試先測
- 測試失敗，能盡速修正，包括重新出 build
- 縮短 [失敗-修正] itertaion 的時間
- main repo 的分支盡量少
- 在 pipeline 中增加 Observability，可以直接觀察到各階段的結果(成功或失敗)

Obervability ，可以降低大家的 workload ，RD/QA 不必每個人都熟 pipeline 的工具或 Kubernetes ，可以直接看到 build fail 的 log 或是 service 的 debug log ， DevOps 也不用常常 ssh 或 kubectl 到目標機器，這事後面慢慢補說

## Tesing Edge

在看 flow 之前，我們需要先定義 testing edge ，包括
- 以 edge 的觀念去區分各階段測試需要涵蓋的範圍
- 適當的分工給 RD、QA ，精確的以 test case 定義交付的 docker image 質量

設計好的 pipeline 好像是挖洞，以 automation 設立 checkpoint ，以測試的角度來看 RD/QA 只要專注在寫 test case ，填到對應的洞， pipeline 就會自動跑完 test case ，並給出人可閱讀的 report

![testing edge](https://i.imgur.com/eLwbaUx.png)

這是我司的 testing edge 定義，每個 team 對於不同的 product ， testing edge 的定義可能不一樣，參考了這篇文章 [Microservice Testing: Unit Tests – Nathan Peck – Medium](https://medium.com/@nathankpeck/microservice-testing-unit-tests-d795194fe14e) ，我認為倒三角更符合我對 testing 的想像， test cases 應該越來越收斂，耗時越久的越晚執行，然後盡早失敗。我司的 RD 的 pipleline 會需要負責上面三個 testing edge 涵蓋的內容
- Unit Tests
	- function, class level 的測試，有 language dependency
- Smoking Tests
	- 依照 RD deliver 的 code、config、Dockerfile 包一個 docker image ，看能不能 build success
	- 跑過 test folder 底下的 test cases
	- 用 mock client 與該 container 互動，打 test case，部分 mock client 可能直接用 docker compose 起一個真的，例如 mysql ，比設計假的還方便
- Integration Tests
	- container landing on Kubernetes ，測試各個微服務的可達性與 config 是否正確

## Flow

![pipeline](https://i.imgur.com/g9ETNnI.png)

可以看到我們除了用，Git + Jenkins + Kubernetes ，還用了 Harbor ，可以參考這篇文章 [Harbor™ by VMware®](http://vmware.github.io/harbor/)，可以想像就是 docker image 的 git ，用起來也很爽，就先不描述了

這個圖還有隱含著我們使用 Github flow + Git flow 去協作，RD 要 fork main repo 的 master branch 到自己的 repo ，所有進 code 都要透過 PR 的方式，經過 CI 的 smoking test 和人為的 code review，而這些結果和人的意見會整合在該 PR 的 issue ，讓 RD head 去決定要不要 merge 這個 PR，可以參考這篇文章 [GitHub Flow 及 Git Flow 流程使用時機 | 小惡魔 - 電腦技術 - 工作筆記 - AppleBOY](https://blog.wu-boy.com/2017/12/github-flow-vs-git-flow/)

RD head 想要出 build 的時候，就下一個 git tag ，透過 web hook trigger 新的 build，直接部署到 Kubernetes ，我司目前是部署到 AWS，之後預期會有幾篇文章和 code snippet 針對這個架構作 POC 再詳述

以上，待續
