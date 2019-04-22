# Opdracht 13: K8S API mogelijkheden

Achtergrond informatie:
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14/
- https://github.com/codecentric/spring-boot-admin
- https://spring.io/projects/spring-cloud-kubernetes

Zorg ervoor dat de K8S API & dashboard beschikbaar zijn met: `kubectl proxy`

Open in de browser de volgende links:
- http://localhost:8001/api/v1/pods
- http://localhost:8001/api/v1/namespaces/ninckblokje/pods
- http://localhost:8001/api/v1/services
- http://localhost:8001/api/v1/namespaces/ninckblokje/services
- http://localhost:8001/api/v1/endpoints
- http://localhost:8001/api/v1/namespaces/ninckblokje/endpoints
- http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login

Maak een nieuwe cluster role binding en service account aan die de cluster role `spring-cloud` gebruiken.

Deploy `spring-boot-admin.yaml` en forward port 8080 van deze port.

Spring Boot Admin ziet nu wat er draait en op Spring Boot applicaties zijn meer metrics en mogelijkheden beschikbaar (afhankelijk van wat er draait). Spring Boot Admin reageert automatisch op wat er gebeurt in het cluster.

![](../assets/spring-boot-admin.png)