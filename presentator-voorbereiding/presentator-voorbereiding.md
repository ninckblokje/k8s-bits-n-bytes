# Presentator voorbereiding

## Dashboard

````
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
````

## admin-user

````
$ kubectl apply -f admin-user.yaml
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created

$ kubectl -n kube-system get secret
NAME                               TYPE                                  DATA   AGE
admin-user-token-hrbpb             kubernetes.io/service-account-token   3      4s
cilium-operator-token-sdln4        kubernetes.io/service-account-token   3      10h
cilium-token-dnstt                 kubernetes.io/service-account-token   3      10h
coredns-token-pgjt6                kubernetes.io/service-account-token   3      10h
csi-do-controller-sa-token-6qsjj   kubernetes.io/service-account-token   3      10h
csi-do-node-sa-token-ft8kh         kubernetes.io/service-account-token   3      10h
default-token-n6w4v                kubernetes.io/service-account-token   3      10h
do-agent-token-5jmgl               kubernetes.io/service-account-token   3      10h
kube-proxy                         Opaque                                1      10h
kube-proxy-token-twqth             kubernetes.io/service-account-token   3      10h
kubernetes-dashboard-certs         Opaque                                0      3m5s
kubernetes-dashboard-csrf          Opaque                                1      3m5s
kubernetes-dashboard-key-holder    Opaque                                2      2m58s
kubernetes-dashboard-token-qfwpm   kubernetes.io/service-account-token   3      3m5s

$ kubectl -n kube-system describe secret admin-user-token-hrbpb
Name:         admin-user-token-hrbpb
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 6ba7a073-5967-11e9-b86c-6a6b29a1bf04

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1156 bytes
namespace:  11 bytes
token:      [VERY_SECRET_TOKEN]
````
