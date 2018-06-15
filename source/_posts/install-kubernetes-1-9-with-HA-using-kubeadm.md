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
![](https://d33wubrfki0l68.cloudfront.net/2555d34e3008aab4b049ca5634cfabc2078ccf92/3269a/images/docs/ha.svg)
[photo credit](https://kubernetes.io/docs/admin/high-availability/building/)

Kubernetes cluster 的概念筆記
- 整個 Kubernetes cluster 分成多個 masters 和多個 nodes(workers) ，在 AWS 的環境中我們會使用 AWS ELB 作為 master nodes 的 load balancer ，如果是 on premise 的情況，可以使用 keepalived 的實作
- 為了安全起見，我們的 masters 不參與工作負載
- kubectl 會在 cluster 外下達各種 control 指令
- master 的重要元件
	- kube-apiserver 
	- kube-scheduler
	- kube-controller-manager
- node 的重要元件
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

借別人的參考圖用
![](https://cdn-images-1.medium.com/max/1200/1*NbqO1_Lj74b38g-hq3OxPg.png)
[photo credit](https://medium.com/@bambash/ha-kubernetes-cluster-via-kubeadm-b2133360b198) 

不過這張圖少了 jumpy，jumpy 必須要能免密碼 ssh 到每個 ec2 instance
我們會有 7 個 ec2 instance
- jumpy
- masters
    - k8s-m0
    - k8s-m1
    - k8s-m2
- nodes
    - k8s-n0
    - k8s-n1
    - k8s-n2

---

## STEP 1: kubeadm minimum requirements

我們主要參考的官方文件是這篇[Creating HA clusters with kubeadm](https://kubernetes.io/docs/setup/independent/high-availability/)，依照建議，我們需要每個 nodes (包括 masters) 都 apply kubeadm minimum requirements，相關的 guide line 在這 [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin)，以下的指令都是用 root 身份執行
- 所有的 masters, nodes 都要安裝且滿足 kubeadm minimum requirements
- jumpy 上只須安裝 kubectl，如果為了產生 kubeadm 的 token 方便，也可以裝個 kubeadm

### network
```
$ cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ modprobe br_netfilter
```

然後方便起見，而且佈署的環境在 AWS 內部，node 不會暴露在公網，所以我們把 firewalld 直接關掉，或是直接乾脆不裝，如果要啟用 firewalld ，要開的 port 請參考官網 
```
$ systemctl stop firewalld
```

### 關閉 swap
```
$ swapoff -a
$ echo "vm.swappiness=0" >> /etc/sysctl.d/k8s.conf
$ sysctl --system
```

### 安裝 docker 17.03.2.ce
```
$ yum remove docker docker-common docker-selinux docker-engine
$ yum install -y yum-utils device-mapper-persistent-data lvm2
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum install -y --setopt=obsoletes=0 docker-ce-17.03.2.ce-1.el7.centos docker-ce-selinux-17.03.2.ce-1.el7.centos
$ iptables -P FORWARD ACCEPT
$ systemctl enable docker && systemctl start docker
```

### 安裝 kubelet, kubeadm, kubectl 1.9.1
```
$ cat <<EOF > /tmp/kubernetes.repo 
[kubernetes] 
name=Kubernetes 
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch 
enabled=1 
gpgcheck=1 
repo_gpgcheck=1 
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg 
EOF
$ mv /tmp/kubernetes.repo /etc/yum.repos.d/kubernetes.repo
$ yum install -y kubelet-1.9.1 kubeadm-1.9.1 kubectl-1.9.1
```

這裡要把 kubeadm 預設用的 cgroup driver 改成跟 dockerd 用的 cgroupfs 一樣
```
$ sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
$ systemctl daemon-reload && systemctl enable kubelet && systemctl start kubelet
```

### 關閉 selinux 再重啟 EC2 instance
```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sed -i 's/SELINUX=permissive/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
reboot
```

---

## STEP 2: 建立 ETCD cluster

(2018/06/15 更新)

ETCD 就像是 zookeeper 一類 key-value 的 store，在 Kubernetes cluster 裡扮演的是紀錄整個 cluster 狀態的角色，可以以 daemon 方式佈署、以 static pod 的方式佈署，或是佈署在其他的 EC2 instances。官方教學有教 daemon 和 pod 的兩種佈署方式，但是 pod 的佈署方式有點像雞生蛋蛋生雞的問題，如果 ETCD cluster 沒起來，那 Kubernetes 怎麼起來？可能要先佈一個 ETCD daemon -> 起 Kubernetes cluster -> 在 Kubernetes cluster 裡起 ETCD cluster -> 資料從 K8S cluster 外的 ETCD daemon migrate 到 K8S cluster 內的 ETCD cluster ，用手動的方式想到就覺得麻煩…所以我們這邊會直接用 daemon 的方式佈署 ETCD cluster，分別在 3 個 masters 上

首先要下載 cfssl 和 cfssl 工具，我們會在 bastion 上產生 etcd 所要用的 key ，再丟到 master node 上
```
$ mkdir -p ~/bin
$ curl -o ~/bin https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
$ curl -o ~/bin https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
$ chmod +x ~/bin/cfssl
```

我們要先建立 etcd 要吃的 config 檔，分別是 ca-config.json 、ca-csr.json、client.json，找個 dir 放，例如 bastion:~/etcd/common
```
$ mkdir -p ~/etcd/common
```

下面的 json 檔請依自己的需求修改，請參考 [Creating HA clusters with kubeadm - Kubernetes](https://kubernetes.io/docs/setup/independent/high-availability/)
```
cat > ~/etcd/common/ca-config.json <<EOF
{
   "signing": {
       "default": {
           "expiry": "43800h"
       },
       "profiles": {
           "server": {
               "expiry": "43800h",
               "usages": [
                   "signing",
                   "key encipherment",
                   "server auth",
                   "client auth"
               ]
           },
           "client": {
               "expiry": "43800h",
               "usages": [
                   "signing",
                   "key encipherment",
                   "client auth"
               ]
           },
           "peer": {
               "expiry": "43800h",
               "usages": [
                   "signing",
                   "key encipherment",
                   "server auth",
                   "client auth"
               ]
           }
       }
   }
}
EOF
```

```
cat > ~/etcd/common/ca-csr.json <<EOF
{
   "CN": "etcd",
   "key": {
       "algo": "rsa",
       "size": 2048
   }
}
EOF
```

```
cat >~/etcd/common/client.json <<EOF
{
  "CN": "client",
  "key": {
      "algo": "ecdsa",
      "size": 256
  }
}
EOF
```

接下來產生 etcd 要用的 key
```
$ cd ~/etcd/common
$ ~/bin/cfssl gencert -initca ca-csr.json | ~/bin/cfssljson -bare ca -
$ ~/bin/cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client.json | ~/bin/cfssljson -bare client
```

這時候會產生所需要的各種 pem 和 key
- ca.pem
- ca-key.pem
- client.pem
- client-key.pem
連同之前的 ca-config.json ，我們會需要這五個檔案，加上下面的 config.json ，各 6 個檔案，對 3 個 etcd 產生所需要的檔案，只有 config.json 是不同的，其他 5 個檔案在 3 個 etcd server 上是相同的，以下是 config.json 的範例
```
cat >~/etcd/common/config.json <<EOF
{
    "CN": "HOSTNAME_OF_ETCD0",
    "hosts": [
        "127.0.0.1",
        "IP_OF_ETCD0",
        "IP_OF_ETCD1",
        "IP_OF_ETCD2",
        "HOSTNAME_OF_ETCD0",
        "HOSTNAME_OF_ETCD1",
        "HOSTNAME_OF_ETCD2"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "TW",
            "L": "Taipei",
            "ST": "Taipei"
        }
    ]
}
```

可以看到主要的差別在 CN ，你應該會想開三個子資料夾，cp 剛剛 6 個檔案，然後執行下面的指令
```
$ ~/bin/cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server config.json | ~/bin/cfssljson -bare server
$ ~/bin/cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer config.json | ~/bin/cfssljson -bare peer
```

將這 3 個子資料夾下產生的所有檔案，各自 scp 到 3 台 etcd server 的 /etc/etcd/ssl，然後在這 3 台 etcd server 分別執行
```
$ cd ~; curl -sSL https://github.com/coreos/etcd/releases/download/v3.1.10/etcd-v3.1.10-linux-amd64.tar.gz \
 | sudo tar -xzv --strip-components=1 -C /usr/local/bin/; \
$ sudo chown root:root /usr/local/bin/etcd*; \
$ sudo systemctl daemon-reload && sudo systemctl enable etcd && sudo systemctl restart etcd
``` 

這樣， etcd 就裝完了 XD

這部分比較繁瑣，而且 ETCD cluster 不僅可以用在紀錄 Kubernets 的狀態，也可以用在 general 需要 Key-Value 的場景。需要注意的是，如果需要 reset kubeadm ，由於 ETCD cluster 若以 daemon 的方式佈署在 K8S cluster 之外，那執行 kubeadm reset 之後，需要先停掉 3 台 ETCD cluster -> 清除各自 /var/lib/etcd/member 下的所有資料 -> 重啟 ETCD cluster ，這樣原先的 K8S cluster 資料才會被完全清除

---

## STEP 3: 建立第一台 master

首先，我們要先產生 kubeadm 所需的 token，寫在 config.yaml 裡
```
$ kubeadm token generate
6fd455.dd6fb4fa31f47c93
```

config.yaml 會長得像這樣
```
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
token: "6fd455.dd6fb4fa31f47c93"
tokenTTL: "0"
etcd:
  endpoints:
  - https://IP_OF_K8S_M0:2379
  - https://IP_OF_K8S_M1:2379
  - https://IP_OF_K8S_M2:2379
  caFile: /etc/etcd/ssl/ca.pem
  certFile: /etc/etcd/ssl/client.pem
  keyFile: /etc/etcd/ssl/client-key.pem
networking:
  podSubnet: 10.244.0.0/16
kubernetesVersion: 1.9.1
apiServerCertSANs:
- 127.0.0.1
- IP_OF_K8S_M0
- IP_OF_K8S_M1
- IP_OF_K8S_M2
- HOSTNAME_OF_K8S_M0
- HOSTNAME_OF_K8S_M1
- HOSTNAME_OF_K8S_M2
- AWS_ELB_DNS_NAME_FOR_MASTER_NODES
apiServerExtraArgs:
  endpoint-reconciler-type: lease
```
寫好 config.yaml 之後，scp 到三個 masters 的 /etc/kubernetes/

其中 podSubnet 如果跟我一樣用的是 flannel 的 CNI plugin ，就必須填得一樣，這邊要特別注意的是 AWS_ELB_DNS_NAME_FOR_MASTER_NODES 必須是全部小寫，而且可以是 internal 的 ELB，要嘛就在給 ELB 取名時全部小寫，要嘛就用 route53 給一個小寫的名字，當 etcd cluster healthy、config.yaml 寫好，dependency 裝好，就可以來起第一台的 master
```
$ ssh k8s-m0 "sudo kubeadm init --config=/etc/kubernetes/config.yaml"
```

跑了一堆訊息後，如果成功之後會給類似下面的訊息
```
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 6fd455.dd6fb4fa31f47c93 IP_OF_K8S_M0:6443 --discovery-token-ca-cert-hash sha256:SOME_VERY_LONG_HASH
```

kubeadm join 這串很重要，要記下來。這個時候，可以去 AWS dashboard 確認給 masters 用的 ELB 是否設置完成，health check 有沒有過，然後可以把 k8s-m0 上的 /etc/kubernetes/admin.conf scp 到 jumpy 上的 ~/.kube/config ，再把 config 裡面 k8s-m0 的 IP 換成 ELB 的 dns name，這樣就可以在 jumpy 上使用 kubectl 操控整個 Kubernetes cluster，kubectl 設置好可以測試一下

```
$ kubectl get po --all-namespaces
```

這個時候也可以順便 apply 一下 Kubernets 要用的 network plugin
```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```

建議 apply 之後，再 get all pods 看一下 flannel 和 kube-dns 有沒有正常跑起來，如果有，恭喜！

---

## STEP 4：建立其他台 master

剛剛建立第一台 master 的 config.yaml 應該也 scp 到剩下兩台 masters 了，然後我們在 jumpy 上的 kubectl 應該也設好了，flannel 也健康地跑起來了，之所以要先跑第一台 master ，就要要將 kube-api 的 key sync 到其他兩台，請自行將下面四個檔案 scp 從 k8s-m0 到 k8s-m1、k8s-m2 的同樣位置

```
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/sa.key
/etc/kubernetes/pki/sa.pub
```

包含
```
/etc/kubernets/config.yaml
```

3個 masters 應該有五個檔案要一致，這時候再對剩下兩台 masters 執行
```
$ ssh k8s-m1 "sudo kubeadm init --config=/etc/kubernetes/config.yaml"
$ ssh k8s-m2 "sudo kubeadm init --config=/etc/kubernetes/config.yaml"
```

回頭檢查一下
- ELB 對 3 個 mastes 的 health check
- 所有 pods 尤其是 kube-dns 和 flannel 是不是正常

---

## STEP 5: 加入 nodes

應該還記得剛剛說要把 kubeadm join 記下來吧，加入 nodes 就非常 trivial 了，直接在每一個要加入負載的 nodes 上執行
```
$ sudo kubeadm join --token 6fd455.dd6fb4fa31f47c93 AWS_ELB_DNS_NAME_FOR_MASTER_NODES:6443 --discovery-token-ca-cert-hash sha256:SOME_VERY_LONG_HASH
```

以上，即使是半自動的建立 Kubernetes 還是相當麻煩的，希望大家都建立成功
