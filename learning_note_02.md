# Logs of Kubenetes

Logs are critical for troubleshooting systems. Generally speaking, besides of logs in the kubenetes, there are also some logs in specific locations.

Normally, logs can be categolarized into the following four: critical/fatal, error, warning, info/debug. 

In kubernetes, we normally use `kubectl logs` to check the logs in a pod, this can only be applied to pod

## Get logs via kubectl

```
kubectl -n <ns> get logs <pod name>
```
Directly get the log

```
kubectl -n <ns> get logs <pod name> -c <container name>
```
Get the log from specific container. If there are many containers in the same pod, `-c` flag is mandetory.

```
$ kubectl -n kube-system get pod --show-labels
NAME                                         READY   STATUS    RESTARTS   AGE   LABELS
azure-ip-masq-agent-rswls                    1/1     Running   0          45h   controller-revision-hash=bd8f68bc9,k8s-app=azure-ip-masq-agent,pod-template-generation=1,tier=node
cloud-node-manager-6fk75                     1/1     Running   0          45h   controller-revision-hash=54c68d7fbd,k8s-app=cloud-node-manager,pod-template-generation=1
coredns-69c47794-9f5g4                       1/1     Running   0          45h   k8s-app=kube-dns,kubernetes.io/cluster-service=true,pod-template-hash=69c47794,version=v20
coredns-69c47794-lhhzw                       1/1     Running   0          45h   k8s-app=kube-dns,kubernetes.io/cluster-service=true,pod-template-hash=69c47794,version=v20
coredns-autoscaler-7d56cd888-xn7fg           1/1     Running   0          45h   k8s-app=coredns-autoscaler,pod-template-hash=7d56cd888
csi-azuredisk-node-jdjnf                     3/3     Running   0          45h   app=csi-azuredisk-node,controller-revision-hash=559b75cd7b,pod-template-generation=1
csi-azurefile-node-f2tjr                     3/3     Running   0          45h   app=csi-azurefile-node,controller-revision-hash=7b4d9d667d,pod-template-generation=1
dashboard-metrics-scraper-5545f488c6-q7gkd   1/1     Running   0          45h   k8s-app=dashboard-metrics-scraper,pod-template-hash=5545f488c6
kube-proxy-cvwsp                             1/1     Running   0          45h   component=kube-proxy,controller-revision-hash=fbc56b478,pod-template-generation=2,tier=node

$ kubectl -n kube-system get pod -l k8s-app=kube-dns
NAME                     READY   STATUS    RESTARTS   AGE
coredns-69c47794-9f5g4   1/1     Running   0          45h
coredns-69c47794-lhhzw   1/1     Running   0          45h

$ kubectl -n kube-system logs -l k8s-app=kube-dns
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
```
Just like the `kubectl get`, `-l` flag can also select the pods with same label.

`--since` flag can help you to determine the time duration of logs that you would like to check
```
$ kubectl -n kube-system logs coredns-69c47794-9f5g4 --since 20s
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server

$ kubectl -n kube-system logs coredns-69c47794-9f5g4 --since 1m
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server
[WARNING] No files matching import glob pattern: custom/*.override
[WARNING] No files matching import glob pattern: custom/*.server

$ kubectl -n kube-system logs coredns-69c47794-9f5g4 --since 1h
... ...
```

`--since-time` followed with UTC standard timestamp, you can select logs since a specific time
```
$ kubectl -n kube-system logs kubernetes-dashboard-dc7976c89-w7fkz --since-time 2022-05-02T16:04:02Z
2022/05/02 16:04:02 Starting overwatch
2022/05/02 16:04:02 Using namespace: kube-system
2022/05/02 16:04:02 Using in-cluster config to connect to apiserver
2022/05/02 16:04:02 Using secret token for csrf signing
2022/05/02 16:04:02 Initializing csrf token from kubernetes-dashboard-csrf secret
2022/05/02 16:04:02 Successful initial request to the apiserver, version: v1.22.6
2022/05/02 16:04:02 Generating JWE encryption key
2022/05/02 16:04:02 New synchronizer has been registered: kubernetes-dashboard-key-holder-kube-system. Starting
2022/05/02 16:04:02 Starting secret synchronizer for kubernetes-dashboard-key-holder in namespace kube-system
2022/05/02 16:04:02 Initializing JWE encryption key from synchronized object
2022/05/02 16:04:02 Creating in-cluster Sidecar client
2022/05/02 16:04:02 Serving insecurely on HTTP port: 9090
2022/05/02 16:04:02 Successful request to sidecar
```

## Locate the logs

If a pods got truncated, you will be unable to access logs via `kubectl logs` command. However, this does not means the logs of a pod cannot be retrived any more.

You need know the node of a pod that has been allocated to.

Then, ssh to that machine.

Go to `/var/log/containers/` directory, then you will see the logs whose pods might already been deleted.

if you execute `ls- ali`, you will find all the logs are softlinks in the `/var/log/containers/` directory, then you will find `/var/log/pods/`

As kubelet in a node is managed by the systemd, demon process, we can find the kubenetes related logs in `/var/log/syslog` or use `journalctl -u kubelet`.


## Event in Kubernetes
Event is a kind of kubernetes resource, we can treat it as a kind of log. It can help us to provide a brief overview of changes in Kubernetes resource level, say, changes, creating and deletion of pods.
```
$ kubectl get events
LAST SEEN   TYPE     REASON           OBJECT                                    MESSAGE
36m         Normal   RegisteredNode   node/aks-controller-31613468-vmss00000j   Node aks-controller-31613468-vmss00000j event: Registered Node aks-controller-31613468-vmss00000j in Controller
```
Normally, Kubernetes will delete evnets earlier than 1 hour to free up the space.

## Exercise
```
kubectl -n kube-system logs tunnelfront-94444f59f-m7brn --since 10s | grep int -i > inf.log
```
Please note `grep xxx -i` will transfer `xxx` to capital letter and then grep.