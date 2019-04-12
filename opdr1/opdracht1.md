# Opdracht 1: Namespace aanmaken

Achtergrond informatie: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

Maak een eigen namespace aan:
- Pas het bestand `namespace.yaml` aan zodat `metadata.name` jouw naam bevat

````yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ninckblokje
````

- Apply de yaml met `kubectl apply -f namespace.yaml`
- Controleer of de namespace is aangemaakt met `kubectl get ns`

````
NAME          STATUS   AGE
default       Active   11h
kube-public   Active   11h
kube-system   Active   11h
ninckblokje   Active   5s
````