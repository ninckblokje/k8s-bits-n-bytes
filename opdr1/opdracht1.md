# Opdracht 1: Namespace aanmaken

Maak een eigen namespace aan:
1. Pas het bestand `namespace.yaml` aan zodat `metadata.name` jouw naam bevat
2. Apply de yaml met `kubectl apply -f namespace.yaml`
3. Controleer of de namespace is aangemaakt met `kubectl get ns`

````
NAME          STATUS   AGE
default       Active   11h
kube-public   Active   11h
kube-system   Active   11h
ninckblokje   Active   5s
````