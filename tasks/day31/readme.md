# [Day 31](https://www.youtube.com/watch?v=VcWpZoRAQXE) - Understanding CoreDNS In Kubernetes

Day 31 of the #40DaysofKubernetes challenge was about the CoreDNS in Kubernetes. This is an internal DNS in Kubernetes.
This is deployed as a deployment in Kubernetes
```
kubectl get deploy -n=kube-system

NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           5d2h
```

The CoreDNS has also a service, so we can reach it internally
```
kubectl get svc -n=kube-system 

NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   7m32s
```

We can see the IP of the DNS is also in the pods
```
kubectl exec -it nginx-bf5d5cf98-fll48 -- cat /etc/resolv.conf

search default.svc.cluster.local svc.cluster.local cluster.local <internal network>
nameserver 10.96.0.10
options ndots:5
```

In the configmap of coredns
```
kubectl get cm -n=kube-system 

NAME                                                   DATA   AGE
coredns                                                1      33m
extension-apiserver-authentication                     6      33m
kube-apiserver-legacy-service-account-token-tracking   1      33m
kube-proxy                                             2      33m
kube-root-ca.crt                                       1      33m
kubeadm-config                                         1      33m
kubelet-config                                         1      33m
```

Here we can see, that the traffic regarding dns to the main /etc/resolv.conf be forwarded
```
kubectl describe cm coredns -n=kube-system

Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}


BinaryData
====

Events:  <none>
```


To learn more about it go to [DNS Pod Service](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) and [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/).