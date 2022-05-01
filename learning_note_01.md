In this Lesson, we will cover the informations about `kubectl get`

## Some Basic Operation of Get

```
$ kubectl get ns
NAME                    STATUS   AGE
default                 Active   4d12h
kube-node-lease         Active   4d12h
kube-public             Active   4d12h
kube-system             Active   4d12h
locust-test-namespace   Active   4d7h
monitoring              Active   4d12h
```
Get Namespace

```
$ kubectl get pods -n kube-system
NAME                                         READY   STATUS              RESTARTS   AGE
azure-ip-masq-agent-rswls                    1/1     Running             0          25s
cloud-node-manager-6fk75                     1/1     Running             0          25s
coredns-69c47794-9f5g4                       0/1     Pending             0          35s
coredns-69c47794-lhhzw                       0/1     Pending             0          35s
coredns-autoscaler-7d56cd888-xn7fg           0/1     Pending             0          34s
csi-azuredisk-node-jdjnf                     3/3     Running             0          25s
csi-azurefile-node-f2tjr                     0/3     ContainerCreating   0          25s
dashboard-metrics-scraper-5545f488c6-q7gkd   0/1     Pending             0          16m
kube-proxy-cvwsp                             1/1     Running             0          25s
kubernetes-dashboard-dc7976c89-b75p4         0/1     Pending             0          34s
metrics-server-6576d9ccf8-2q8hl              0/1     Pending             0          33s
tunnelfront-94444f59f-m7brn                  0/1     Pending             0          34s
```
Get Pods

```
$ kubectl -n kube-system describe pod kube-proxy-cvwsp
Name:                 kube-proxy-cvwsp
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 aks-controller-31613468-vmss00000j/10.240.0.4
Start Time:           Sat, 30 Apr 2022 00:51:28 +0800
Labels:               component=kube-proxy
                      controller-revision-hash=fbc56b478
                      pod-template-generation=2
                      tier=node
Annotations:          <none>
Status:               Running
IP:                   10.240.0.4
IPs:
  IP:           10.240.0.4
Controlled By:  DaemonSet/kube-proxy
Containers:
  kube-proxy:
    Container ID:  containerd://afb4fd0fdee3f1d0e6c0e799336fddbd14f0f5026a1713a78ce52f6a0f913d0b
    Image:         mcr.microsoft.com/oss/kubernetes/kube-proxy:v1.22.6-hotfix.20220330.2
    Image ID:      mcr.microsoft.com/oss/kubernetes/kube-proxy@sha256:7bf8f4acbecf87cf243ae241f6a042bb28c3b13fe23f1cedafa9d956377d9da9
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-proxy
      --metrics-bind-address=0.0.0.0:10249
      --kubeconfig=/var/lib/kubelet/kubeconfig
      --cluster-cidr=10.244.0.0/16
      --v=3
    State:          Running
    ... ... ...
```
Get a description of a pod

## Full Pod Information and Selectors

```
$ kubectl -n kube-system get pod kube-proxy-cvwsp -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-04-29T16:51:16Z"
  generateName: kube-proxy-
  labels:
    component: kube-proxy
    controller-revision-hash: fbc56b478    
    pod-template-generation: "2"
    tier: node
  name: kube-proxy-cvwsp
  namespace: kube-system
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: DaemonSet
    name: kube-proxy
    uid: 616f2587-c8d6-441f-8416-8eb47cee9d90
  resourceVersion: "2185517"
  uid: 24d67e5c-d6c3-44dd-b094-e71445372587
  ...
  ...
status:
  ...
  ...
  hostIP: 10.240.0.4
  phase: Running
  podIP: 10.240.0.4
  podIPs:
  - ip: 10.240.0.4
  qosClass: Burstable
  startTime: "2022-04-29T16:51:28Z"
```
Get Yaml based pod information, full information, change `-o yaml` to `-o json` can obtain the Json format. Inside the Kubernetes, all data are storaged as json format, however, for our efficient reading, yaml is also supported. If you are good at `jq`, just go with `-o json`.

```
$ kubectl -n kube-system get pod --field-selector status.phase=Running
NAME                                         READY   STATUS    RESTARTS   AGE
azure-ip-masq-agent-rswls                    1/1     Running   0          13m
cloud-node-manager-6fk75                     1/1     Running   0          13m
coredns-69c47794-9f5g4                       1/1     Running   0          13m
coredns-69c47794-lhhzw                       1/1     Running   0          13m
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          13m
csi-azuredisk-node-jdjnf                     3/3     Running   0          13m
csi-azurefile-node-f2tjr                     3/3     Running   0          13m
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          28m
kube-proxy-cvwsp                             1/1     Running   0          13m
```
Filter pods with phase `Running`, please note, the `--field-selector` should only be used for information contained in `-o yaml` or `-o json`, not `kubectl describe`


`labels` plays an important role in Kubenetes to distinguish pod type, can be used for many purposes which we will cover in the future. It is like:
```
metadata:
  labels:
    component: kube-proxy
    controller-revision-hash: fbc56b478    
    pod-template-generation: "2"
    tier: node
```

`kubectl` natively provided some operation to dealing with the `lavels` meatdata.
```
$ kubectl -n kube-system get pod --show-labels
NAME                                         READY   STATUS    RESTARTS   AGE   LABELS
azure-ip-masq-agent-rswls                    1/1     Running   0          22m   controller-revision-hash=bd8f68bc9,k8s-app=azure-ip-masq-agent,pod-template-generation=1,tier=node
cloud-node-manager-6fk75                     1/1     Running   0          22m   controller-revision-hash=54c68d7fbd,k8s-app=cloud-node-manager,pod-template-generation=1
coredns-69c47794-9f5g4                       1/1     Running   0          22m   k8s-app=kube-dns,kubernetes.io/cluster-service=true,pod-template-hash=69c47794,version=v20
coredns-69c47794-lhhzw                       1/1     Running   0          22m   k8s-app=kube-dns,kubernetes.io/cluster-service=true,pod-template-hash=69c47794,version=v20
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          22m   k8s-app=coredns-autoscaler,pod-template-hash=7d56cd888
csi-azuredisk-node-jdjnf                     3/3     Running   0          22m   app=csi-azuredisk-node,controller-revision-hash=559b75cd7b,pod-template-generation=1
csi-azurefile-node-f2tjr                     3/3     Running   0          22m   app=csi-azurefile-node,controller-revision-hash=7b4d9d667d,pod-template-generation=1
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          38m   k8s-app=dashboard-metrics-scraper,pod-template-hash=5545f488c6
kube-proxy-cvwsp                             1/1     Running   0          22m   component=kube-proxy,controller-revision-hash=fbc56b478,pod-template-generation=2,tier=node
```
`--show-labels` flag will let the `kubectl get` show all the labels in a standalone column

If you only want to check some of the labels, you can use
```
kubectl -n kube-system get pod -L
kubectl -n kube-system get pod --label-columns=
```
for example
```
$ kubectl -n kube-system get pod --label-columns=k8s-app,version
$ kubectl -n kube-system get pod -L=k8s-app,version
$ kubectl -n kube-system get pod -L='k8s-app','version'
$ kubectl -n kube-system get pod -L k8s-app,version
# will all show
NAME                                         READY   STATUS    RESTARTS   AGE   K8S-APP                     VERSION
azure-ip-masq-agent-rswls                    1/1     Running   0          28m   azure-ip-masq-agent
cloud-node-manager-6fk75                     1/1     Running   0          28m   cloud-node-manager
coredns-69c47794-9f5g4                       1/1     Running   0          28m   kube-dns                    v20
coredns-69c47794-lhhzw                       1/1     Running   0          28m   kube-dns                    v20
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          28m   coredns-autoscaler
csi-azuredisk-node-jdjnf                     3/3     Running   0          28m
csi-azurefile-node-f2tjr                     3/3     Running   0          28m
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          43m   dashboard-metrics-scraper
kube-proxy-cvwsp                             1/1     Running   0          28m
```

Also, use `-l` or `--selector` flag, you can select labels with specific value

```
$ kubectl -n kube-system get pod --selector k8s-app==kube-dns
$ kubectl -n kube-system get pod --l k8s-app==kube-dns
$ kubectl -n kube-system get pod --l k8s-app=kube-dns
NAME                     READY   STATUS    RESTARTS   AGE
coredns-69c47794-9f5g4   1/1     Running   0          47m
coredns-69c47794-lhhzw   1/1     Running   0          47m
```

Based on my test, use multiple times of this flag will make the last one really apply:
```
$ kubectl -n kube-system get pod --selector k8s-app!=kube-dns
NAME                                         READY   STATUS    RESTARTS   AGE
azure-ip-masq-agent-rswls                    1/1     Running   0          49m
cloud-node-manager-6fk75                     1/1     Running   0          49m
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          49m
csi-azuredisk-node-jdjnf                     3/3     Running   0          49m
csi-azurefile-node-f2tjr                     3/3     Running   0          49m
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          64m
kube-proxy-cvwsp                             1/1     Running   0          49m

$ kubectl -n kube-system get pod --selector k8s-app!=kube-dns -l version==v20
NAME                     READY   STATUS    RESTARTS   AGE
coredns-69c47794-9f5g4   1/1     Running   0          49m
coredns-69c47794-lhhzw   1/1     Running   0          49m
```
You can see, `--selector k8s-app!=kube-dns` is being ignored.

## Dealing with Standard Output

If you don't want the table header, you can use `--no-headers` to hide it, this could be very useful when you want to export the stdout and deal with it.
```
$ kubectl -n kube-system get pod -L k8s-app,version --no-headers
azure-ip-masq-agent-rswls                    1/1   Running   0     30m   azure-ip-masq-agent
cloud-node-manager-6fk75                     1/1   Running   0     30m   cloud-node-manager
coredns-69c47794-9f5g4                       1/1   Running   0     30m   kube-dns                    v20
coredns-69c47794-lhhzw                       1/1   Running   0     30m   kube-dns                    v20
coredns-autoscaler-7d56cd888-xn7fg           1/1   Running   0     30m   coredns-autoscaler
csi-azuredisk-node-jdjnf                     3/3   Running   0     30m
csi-azurefile-node-f2tjr                     3/3   Running   0     30m
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1   Running   0     46m   dashboard-metrics-scraper
kube-proxy-cvwsp                             1/1   Running   0     30m
```

Using `--sort-by` can sort the pods in certain order
```
kubectl -n kube-system get pod --sort-by .metadata.creationTimestamp
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          69m
coredns-69c47794-9f5g4                       1/1     Running   0          54m
coredns-69c47794-lhhzw                       1/1     Running   0          54m
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          54m
tunnelfront-94444f59f-m7brn                  1/1     Running   0          54m
metrics-server-6576d9ccf8-2q8hl              1/1     Running   0          54m
...
```


## Constant Monitoring

`--watch` flag can help you monitoring the resource change during the watching period(Before you hit `Ctrl+C`).
say, a cluster originally looked like this
```
$ kubectl -n kube-system get pod --watch
NAME                                         READY   STATUS    RESTARTS   AGE
azure-ip-masq-agent-rswls                    1/1     Running   0          35m
cloud-node-manager-6fk75                     1/1     Running   0          35m
coredns-69c47794-9f5g4                       1/1     Running   0          36m
coredns-69c47794-lhhzw                       1/1     Running   0          36m
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          36m
csi-azuredisk-node-jdjnf                     3/3     Running   0          35m
csi-azurefile-node-f2tjr                     3/3     Running   0          35m
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          51m
kube-proxy-cvwsp                             1/1     Running   0          35m
kubernetes-dashboard-dc7976c89-b75p4         1/1     Running   0          36m
```
Then I `kubectl -n kube-system delete pod kubernetes-dashboard-dc7976c89-b75p4`, it will be like:
```
$ kubectl -n kube-system get pod --watch
NAME                                         READY   STATUS    RESTARTS   AGE
azure-ip-masq-agent-rswls                    1/1     Running   0          35m
cloud-node-manager-6fk75                     1/1     Running   0          35m
coredns-69c47794-9f5g4                       1/1     Running   0          36m
coredns-69c47794-lhhzw                       1/1     Running   0          36m
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          36m
csi-azuredisk-node-jdjnf                     3/3     Running   0          35m
csi-azurefile-node-f2tjr                     3/3     Running   0          35m
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          51m
kube-proxy-cvwsp                             1/1     Running   0          35m
kubernetes-dashboard-dc7976c89-b75p4         1/1     Running   0          36m
kubernetes-dashboard-dc7976c89-b75p4         1/1     Terminating   0          37m
kubernetes-dashboard-dc7976c89-w7fkz         0/1     Pending       0          0s
kubernetes-dashboard-dc7976c89-w7fkz         0/1     Pending       0          0s
kubernetes-dashboard-dc7976c89-b75p4         0/1     Terminating   0          37m
kubernetes-dashboard-dc7976c89-b75p4         0/1     Terminating   0          37m
kubernetes-dashboard-dc7976c89-b75p4         0/1     Terminating   0          37m
kubernetes-dashboard-dc7976c89-w7fkz         0/1     Pending       0          1s
kubernetes-dashboard-dc7976c89-w7fkz         0/1     ContainerCreating   0          1s     
kubernetes-dashboard-dc7976c89-w7fkz         1/1     Running             0          6s 
```

## Exercise
