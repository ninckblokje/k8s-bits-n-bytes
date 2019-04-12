# Opdracht 4: Deployment

Achtergrond informatie: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

Verwijder de oude pods:

````
$ kubectl delete -n ninckblokje po --all
````

Start de deployment:

````
$ kubectl apply -n ninckblokje -f deployment.yaml
````

Hoeveel pods zijn er? Ziet de service de pods?

Schaal de pod op naar 3:

````
$ kubectl scale -n ninckblokje deploy/ping-pong --replicas=3
````

Hoeveel pods zijn er? Ziet de service de pods?

Maak een kopie van het bestand `deployment.yaml` genaamd: `deployment-upgrade.yaml`. Verander hier het volgende in:

- De nieuwe image wordt: `ninckblokje/ping-pong:node-express`
- Onder spec moet de upgrade strategy worden toegevoegd:

````yaml
...
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 50%
      maxSurge: 1
...
````

Voer nu de upgrade uit (deze gaat snel). Er zijn nu tijdelijk 2 replicatie sets

Kill nu 1 pod, wat gebeurt er?

````
$ kubectl get -n ninckblokje po
$ kubectl delete -n ninckblokje po/ping-pong-7dd8c58b58-dp565
$ kubectl get -n ninckblokje po
````