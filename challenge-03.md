# Challenge 03 - Eigene Applikation in AKS bereitstellen

Für diese Challenge benötigt es weitere Resourcen in Azure. Eine Azure Container Registry wird genutzt um Container Images zu hosten.

Hier wird eine kleine .NET Applikation bereitgestellt. Sie liegt als Code auf Github bereit, als auch als Container Images im Docker Hub.

Projekt auf Github: [daniellindemann/lotto-microservices](https://github.com/daniellindemann/lotto-microservices/)

Images in Docker Hub:

- Random Number Service: [lotto-randomnumberservice](https://hub.docker.com/r/daniellindemann/lotto-randomnumberservice)
- Lotto Service: [lotto-lottoservice](https://hub.docker.com/r/daniellindemann/lotto-lottoservice)
- Web Frontend: [lotto-web](https://hub.docker.com/r/daniellindemann/lotto-web)

Aufgabe:

- Erstelle eine Container Registry
- Erstelle für alle Dienste der Lotto Solution Deployments mit folgender Konfiguration:
  - Von jedem Container soll es 3 Instanzen geben
  - Die Container laufen im Namespace `lotto`

[[Hint: Applikation in AKS bereitstellen](hints/solution-in-aks.md)]
