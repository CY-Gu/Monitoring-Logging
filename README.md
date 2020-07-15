---
title: Logging & Monitoring
tags: kubernetes, k8s, Monitoring, Logging
---

:::warning
發生 ```The connection to the server localhost:8080 was refused - did you specify the right host or port?``` 時的處理方式
```=
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
:::

查看k8s cluster資訊
```bash=
$ kubectl config view
```
```bash=
$ kubectl cluster-info
```

# Monitoring & Logging

## k8s cluster monitor
k8s具備基本的伺服器監控工具，主要工具如下：

*    **K8s Dashboard**：插件工具，展示每個k8s集群上的資源利用情況，也是實現資源和環境管理與交互的主要工具。
*    容器探針：container健康狀態診斷工具。
*    **Kubelet**：每個Node上都運行著Kubelet，監控container的運行情況。Kubelet也是Master與各個Node通信的渠道。Kubelet能夠直接暴露cAdvisor中與container使用相關的個性化指標數據。
:::warning
kubelet default 監聽的port是10250，所以我們可以在Master或Slave上直接訪問
```=
curl https://127.0.0.1:10250/metrics/cadvisor -k
```
*    需使用 https
*    metrics/cAdvisor 是 kubelet pod 相關的監控指標，它還有一個 metrics，是 kubelet 自身的監控指標
*    ```-k``` 表示不驗證 kubelet 證書，因整個叢集都是使用自簽署證書，因此沒必要驗證
::: 
*    **cAdvisor**：開源的單節點agent，負責監控容器資源使用情況與性能，採集機器上所有container的memory、網絡使用情況、文件系統和CPU等數據。
:::info
cAdvisor雖然好用，但有些缺點：
*    僅能監控基礎資源利用情況，無法分析應用的實際性能
*    不具備長期存儲和趨勢分析能力。
:::

*    **Kube-state-metrics**：輪詢Kubernetes API，並將Kubernetes的結構化信息轉換為metrics。

除了上述基本監控工具外，還有一些功能更多，更好用的Monitor，下面列舉幾個。


## Monitor

:::success

*    [Heapster (Matric-Server的原版)](https://github.com/kubernetes-retired/heapster)
*    [Matric-Server](https://github.com/kubernetes-sigs/metrics-server)
*    [Prometheus](https://prometheus.io/)
*    [Elastic Stack](https://www.elastic.co/cn/products/)
*    [DataDog](https://www.datadoghq.com/)
*    [Dynatrace](https://www.dynatrace.com/)

:::



### Metric-Server
一個 k8s cluster可以有一個 Metric-Server監控資源，包括監控Node或Pod資訊。但須注意的是，Metric-Server是 **in memory** 的 monitor，他不會將監控的資料儲存到disk，所以無法獲得過去的監控資料。

#### Metric-Server Install

**Step 1**
```bash=
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
```
創建完成後會多一個deploy
```bash=
╰──➤  kubectl get deploy -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
coredns          2/2     2            2           5d20h
metrics-server   1/1     1            1           2d11h
```
用```kubectl top no <node-name>```查看，會發現有error，這個error可能是因為node名稱找不到，或是CA認證沒過 (通常是這兩種原因)，這時只要在metrics-server deploy 加上一些參數就可以了
```bash=
$ kubectl edit deploy metrics-server -n kube-system

...

      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server-amd64:v0.3.5
        ## 加入下面這些command
        command:
        - /metrics-server
        - metrics-resolution=30s
        - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
        - --kubelet-insecure-tls
...
```
檢查一下，發現可以了
```bash=
╰──➤  kubectl top no        
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
g8master   188m         4%     1827Mi          23%       
g8node1    83m          4%     2201Mi          57%       
g8node2    72m          3%     1607Mi          41% 
```


#### Monitor 如何監控

我們都知道在Slave node中是透過```kubelet```來管理node，包括透過 apiserver 接收Master的指令以及在node中運行Pod等。而kubelet中還包含一個subcomponent，稱為  **cAdvisor** 或是 **Container Advisor** ，cAdvisor 負責從Pod中擷取performance metrics，並將收集到的數據以metrics-api的形式，透過Summary API expose給Metric-Server。




## Logging

查看Pod內的log可用指令 ```kubectl logs -f <pod-name>``` ，例如
```=
george@kmaster:~$ kubectl logs -f demopod
[2020-03-13 09:11:16,540] INFO in event-simulator: USER1 logged out
[2020-03-13 09:11:17,541] INFO in event-simulator: USER1 is viewing page3
[2020-03-13 09:11:18,543] INFO in event-simulator: USER4 is viewing page3
[2020-03-13 09:11:19,545] INFO in event-simulator: USER3 logged in
[2020-03-13 09:11:20,547] INFO in event-simulator: USER2 is viewing page1
...
```
```-f```參數是讓log不停輸出最新的event。要注意若Pod內有兩個以上的container，要在指令後方加上 container name，否則會發生error。
```=
kubectl logs -f <pod-name> <container-name>
```

---

## 查看API

可用```curl http://localhost:6443 -k```查看API，但會發現
```bash=
$ gmaster@gmaster:~/k8s_demo/6-ingress$ curl http://localhost:6443 -k 
Client sent an HTTP request to an HTTPS server.
```
無法存取到，這是因為沒有認證機制輔助。可以先用```kubectl proxy```，再透過proxy的8001 port連進去查看。另外開啟一個tmux，輸入```curl http://localhost:8001 -k```就看得到了
![](https://i.imgur.com/UvyTqRZ.png)
:::success
```kubectl proxy```是由kubectl utility 創建的 http proxy service，用來存取kube-apiserver
:::

---

### Thank you! :sheep: 

You can find me on

- george4908090@gmail.com