# Opdracht 13: Debugging pods

Deploy `sh.yaml` en tail de logging. Wat gaat er fout?

````
$ kubectl apply -n ninckblokje -f sh.yaml
$ kubectl logs -f -n ninckblokje sh
````

Deploy `os.yaml`. Wat gaat er fout?

````
$ kubectl apply -n ninckblokje -f os.yaml
$ kubectl logs -f -n ninckblokje os
$ kubectl get -n ninckblokje po/os -o yaml
$ kubectl describe -n ninckblokje po/os
````

Deploy `oom.yaml`. Wat gaat er fout?

````
$ kubectl apply -n ninckblokje -f oom.yaml
$ kubectl logs -f -n ninckblokje oom
$ kubectl get -n ninckblokje po/oom -o yaml
$ kubectl describe -n ninckblokje po/oom
````