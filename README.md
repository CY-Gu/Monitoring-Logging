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


# Monitoring & Logging

## Monitor

:::success
*    [Heapster (Matric-Server的原版)](https://github.com/kubernetes-retired/heapster)
*    [Matric-Server](https://github.com/kubernetes-sigs/metrics-server)
*    [Prometheus](https://prometheus.io/)
*    [Elastic Stack](https://www.elastic.co/cn/products/)
*    [DataDog](https://www.datadoghq.com/)
*    [Dynatrace](https://www.dynatrace.com/)
:::

### Monitor 如何監控




#### Matric-Server
一個 k8s cluster可以有一個 Matric-Server監控資源，包括監控Node或Pod資訊。但須注意的是，Matric-Server是 **in memory** 的 monitor，他不會將監控的資料儲存到disk，所以無法獲得過去的監控資料。


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
```-f```參數是讓log不停輸出最新的event，要注意，若Pod內有兩個以上的container，要在指令後方加上container name，不然會error
```=
kubectl logs -f <pod-name> <container-name>
```


---

### Thank you! :sheep: 

You can find me on

- george4908090@gmail.com