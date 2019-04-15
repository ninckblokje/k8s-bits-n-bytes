# Opdracht 10: Resources

Achtergrond informatie: https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/

Deploy `heater.yaml` en volg de logging:

````
$ kubectl apply -n ninckblokje -f heater.yaml
$ kubectl logs -n ninckblokje -f heater
-- 1
free memory: 7781584
total memory: 8257536
max memory: 127729664
-- 2
free memory: 2793024
total memory: 8257536
max memory: 127729664
-- 3
free memory: 7667048
total memory: 18149376
max memory: 127729664
-- 4
free memory: 9674208
total memory: 25210880
max memory: 127729664
...
-- 25
free memory: 7236432
total memory: 127729664
max memory: 127729664
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        at java.nio.file.Files.read(Unknown Source)
        at java.nio.file.Files.readAllBytes(Unknown Source)
        at ninckblokje.tools.heater.Heater.main(Heater.java:27)
````

Wacht op de OutOfMemoryError.

Pas in `heater.yaml` de request aan naar `64Mi` en deploy opnieuw:

````
$ kubectl delete -n ninckblokje po/heater
$ kubectl apply -n ninckblokje -f heater.yaml
$ kubectl logs -n ninckblokje -f heater
-- 1
free memory: 7781584
total memory: 8257536
max memory: 127729664
-- 2
free memory: 2793024
total memory: 8257536
max memory: 127729664
-- 3
free memory: 7667048
total memory: 18149376
max memory: 127729664
-- 4
free memory: 9674208
total memory: 25210880
max memory: 127729664
...
-- 25
free memory: 7236432
total memory: 127729664
max memory: 127729664
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        at java.nio.file.Files.read(Unknown Source)
        at java.nio.file.Files.readAllBytes(Unknown Source)
        at ninckblokje.tools.heater.Heater.main(Heater.java:27)
````

Pas in `heater.yaml` de limit aan naar `64Mi` en deploy opnieuw:

````
$ kubectl delete -n ninckblokje po/heater
$ kubectl apply -n ninckblokje -f heater.yaml
$ kubectl logs -n ninckblokje -f heater
-- 1
free memory: 7781584
total memory: 8257536
max memory: 32440320
-- 2
free memory: 2793024
total memory: 8257536
max memory: 32440320
-- 3
free memory: 7667048
total memory: 18149376
max memory: 32440320
-- 4
free memory: 9674208
total memory: 25210880
max memory: 32440320
-- 5
free memory: 4687216
total memory: 25210880
max memory: 32440320
-- 6
free memory: 6893328
total memory: 32440320
max memory: 32440320
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        at java.nio.file.Files.read(Unknown Source)
        at java.nio.file.Files.readAllBytes(Unknown Source)
        at ninckblokje.tools.heater.Heater.main(Heater.java:27)
````