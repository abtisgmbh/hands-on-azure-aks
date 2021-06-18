# Lösung Challenge 04 - Applikation über Services veröffentlichen

*Alle Skripte werden in der Bash ausgeführt. Für Windows PowerShell müssen Variablen anders gesetzt werden*

## 1. Services erstellen

- Service für *RandomNumberService* Deployment
    ```
    kubectl expose deployment random-number-service --port=80 --target-port=80 --name=random-number-service -n lotto
    ```
- Service für *LottoService* Deployment
    ```
    kubectl expose deployment lotto-service --port=80 --target-port=80 --name=lotto-service -n lotto
    ```
- Service für *Web* Deployment
    ```
    kubectl expose deployment lotto-web --port=80 --target-port=80 --name=lotto-web -n lotto
    ```
- Check the services
    ```
    kubectl get services -n lotto
    ```

## 2. Port-Forwarding

- Test den Dienst mittels Port-Forwarding
    ```
    kubectl port-forward service/lotto-web 8123:80 -n lotto
    ```
- Greife auf die zurückgegebene URL zu

## 3. Public IP für *LottoWeb* Service

- Ändere den Service `Type` von `ClusterIP` zu `LoadBalancer`
    ```
    kubectl edit service lotto-web -n lotto
    ```
- Warte bis der Dienst eine Public IP erhalten hat
    ```
    kubectl get svc -n lotto --watch
    ```
    oder
    ```
    watch kubectl get svc -n lotto
    ```
- Öffne die IP mit deinem Browser

## 4. nginx Ingress

- Konzept: https://kubernetes.io/docs/concepts/services-networking/ingress/
- Info: Bevor der Ingress installiere, kann der *LottoWeb* Service wieder auf `ClusterIP` umgestellt werden

### 4.1 Installiere nginx mit `helm`
- nginx-ingress repo herunterladen
    ```
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    ```
- Namespace erstellen
    ```
    INGRESS_NAMESPACE=ingress-nginx
    kubectl create namespace $INGRESS_NAMESPACE
    ```
- Ingress Controller installieren
    ```
    helm install nginx-ingress ingress-nginx/ingress-nginx \
        --namespace $INGRESS_NAMESPACE \
        --set controller.replicaCount=2 \
        --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
        --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
    ```
- Check installation
    ```
    kubectl get pods,services --namespace ingress-nginx
    ```

### 4.2 Ingress konfigurieren

- Die Applikation würd über den Dienst nip.io verfügbar gemacht
- Die URL lautet frontend.< ip adresse >.nip.io

- Erstelle eine yaml Datei mit dem Namen `ingress-lottoweb.yaml`, konfiguriere den Ingress entsprechend und speichere die Datei ab
    ```
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-lotto-web
      labels:
        app: lotto-web
    spec:
      rules:
      - host: "frontend.20.79.89.195.nip.io"
        http:
          paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: lotto-web
                port:
                  number: 80
    ```
- Stelle die yaml Konfiguration in Kubernetes bereit
    ```
    kubectl apply -f ingress-lottoweb.yaml -n lotto
    ```
- Greife auf die URL `frontend.<ip adresse>.nip.io` zu
