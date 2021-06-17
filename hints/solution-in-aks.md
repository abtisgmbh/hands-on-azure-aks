# Lösung Challenge 03 - Eigene Applikation in AKS bereitstellen

*Alle Skripte werden in der Bash ausgeführt. Für Windows PowerShell müssen Variablen anders gesetzt werden*

[Hint: Die verwendeten Parameter wurden in [*Challenge 02 - Kubernetes Cluster über Azure CLI erstellen*](../challenge-02.md) angelegt]

## 1. Erstelle eine Container Registry

Für das bereitstellen eigener Applikationen benötigt es eine Container Registry. Die bekannteste ist der [Docker Hub](https://hub.docker.com). Hier liegen viele öffentliche Container Images mit denen Applikationen bereitgestellt werden können.  
Damit eigene private Container Images bereitgestellt werden können, gibt es die [Azure Container Registry](https://azure.microsoft.com/de-de/services/container-registry/)

```
ACR_NAME=acrabtistraining$RANDOM
az acr create \
    --resource-group $RESOURCE_GROUP \
    --location $REGION_NAME \
    --name $ACR_NAME \
    --sku Basic
```

## 2. Container Registry mit AKS verknüpfen

Damit Images aus der Container Registry durch Kubernetes geladen werden können, müssen die beiden miteinander verheiratet werden. Das geht am einfachsten über die Azure CLI.

```
az aks update \
    --name $AKS_CLUSTER_NAME \
    --resource-group $RESOURCE_GROUP \
    --attach-acr $ACR_NAME
```

## 3. Erstellen der Images

Über die Azure Container Registry können Images direkt gebaut werden, ohne dass Docker oder eine andere Container Runtime auf dem Client installiert werden muss.

- Wechsele in den `src` Ordner des `lotto-microservices` Repos
    ```
    cd lotto-microservices/src
    ```
- Erstelle das *RandomNumberService* Image über die Azure Container Registry
    ```
    IMAGE_RANDOMNUMBERSERVICE='dlindemann/randomnumberservice'
    az acr build \
        --resource-group $RESOURCE_GROUP \
        --registry $ACR_NAME \
        -t $IMAGE_RANDOMNUMBERSERVICE:v1 \
        -t $IMAGE_RANDOMNUMBERSERVICE:{{.Run.ID}} \
        ./RandomNumberService
    ```
- Erstelle das *LottoService* Image über die Azure Container Registry
    ```
    IMAGE_LOTTOSERVICE='dlindemann/lottoservice'
    az acr build \
        --resource-group $RESOURCE_GROUP \
        --registry $ACR_NAME \
        -t $IMAGE_LOTTOSERVICE:v1 \
        -t $IMAGE_LOTTOSERVICE:{{.Run.ID}} \
        ./LottoService
    ```
- Erstelle das *Web* Image über die Azure Container Registry
    ```
    IMAGE_WEB='dlindemann/lottoweb'
    az acr build \
        --resource-group $RESOURCE_GROUP \
        --registry $ACR_NAME \
        -t $IMAGE_WEB:v1 \
        -t $IMAGE_WEB:{{.Run.ID}} \
        ./Web
    ```
- Images prüfen
    ```
    az acr repository list \
        --name $ACR_NAME \
        --output table
    ```

Gerne auch mal im Portal reinschauen, um sich mit der Azure Container Registry vertraut zu machen.

## 4. Deployments erstellen

Damit die Deployments bereitgestellt werden können, muss zuerst der Namespace 'lotto' angelegt werden. Danach wird jeweils ein Deployment pro Image angelegt.  
⚠ Achtung: Der Imagename muss nun auch den Namen der Container Registry enthalten, in der die Images liegen.

- Namespace erstellen
    ```
    kubectl create namespace lotto
    ```
- ACR Server in Variable speichern
    ```
    ACR_SERVER=$(az acr show -g $RESOURCE_GROUP -n $ACR_NAME --query 'loginServer' -o tsv)
    ```
- Deployment für *RandomNumberService*
    ```
    # image name sample: myacr.azurecr.io/dlindemann/randomnumberservice:v1
    kubectl create deployment random-number-service --image=$ACR_SERVER/$IMAGE_RANDOMNUMBERSERVICE:v1 -n lotto
    ```
- Deployment für *LottoService*
    ```
    # image name sample: myacr.azurecr.io/dlindemann/lottoservice:v1
    kubectl create deployment lotto-service --image=$ACR_SERVER/$IMAGE_LOTTOSERVICE:v1 -n lotto
    ```
- Deployment für *Web*
    ```
    # image name sample: myacr.azurecr.io/dlindemann/lottoweb:v1
    kubectl create deployment lotto-web --image=$ACR_SERVER/$IMAGE_WEB:v1 -n lotto
    ```
- Deployments prüfen
    ```
    kubectl get pods,deployments -n lotto
    ```
- *Web* Pods inspizieren
    ```
    kubectl describe pod lotto-web-<pod name postfix>
    ```
- *Web* Pods logs einsehen
    ```
    kubectl log pod -l app=lotto-web --all-containers=true
    ```
- *Web* Pods löschen
    ```
    kubectl delete pod <pod name>
    ```
