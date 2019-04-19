# Opdracht 11: RBAC

Achtergrond informatie:
- https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- https://kubernetes.io/docs/reference/access-authn-authz/authorization/

Delete de andere resources uit je namespace:

````
$ kubectl delete -n ninckblokje po,svc,cm,secret,ingress --all
````

Deploy de sleeper pod en start bash:

````
$ kubectl apply -n ninckblokje -f sleeper.yaml
$ kubectl exec -it -n ninckblokje sleeper /bin/bash
[root@sleeper /]#
````

K8S mount gegevens en een service account in de pod:

````
[root@sleeper /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          50G  4.0G   44G   9% /
tmpfs            64M     0   64M   0% /dev
tmpfs           998M     0  998M   0% /sys/fs/cgroup
/dev/vda1        50G  4.0G   44G   9% /etc/hosts
shm              64M     0   64M   0% /dev/shm
tmpfs           998M   12K  998M   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs           998M     0  998M   0% /proc/acpi
tmpfs           998M     0  998M   0% /sys/firmware
[root@sleeper /]# cd /run/secrets/kubernetes.io/serviceaccount
[root@sleeper serviceaccount]# ls
ca.crt  namespace  token
[root@sleeper serviceaccount]# cat namespace
ninckblokje
[root@sleeper serviceaccount]# cat token
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJuaW5ja2Jsb2tqZSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLTc1bHpnIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiM2Y4MmQ1NC02MmQ5LTExZTktODIxNC05YTg1YmU2MjZhY2EiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6bmluY2tibG9ramU6ZGVmYXVsdCJ9.L-QjPsgmITILtgdgt3PNNVTpABLsjSN3xOHXFKQVL67CReUvIVRs-c7ZIE7aS5U6YKM2hDs1i-ojT-HvOefi5R4RCv6j9pmQ6ENwtTWsqpa7jnC4mxnqAxvbtwpAOhMAKQc4xGuf42aBg_yfIMsoY0oQSrFNwALy-7JVHC0LJIGvQMkQ2nxC8MUVMZ00tqMGdLu7c4X3f1_2Qfu2QrDoOexB99P6ZjU1wq7BpW8_wR0YkBOJODABi9ML3f0NomL1xlWUzgOEpwG--PU13c9YwApyqjSDFx_m8qBxaUsk_uhUWqwiqmEoA1rAsWG7pzaVb2_78fgdgapVw70hZAWvdA
[root@sleeper serviceaccount]#
````

In de ENV van de pod staan de gegevens waarmee de K8S API te bereiken is:

````
[root@sleeper serviceaccount]# env | grep KUBERNETES
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT=tcp://10.245.0.1:443
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.245.0.1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_ADDR=10.245.0.1
KUBERNETES_PORT_443_TCP=tcp://10.245.0.1:443
````

Installeer cURL en spreek de K8S API aan:

````
[root@sleeper serviceaccount]# yum install -y yum
...
[root@sleeper serviceaccount]# curl -k https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "namespaces is forbidden: User \"system:anonymous\" cannot list resource \"namespaces\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "namespaces"
  },
  "code": 403
}
````

Voeg nu de token van het service account toe:

````
[root@sleeper serviceaccount]# TOKEN=`cat token`
[root@sleeper serviceaccount]# curl -k -H "Authorization: Bearer $TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "namespaces is forbidden: User \"system:serviceaccount:ninckblokje:default\" cannot list resource \"namespaces\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "namespaces"
  },
  "code": 403
}
[root@sleeper serviceaccount]# curl -k -H "Authorization: Bearer $TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces/ninckblokje/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:ninckblokje:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"ninckblokje\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
````

Maak nu een `clusterrole.yaml` aan met de volgende inhoud:

````yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: [naam]-pod-ro-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
````

En een `serviceaccount.yaml`:

````yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: [naam]-pod-ro-sa
````

Apply beide YAML bestanden:

````
$ kubectl apply -f clusterrole.yaml
$ kubectl apply -n ninckblokje -f serviceaccount.yaml
$ kubectl get -n ninckblokje sa
NAME                    SECRETS   AGE
default                 1         36m
ninckblokje-pod-ro-sa   1         80
````

Pas nu `sleeper.yaml` aan zodat deze wijst naar het nieuwe service account (onder `spec`):

````yaml
...
  serviceAccountName: [naam]-pod-ro-sa
````

Redeploy de pod:

````
$ kubectl delete -n ninckblokje po/sleeper
$ kubectl apply -n ninckblokje -f sleeper.yaml
````

Zoek nu opnieuw het token op:

````
$ kubectl exec -it -n ninckblokje sleeper /bin/bash
[root@sleeper /]# cd /run/secrets/kubernetes.io/serviceaccount
[root@sleeper serviceaccount]# TOKEN=`cat token`
[root@sleeper serviceaccount]# curl -k -H "Authorization: Bearer $TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "namespaces is forbidden: User \"system:serviceaccount:ninckblokje:ninckblokje-pod-ro-sa\" cannot list resource \"namespaces\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "namespaces"
  },
  "code": 403
}
[root@sleeper serviceaccount]# curl -k -H "Authorization: Bearer $TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces/ninckblokje/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:ninckblokje:ninckblokje-pod-ro-sa\" cannot list resource \"pods\" in API group \"\" in the namespace \"ninckblokje\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
````

De binding tussen de cluster role en het service account ontbreekt nog. Stop de pod en de exec sessie niet. Maak `clusterrolebinding.yaml` aan en deploy deze:

````yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: [naam]-pod-ro-rbc
subjects:
- kind: ServiceAccount
  name: [naam]-pod-ro-sa
  namespace: [naam]
roleRef:
  kind: ClusterRole
  name: [naam]-pod-ro-role
  apiGroup: rbac.authorization.k8s.io
````

Doe nu wederom in de pod:

````
[root@sleeper serviceaccount]# curl -k -H "Authorization: Bearer $TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "namespaces is forbidden: User \"system:serviceaccount:ninckblokje:ninckblokje-pod-ro-sa\" cannot list resource \"namespaces\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "namespaces"
  },
  "code": 403
}
[root@sleeper serviceaccount]# curl -k -H "Authorization: Bearer $TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces/ninckblokje/pods
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/ninckblokje/pods",
    "resourceVersion": "5016"
  },
  "items": [
...
  ]
}
````

Pods uit alle namespaces zijn nu te bekijken.