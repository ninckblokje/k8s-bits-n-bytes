# Opdracht 5: Configuratie

Achtergrond informatie:
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
- https://kubernetes.io/docs/concepts/configuration/secret/

Delete alle deployments en services:

````
$ kubectl delete -n ninckblokje deploy,svc --all
````

1. Deploy Postgres

Voeg een omgevings variabele met het wachtwoord toe aan het bestand `postgres.yaml`:

````
    env:
      - name: POSTGRES_PASSWORD
        value: Dummy_123
````

Deploy Postgres (zowel als een pod worden gedeployed):

````
$ kubectl apply -n ninckblokje postgres.yaml
pod/postgres created
service/postgres created
````

2. Deploy nu who-was-here

````
$ kubectl apply -n ninckblokje -f who-was-here.yaml
pod/who-was-here created
````

Deze maakt nu verbinding met een H2 database, maar de applicatie is slecht geprogrammeerd: Het kost 90 seconden voordat deze op.

````
$ kubectl port-forward -n ninckblokje po/who-was-here 8080:8080
$ curl http://localhost:8080/here
[]
$ curl -X POST http://localhost:8080/here/jnb
$ curl http://localhost:8080/here
[{"dateTime":"2019-04-09T18:15:25.241","name":"jnb"}]
````

Herstart de container en de data is weg

````
$ kubectl exec -n ninckblokje who-was-here reboot
$ kubectl port-forward -n ninckblokje po/who-was-here 8080:8080
$ curl http://localhost:8080/here
[]
````

3. Aanmaken config map

Maak een config map aan in het bestand `configmap.yaml`:

````yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: who-was-here
data:
  SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/postgres
  SPRING_DATASOURCE_USERNAME: postgres
  SPRING_DATASOURCE_PASSWORD: Dummy_123
````

Deploy de config map

````
$ kubectl apply -n ninckblokje -f configmap.yaml
configmap/who-was-here created
````

Kopieer het bestand `who-was-here.yaml` naar `who-was-here-cm.yaml` en haal de omgevings variabelen direct uit de config map. Plaats onder `container`:

````yaml
    envFrom:
      - configMapRef:
          name: who-was-here
````

Deploy de aangepaste pod:

````
$ kubectl delete -n ninckblokje -f po/who-was-here
$ kubectl apply -n ninckblokje -f who-was-here-cm.yaml
````

````
$ kubectl port-forward -n ninckblokje po/who-was-here 8080:8080
$ curl http://localhost:8080/here
[]
$ curl -X POST http://localhost:8080/here/jnb
$ curl http://localhost:8080/here
[{"dateTime":"2019-04-09T18:47:17.331","name":"jnb"}]
$ kubectl exec -n ninckblokje who-was-here reboot
$ kubectl port-forward -n ninckblokje po/who-was-here 8080:8080
$ curl http://localhost:8080/here
[{"dateTime":"2019-04-09T18:47:17.331","name":"jnb"}]
````

4. Secrets

Secrets horen los van de reguliere configuratie te staan. Maak een secret aan met dezelfde waarden als de config map en noem deze `secret.yaml`.

````yaml
apiVersion: v1
kind: Secret
metadata:
  name: who-was-here
type: Opaque
stringData:
  SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/postgres
  SPRING_DATASOURCE_USERNAME: postgres
  SPRING_DATASOURCE_PASSWORD: Dummy_123
````

Kopieer het bestand `who-was-here.yaml` naar `who-was-here-secrets.yaml` en haal de omgevings variabelen direct uit de config map. Plaats onder `container`:

````yaml
    envFrom:
      - secretRef:
          name: who-was-here
````

Deploy de aangepaste pod en secret:

````
$ kubectl delete -n ninckblokje po/who-was-here
$ kubectl delete -n ninckblokje cm/who-was-here
$ kubectl apply -n ninckblokje -f secret.yaml
$ kubectl apply -n ninckblokje -f who-was-here-secrets.yaml
$ kubectl port-forward -n ninckblokje po/who-was-here 8080:8080
$ curl http://localhost:8080/here
[{"dateTime":"2019-04-09T18:47:17.331","name":"jnb"}]
````

5. Opruimen

````
$ kubectl delete -n ninckblokje po/who-was-here
````