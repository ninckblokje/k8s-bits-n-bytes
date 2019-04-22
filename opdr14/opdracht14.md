# Opdracht 14: Build your own!

Maak de volgende opzet:
- MongoDB
- Mongo Express
- Rate training

## MongoDB

Container:
- Image: `mongo:4.0`
- Port: `27017`
- Link: https://hub.docker.com/_/mongo

Variabelen:
- MONGO_INITDB_ROOT_USERNAME=root
- MONGO_INITDB_ROOT_PASSWORD
  - Geheim wachtwoord

Zorg voor storage in de directory: `/data/db`

Kenmerken:
- Eén instance
- Ontsloten via service, alleen binnen K8S cluster
- Storage in directory: `/data/db`

## Mongo Express

Container:
- Image: `mongo-express`
- Port: `8081`
- Link: https://hub.docker.com/_/mongo-express

Variabelen:
- ME_CONFIG_MONGODB_SERVER
  - Wijst naar MongoDB service
- ME_CONFIG_MONGODB_ADMINUSERNAME=root
- ME_CONFIG_MONGODB_ADMINPASSWORD
  - Gelijk aan MONGO_INITDB_ROOT_PASSWORD

Kenmerken:
- Eén instance
- Ontsloten via service, alleen binnen K8S cluster

## Rate training

Container:
- Image: `ninckblokje/ratetraining:1.0.0`
- Port: `8080`

Variabelen:
- SPRING_DATA_MONGODB_URI=mongodb://root:[Wachtwoord MongoDB]@[MongoDB Service]:27017/training?authSource=admin
  - Vervang [Wachtwoord MongoDB] en [MongoDB Service] met de juiste waarden

Kenmerken
- Twee instances
- Ontsloten via een service, alleen binnen K8S cluster
- Ontsloten via Ingress op port 80 op hostname rate-training.[NAMESPACE].bnb
  - Vervang [NAMESPACE] met je eigen namespace

## Testen

````
$ curl -X POST -H "Host: rate-training.ninckblokje.bnb" -H "Content-Type: appication/json" -d '{ "rating": 10, "byWho": "ninckblokje" }' http://[INGRESS_EXTERNAL_URL]/rating/k8s%20bits%20n%20bytes
$ curl -H "Host: rate-training.ninckblokje.bnb" http://[INGRESS_EXTERNAL_URL]/training
````