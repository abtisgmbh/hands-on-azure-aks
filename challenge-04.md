# Challenge 04 - Applikation über Services veröffentlichen

Damit die einzelnen Pods der Applikation per Load Balancing verfügbar gemacht werden können, benötigt es jeweils einen Service im Kubernetes Cluster. Über diese Services werden Anfragen an die richtigen Pods weitergeleitet.  
Kubernetes benutzt dafür intern eine Namensauflösung.

Aufgabe:
> - Erstelle Services für alle Deployments
>   - RandomNumberService
>   - LottoService
>   - LottoWeb
> - Teste den Zugriff auf den Lotto Service über ein Port-Forwarding
> - Mache das Web Deployment im Internet verfügbar
> - Mache den Dienst über einen Ingress verfügbar
>   - Ein Ingress hilft dabei ein OSI-Layer-7 aufzubauen
>   - Die Applikation wird über eine URL verfügbar gemacht (hierfür kann nip.io eine Domain bereitstellen)
>   - Benutze einen nginx Ingress

[[Hint: Applikation über Services veröffentlichen](hints/expose-application.md)]

