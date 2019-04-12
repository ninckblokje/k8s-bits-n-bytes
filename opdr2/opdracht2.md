# Opdracht 2: Pod deployen

Achtergrond informatie: https://kubernetes.io/docs/concepts/workloads/pods/pod/

1. Deploy de pod

Deploy de pod in je eigen namespace:

````
kubectl apply -n ninckblokje -f pod.yaml
````

Controleer of deze goed gestart is:

````
$ kubectl get -n ninckblokje po
NAME        READY   STATUS    RESTARTS   AGE
ping-pong   1/1     Running   0          106s
````

Details pod opvragen:

````
$ kubectl get -n ninckblokje po/ping-pong -o yaml
````

2. Netwerk toegang tot pod:

````
$ kubectl port-forward -n ninckblokje po/ping-pong 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
````

Open de pagina: http://localhost:8080/ping

Dit is ongeveer de output: `2019-04-07T19:25:11Z Pong from ping-pong`

3. Logs:

Bekijken van logs:

````
$ kubectl logs -n ninckblokje ping-pong
/ping: map[Accept:[text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8] Accept-Language:[nl,en-US;q=0.7,en;q=0.3] Dnt:[1] Connection:[keep-alive] Upgrade-Insecure-Requests:[1] User-Agent:[Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0] Accept-Encoding:[gzip, deflate] Cookie:[m=]]
````

Tailen van logs:

````
$ kubectl logs -n ninckblokje -f ping-pong
/ping: map[Accept:[text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8] Accept-Language:[nl,en-US;q=0.7,en;q=0.3] Dnt:[1] Connection:[keep-alive] Upgrade-Insecure-Requests:[1] User-Agent:[Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0] Accept-Encoding:[gzip, deflate] Cookie:[m=]]
````

4. Toegang tot pod:

Deploy de sleeper pod:

````
$ kubectl apply -n ninckblokje -f sleeper.yaml
````

Start een shell in de pod en installeer wget.

````
$ kubectl exec -it -n ninckblokje sleeper /bin/bash
[root@sleeper /]# yum install -y wget
````