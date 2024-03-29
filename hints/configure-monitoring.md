# Lösung Challenge 05 - Monitoring und Logging

## 1. *LottoWeb* Pods einsehen

- Alles was in STDOUT geschrieben wird, wird automatisch geloggt
- Logs einsehen

    Bash/PowerShell:
    ```bash
    kubectl logs deployment/lotto-web -n lotto
    ```

## 2. *LottoWeb* Umgebungsvariablen einsehen

- Manchmal ist es wichtig und praktisch, wenn man Befehle im Pod bzw. Container ausführen kann. Damit können Fehlkonfigurationen festgestellt werden
- Umgebungsvariablen einsehen

    Bash/PowerShell:
    ```bash
    kubectl exec < pad name > -n lotto -- env
    ```

## 3. Auslastung der Nodes einsehen

- Kubernetes stellt die Metriken der Nodes und Pods bereit
- Metriken der Nodes einsehen

    Bash/PowerShell:
    ```bash
    kubectl top nodes
    ```

- Metriken der Pods einsehen

    Bash/PowerShell:
    ```bash
    kubectl top pods
    ```

## 4. Resource Quotas für *LottoWeb* Pods erstellen

- Resource Quotas werden nicht automatisch zu Deployment hinzugefügt
- Aktuelles Deployment in Datei speichern

    Bash/PowerShell:
    ```bash
    kubectl get deployment lotto-web -n lotto -o yaml > deployment-lotto-web.yaml
    ```

- Editiere die Datei und füge Resource Requests und Resource Limits hinzu

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: lotto-web
      name: lotto-web
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: lotto-web
      template:
        metadata:
          labels:
            app: lotto-web
        spec:
          containers:
          - image: acrabtisworkshop32258.azurecr.io/daniellindemann/lotto-web:1.0.0
            name: lotto-web
            resources:
              requests:
                cpu: "100m"
                memory: "64Mi"
              limits:
                cpu: "250m"
                memory: "256Mi"
    ```

- Datei abspeichern und apply

    Bash/PowerShell:
    ```bash
    kubectl apply -f deployment-lotto-web.yaml -n lotto
    ```

## 5. Kubernetes Dashboard bereitstellen

- Das Kubernetes Dashboard ist eine Web UI mit der ein Cluster eingesehen werden kann
- Das Kubernetes Dashboard bietet einem Benutzer (mit entsprechenden Rechten) jegliche Konfiguration im Cluster vorzunehmen
- Das Kubernetes Dashboard sollte in Produktivumgebungen nie bereitgestellt sein
- Das Dashboard kann mit der Azure CLI bereitgestellt werden werden

    Bash/PowerShell:
    ```bash
    az aks browse -g $RESOURCE_GROUP -n $AKS_CLUSTER_NAME
    ```

## 6. Logging in Log Analytics Workspace

### 6.1 Log Analytics Workspace erstellen

- Log Analytics Work erstellen

    Bash:
    ```bash
    LOGANALYTICS_NAME=log-abtis-workshop-$AKS_RANDOM
    az monitor log-analytics workspace create -g $RESOURCE_GROUP -n $LOGANALYTICS_NAME -l northeurope
    ```

    PowerShell:
    ```powershell
    $LOGANALYTICS_NAME=log-abtis-workshop-$AKS_RANDOM
    az monitor log-analytics workspace create -g $RESOURCE_GROUP -n $LOGANALYTICS_NAME -l northeurope
    ```

- Log Analytics Workspace ID auslesen

    Bash:
    ```bash
    LOGANALYTICS_ID=$(az monitor log-analytics workspace show -g $RESOURCE_GROUP -n $LOGANALYTICS_NAME --query 'id' -o tsv)
    ```

    PowerShell:
    ```powershell
    $LOGANALYTICS_ID=(az monitor log-analytics workspace show -g $RESOURCE_GROUP -n $LOGANALYTICS_NAME --query 'id' -o tsv)
    ```

### 6.2 Log Analytics Workspace mit AKS verheiraten

- Log Analytics Workspace anbinden

    Bash/PowerShell:
    ```bash
    az aks enable-addons -a monitoring -n $AKS_CLUSTER_NAME -g $RESOURCE_GROUP --workspace-resource-id $LOGANALYTICS_ID
    ```

- Azure stellt benötigte Resourcen in Kubernetes Cluster bereit

### 6.3 Testen

- Ein bisschen warten
- Kubernetes Metriken und Logs ansehen
