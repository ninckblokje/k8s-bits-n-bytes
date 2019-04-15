# Opdracht 7: Opslag

Achtergrond informatie:
- https://kubernetes.io/docs/concepts/storage/storage-classes/
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

Delete de postgres pod:

````
$ kubectl delete -n ninckblokje po/postgres
````

In één keer kan een persistant volume en een persistante volume claim aangemaakt worden. De storage class zorgt voor de provisioning van de storage. Deploy `postgres-[K8S-IMPL]-pvc.yaml`:

````
$ kubectl apply -n ninckblokje -f postgres-microk8s-pvc.yaml
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS        REASON   AGE
pvc-2a07fcdf-5dc3-11e9-91d0-00155d021822   5Gi        RWO            Delete           Bound    ninckblokje/postgres-pvc   microk8s-hostpath            32m
$ kubectl get -n ninckblokje pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
postgres-pvc   Bound    pvc-2a07fcdf-5dc3-11e9-91d0-00155d021822   5Gi        RWO            microk8s-hostpath   33m
````

Verander `postgres.yaml` zodat deze de persistant volume claim gebruikt. Voeg een `volumeMount` onder `containers` en voeg een `volume` onder `spec` toe:

````
...
    volumeMounts:
      - mountPath: "/var/lib/postgresql"
        name: postgres-volume
  volumes:
    - name: postgres-volume
      persistentVolumeClaim:
        claimName: postgres-pvc
...
````

Voeg tevens een init container toe. Deze maakt een subdirectory en zet de rechten goed

````yaml
...
      initContainers:
        - name: postgres-volume-init
          image: centos:7.6.1810
          command: ["/bin/bash"]
          args: ["-c", "mkdir /data/data; chown -R 777 /data"]
          volumeMounts:
          - name: postgres-volume
            mountPath: /data
...
````

Deploy Postgres opnieuw:

````
$ kubectl apply -n ninckblokje -f postgres.yaml
````

Een manier om de storage te bekijken is deze via een andere pod te mount. Voeg hetzelfde als boven toe aan `storage-viewer.yaml`, met als extra optie onder `persistentVolumeClain` `readOnly`.

````yaml
...
    volumeMounts:
      - mountPath: "/var/lib/postgresql/data"
        name: postgres-volume
  volumes:
    - name: postgres-volume
      persistentVolumeClaim:
        claimName: postgres-pvc
        readOnly: true
...
````

Storage kan automatisch worden aangemaakt bij het gebruik van een stateful set. Delete eerst de bestaande pod, pvc en pv (deze is dynamisch aangemaakt).

````
$ kubectl delete -n ninckblokje po/postgres
$ kubectl delete -n ninckblokje pvc/postgres-pvc
$ kubectl get -n ninckblokje pv
$ kubectl delete -n ninckblokje pv/pvc-2a07fcdf-5dc3-11e9-91d0-00155d021822
````

Het verwijderen van de pvc en pv gaat niet, omdat er nog een pod aan gekoppeld is. Delete eerst deze:

````
$ kubectl delete -n ninckblokje po/storage-viewer
$ kubectl delete -n ninckblokje pvc/postgres-pvc
$ kubectl get ninckblokje pv
$ kubectl delete pv/pvc-2a07fcdf-5dc3-11e9-91d0-00155d021822
````

Deploy nu postgres via een stateful set:

````yaml
$ kubectl -n ninckblokje -f kubectl apply -n ninckblokje -f postgres-[K8S-IMPL]-statefulset.yaml
````

Er worden nu 2 pods, 2 pvc's en 2 pv's aangemaakt.

Delete nu de statefulset

````
$ kubectl delete -n ninckblokje statefulset/postgres
$ kubectl get -n ninckblokje po
No resources found.
$ kubectl get -n ninckblokje pvc
NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
postgres-volume-postgres-0   Bound    pvc-1536fe93-5e87-11e9-a6d2-00155d021822   1Gi        RWO            microk8s-hostpath   6m33s
postgres-volume-postgres-1   Bound    pvc-1728b871-5e87-11e9-a6d2-00155d021822   1Gi        RWO            microk8s-hostpath   6m30s
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                    STORAGECLASS        REASON   AGE
pvc-1536fe93-5e87-11e9-a6d2-00155d021822   1Gi        RWO            Delete           Bound    ninckblokje/postgres-volume-postgres-0   microk8s-hostpath            6m42s
pvc-1728b871-5e87-11e9-a6d2-00155d021822   1Gi        RWO            Delete           Bound    ninckblokje/postgres-volume-postgres-1   microk8s-hostpath            6m39s
````

De pvc's en pv's blijven bestaan. Delete deze handmatig:

````
$ kubectl delete -n ninckblokje pvc --all
````