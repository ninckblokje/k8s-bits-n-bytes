# Opdracht 4: Configuratie

1. Deploy Postgres

Dit maakt zowel een Pod als een Service aan.

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

Deze maakt nu verbinding met een H2 database.

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

Deploy de aangepaste pod:

````
$ kubectl delete -n ninckblokje po/who-was-here
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

Secrets horen los van de reguliere configuratie te staan. Maak een secret aan met dezelfde waarden als de config map

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

Deploy de aangepaste pod en config map:

````
$ kubectl delete -n ninckblokje po/who-was-here
$ kubectl delete -n ninckblokje cm/who-was-here
$ kubectl apply -n ninckblokje -f secret.yaml
$ kubectl apply -n ninckblokje -f who-was-here-secrets.yaml
$ kubectl port-forward -n ninckblokje po/who-was-here 8080:8080
$ curl http://localhost:8080/here
[{"dateTime":"2019-04-09T18:47:17.331","name":"jnb"}]
````