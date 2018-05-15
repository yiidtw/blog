---
title: 使用 kubeadm 安裝 Kubernetes 1.9 高可用 cluster
date: 2018-05-15 13:48:12
tags:
- Kubernetes
- kubeadm
- High Availability
---

最近在 AWS 用上了 Kubernetes ，試了幾套工具之後，決定使用 kubeadm 來安裝高可用群集。但是，請注意 kubeadm 仍然在 beta 階段，如果要嘗試，[官方文件](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)建議[定期備份 etcd](https://coreos.com/etcd/docs/latest/v2/admin_guide.html)

在寫這篇文章的時候 kubeadm 還不直接支援 HA ，必須手動配置，[官方文件](https://kubernetes.io/docs/setup/independent/high-availability/)照著做會踩到坑，這篇就稍微說一下踩坑的經歷。除了 kubeadm，還 survey 過幾種部署方式：

- [kops](https://github.com/kubernetes/kops)
- [kubespray](https://github.com/kubernetes-incubator/kubespray)
- [rancher](https://rancher.com/)
- [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

Kubernetes 佈署在哪裡還蠻重要的，比如說我們佈署在 AWS ，要讓 Kubernetes 去認得 AWS 的 Native Service ，事前要對 EC2 的 instance 、 IAM role 進行許多設定，這裡我用過 kubeadm、kubespray、rancher ，實驗的環境是 vSphere 和 AWS，我的建議是：

- 如果 Kubernetes 只需要長在雲端，而且你是新手沒什麼研究的時間，卻要部署 production grade 的 Kubernetes，那可以用 kops
- 如果你對 Ansible 很熟，而且需要 deliver both on premise and cloud ，那可以考慮 kubespray
- 如果要同時管理不同 orchestration tool ，例如 Kubernetes 和 mesos ，那可以考慮 rancher

對我而言，我希望雲端和 on premise 是同一套佈署方式，所以我採用 kubeadm ，除此之外，我在使用 kubespray 和 rancher 的時候，分別遇到了 worker node 無法 scale 還有 rancher 管理用的 pod 起太多吃我太多資源的問題，也許有人用這兩個工具用得很愉快，不過我對 kubespray 和 rancher 已失去耐心，然後 kops 需要的 prerequisite 太多，包括預先就要準備 cluster 用的 DNS name 和額外需要 S3 bucket 儲存 cluster 所需要的設定檔，還有 auto scaling group 、各種 IAM role 和 tag 設定，也讓我對 kops 興致缺缺， kops 看起來更像 Kubernetes + Cloud infra 的全家桶，而我僅需要一個協助我部署 Kubernetes 、有效且可靠的中間件，於是我後來選擇了 kubeadm

---

## 預備知識

當然首先要大概了解一下 Kubernetes 的架構，先看以下這張圖
![](https://images2015.cnblogs.com/blog/892532/201705/892532-20170530111847102-1480506150.png)
[photo credit](http://www.itread01.com/content/1496132408.html)

- 整個 Kubernetes cluster 分成多個 masters 和多個 nodes(workers) ，在 AWS 的環境中我們會使用 AWS ELB 作為 master nodes 的 load balancer ，如果是 on premise 的情況，可以使用 keepalived 的實作
- 為了安全起見，我們的 masters 不參與工作負載
- kubectl 會在 cluster 外下達各種 control 指令
- master components
	- kube-apiserver 
	- kube-scheduler
	- kube-controller-manager
- node components
	- kubelet: pod 的管家，管理 pod 的 life-cycle
	- kube-proxy
- etcd
	- 紀錄整個 cluster 最重要的 key-value store
	- go 實作、[Raft 算法](http://thesecretlivesofdata.com/raft/)
	- 可以選擇放在 masters 上、以 static pods 放在 Kubernetes，或是另外建 cluster
- 使用 kubeadm 佈署的話， kubeadm 會安裝在每個節點，both masters and nodes
- [Reference](https://kubernetes.io/docs/concepts/overview/components/)

---

## Component Version

- CentOS 7.4
- Kubernetes 1.9.1
- ETCD 3.1.10
- kubeadm 1.9.1
- kubelet 1.9.1
- kubectl 1.9.1
- docker 17.03.2-ce

---

## Create AWS Environment

這部分請依照自己的習慣建立，無論是從 UI 上選擇、使用 AWS Cloudformation，或是 Terraform ，我們會需要
- 1 個 t2.micro 作為 kubectl 發號施令和佈署用的堡壘機(jumpy)
- 3 個 t2.medium 作為 masters
- 3 個 最少 t2.medium 作為 nodes (依照 workload 自行決定 instance type)
- 1 個 ELB 作為 masters 的 load balancer
- 1 個 ELB 作為服務的 load balancer ，我們會接在 nodes 的 NodePort 上
- [WARNING] 這種部署方法不需要 AWS 作 IAM role 或 tag 的額外設定

---

## kubeadm minimum requirements

我們主要參考的官方文件是這篇[Creating HA clusters with kubeadm](https://kubernetes.io/docs/setup/independent/high-availability/)，依照建議，我們需要每個 nodes (包括 masters) 都 apply kubeadm minimum requirements，相關的 guide line 在這 [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin)


