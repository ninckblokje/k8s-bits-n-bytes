# Opdracht 14: Security

Deploy `run-as.yaml`, start en shell en type het commando `id`:

````
$ kubectl exec -it -n ninckblokje run-as /bin/sh
/ # id
uid=0(root) gid=0(root) groups=10(wheel
/ # cd /data
/data # ls -l
total 4
drwxrwsrwx    2 root     root          4096 Apr 22 14:45 demo
/data # ping www.syntouch.nl
PING www.syntouch.nl (185.104.29.56): 56 data bytes
````

Breidt `run-as.yaml` nu uit met een `securityContext`:

````yaml
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
````

````
$ kubectl exec -it -n ninckblokje run-as /bin/sh
/ # id
uid=0(root) gid=0(root) groups=10(wheel
/ # cd /data
/data # ls -l
total 4
drwxrwsrwx    2 root     2000          4096 Apr 22 14:45 demo
/data # ping www.syntouch.nl
PING www.syntouch.nl (185.104.29.56): 56 data bytes
ping: permission denied (are you root?)
````

Doe nu in de pod een wget van https://www.syntouch.nl:

````
/ # wget https://www.syntouch.nl
Connecting to www.syntouch.nl (185.104.29.56:443)
wget: note: TLS certificate validation not implemented
index.html           100% |**************************************************************************************************************************************************| 92815  0:00:00 ETA
/ # wget http://mongodb:27017/actuator/health
Connecting to mongodb:27017 (10.245.43.21:27017)
health               100% |**************************************************************************************************************************************************|    85  0:00:00 ETA
````

Maak een policy aan om verkeer naar buiten het cluster te blokkeren in het bestand `deny-external-egress.yaml`:

````yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-external-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector: {}
````

Doe nog een wget naar https://www.syntouch.nl:

````
/ # wget https://www.syntouch.nl
Connecting to www.syntouch.nl (185.104.29.56:443)
wget: can't connect to remote host (185.104.29.56): Connection timed out
````

Andere pods zijn wel te bereiken:

````
/ # wget http://mongodb:27017/actuator/health
Connecting to mongodb:27017 (10.245.43.21:27017)
health               100% |**************************************************************************************************************************************************|    85  0:00:00 ETA
````

Het is eenvoudig om de Docker daemon beschikbaar te stellen in een pod, via de Docker socket. Pas het bestand `docker.yaml` aan zodat een volume (wijst naar de Docker socket) beschikbaar komt:

````yaml
...
    volumeMounts:
      - name: dockersock
        mountPath: "/var/run/docker.sock"
  volumes:
    - name: dockersock
      hostPath:
        path: /var/run/docker.sock
...
````

Deploy de pod, start een shell en doe een `docker ps`:

````
$ kubectl exec -it -n ninckblokje docker /bin/sh
/ # docker ps
...
````

Nu zijn alle containers (inclusief de containers die draaien op de host) zichtbaar en beheerbaar vanuit deze pod.