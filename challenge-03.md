# Challenge 03 - Eigene Applikation in AKS bereitstellen

Für diese Challenge benötigt es weitere Resourcen in Azure. Sowie in deiner Umgebung.

Hier wird eine kleine .NET Applikation bereitgestellt. Sie liegt auf Github bereit.

Aufgabe:
> - Erstelle eine Container Registry
> - Clone Github Projekt: [daniellindemann/lotto-microservices](https://github.com/daniellindemann/lotto-microservices/)
> - Erstelle für alle Projektbestandteile Docker Images und pushe sie in die Container Registry
>   - Die benötigten Dockerfiles liegen in den Projektordnern unter `src`
>       - LottoService: `src/LottoService/Dockerfile`
>       - LottoService: `src/RandomNumberService/Dockerfile`
>       - LottoService: `src/Web/Dockerfile`
> - Erstelle ein Deployment in Kubernetes, dass jeweils 2 Instanzen des jeweiligen Images im Namespace 'lotto' laufen lässt

[[Hint: Applikation in AKS bereitstellen](hints/create-cluster-cli.md)]

