# Lösung Challenge 03 - Eigene Applikation in AKS bereitstellen

[Hint: Die verwendeten Parameter wurden in [*Challenge 02 - Kubernetes Cluster über Azure CLI erstellen*](../challenge-02.md) angelegt]

## 1. Erstelle eine Container Registry

Für das bereitstellen eigener Applikationen benötigt es eine Container Registry. Die bekannteste ist der [Docker Hub](https://hub.docker.com). Hier liegen viele öffentliche Container Images mit denen Applikationen bereitgestellt werden können.
Damit eigene private Container Images bereitgestellt werden können, gibt es die [Azure Container Registry](https://azure.microsoft.com/de-de/services/container-registry/)

Bash:
```bash
ACR_NAME=acrabtisworkshop$AKS_RANDOM
az acr create \
    --resource-group $RESOURCE_GROUP \
    --location $REGION_NAME \
    --name $ACR_NAME \
    --sku Basic
```

PowerShell:
```powershell
$ACR_NAME="acrabtisworkshop$AKS_RANDOM"
az acr create `
    --resource-group $RESOURCE_GROUP `
    --location $REGION_NAME `
    --name $ACR_NAME `
    --sku Basic
```

## 2. Container Registry mit AKS verknüpfen

Damit Images aus der Container Registry durch Kubernetes geladen werden können, müssen die beiden miteinander verheiratet werden. Das geht am einfachsten über die Azure CLI.

Bash:
```bash
az aks update \
    --name $AKS_CLUSTER_NAME \
    --resource-group $RESOURCE_GROUP \
    --attach-acr $ACR_NAME
```

PowerShell:
```powershell
az aks update `
    --name $AKS_CLUSTER_NAME `
    --resource-group $RESOURCE_GROUP `
    --attach-acr $ACR_NAME
```

## 3. Images in Container Registry pushen

Es gibt verschiedene Möglichkeiten die Images in eine Azure Container Registry zu pushen. Entscheide selbst, wie du es machen möchtest. Hier werden 2 Alternativen aufgezeigt:

- 3.A Erstellen der Images mit Quellcode und Azure ACR
- 3.B Ziehen der Images aus DockerHub


### 3.A Erstellen der Images mit Quellcode und Azure ACR

Über die Azure Container Registry können Images direkt gebaut werden, ohne dass Docker oder eine andere Container Runtime auf dem Client installiert werden muss.

- Lade das Repo [daniellindemann/lotto-microservices](https://github.com/daniellindemann/lotto-microservices/)

    ```
    git clone https://github.com/daniellindemann/lotto-microservices.git
    ```

- Wechsele in den `src` Ordner des `lotto-microservices` Repos

    Bash/PowerShell:
    ```
    cd lotto-microservices/src
    ```

- Erstelle das *RandomNumberService* Image über die Azure Container Registry

    Bash:
    ```bash
    IMAGE_RANDOMNUMBERSERVICE='daniellindemann/lotto-randomnumberservice'
    az acr build \
        --resource-group $RESOURCE_GROUP \
        --registry $ACR_NAME \
        -t $IMAGE_RANDOMNUMBERSERVICE:1.0.0 \
        -t $IMAGE_RANDOMNUMBERSERVICE:{{.Run.ID}} \
        ./RandomNumberService
    ```

    PowerShell:
    ```powershell
    $IMAGE_RANDOMNUMBERSERVICE="daniellindemann/lotto-randomnumberservice"
    az acr build `
        --resource-group $RESOURCE_GROUP `
        --registry $ACR_NAME `
        -t "$IMAGE_RANDOMNUMBERSERVICE:1.0.0" `
        -t "$IMAGE_RANDOMNUMBERSERVICE:{{.Run.ID}}" `
        ./RandomNumberService
    ```

- Erstelle das *LottoService* Image über die Azure Container Registry

    Bash:
    ```bash
    IMAGE_LOTTOSERVICE='daniellindemann/lotto-lottoservice'
    az acr build \
        --resource-group $RESOURCE_GROUP \
        --registry $ACR_NAME \
        -t $IMAGE_LOTTOSERVICE:1.0.0 \
        -t $IMAGE_LOTTOSERVICE:{{.Run.ID}} \
        ./LottoService
    ```

    PowerShell:
    ```powershell
    $IMAGE_LOTTOSERVICE="daniellindemann/lotto-lottoservice"
    az acr build `
        --resource-group $RESOURCE_GROUP `
        --registry $ACR_NAME `
        -t "$IMAGE_LOTTOSERVICE:1.0.0" `
        -t "$IMAGE_LOTTOSERVICE:{{.Run.ID}}" `
        ./LottoService
    ```

- Erstelle das *Web* Image über die Azure Container Registry

    Bash:
    ```bash
    IMAGE_WEB='daniellindemann/lotto-web'
    az acr build \
        --resource-group $RESOURCE_GROUP \
        --registry $ACR_NAME \
        -t $IMAGE_WEB:1.0.0 \
        -t $IMAGE_WEB:{{.Run.ID}} \
        ./Web
    ```

    PowerShell:
    ```powershell
    $IMAGE_WEB="daniellindemann/lotto-web"
    az acr build `
        --resource-group $RESOURCE_GROUP `
        --registry $ACR_NAME `
        -t "$IMAGE_WEB:1.0.0" `
        -t "$IMAGE_WEB:{{.Run.ID}}" `
        ./Web
    ```

- Images prüfen

    Bash/PowerShell:
    ```bash
    az acr repository list --name $ACR_NAME --output table
    ```

    Gerne auch mal im Portal reinschauen, um sich mit der Azure Container Registry vertraut zu machen.

### 3.B Ziehen der Images aus DockerHub

- Import *RandomNumberService* Image von Docker Hub in die Azure Container Registry

    Bash:
    ```bash
    IMAGE_RANDOMNUMBERSERVICE='daniellindemann/lotto-randomnumberservice'
    az acr import \
        --resource-group $RESOURCE_GROUP \
        --name $ACR_NAME \
        --source docker.io/$IMAGE_RANDOMNUMBERSERVICE:1.0.0
    ```

    PowerShell:
    ```powershell
    $IMAGE_RANDOMNUMBERSERVICE="daniellindemann/lotto-randomnumberservice"
    az acr import `
        --resource-group $RESOURCE_GROUP `
        --name $ACR_NAME `
        --source "docker.io/$IMAGE_RANDOMNUMBERSERVICE`:1.0.0"
    ```

- Import *LottoService* Image von Docker Hub in die Azure Container Registry

    Bash:
    ```bash
    IMAGE_LOTTOSERVICE='daniellindemann/lotto-lottoservice'
    az acr import \
        --resource-group $RESOURCE_GROUP \
        --name $ACR_NAME \
        --source docker.io/$IMAGE_LOTTOSERVICE:1.0.0
    ```

    PowerShell:
    ```powershell
    $IMAGE_LOTTOSERVICE="daniellindemann/lotto-lottoservice"
    az acr import `
        --resource-group $RESOURCE_GROUP `
        --name $ACR_NAME `
        --source "docker.io/$IMAGE_LOTTOSERVICE`:1.0.0"
    ```

- Import *Web* Image von Docker Hub in die Azure Container Registry

    Bash:
    ```bash
    IMAGE_WEB='daniellindemann/lotto-web'
    az acr import \
        --resource-group $RESOURCE_GROUP \
        --name $ACR_NAME \
        --source docker.io/$IMAGE_WEB:1.0.0
    ```

    PowerShell:
    ```powershell
    $IMAGE_WEB="daniellindemann/lotto-web"
    az acr import `
        --resource-group $RESOURCE_GROUP `
        --name $ACR_NAME `
        --source "docker.io/$IMAGE_WEB`:1.0.0"
    ```

- Images prüfen

    Bash/PowerShell:
    ```bash
    az acr repository list --name $ACR_NAME --output table
    ```

    Gerne auch mal im Portal reinschauen, um sich mit der Azure Container Registry vertraut zu machen.

## 4. Deployments erstellen

Damit die Deployments bereitgestellt werden können, muss zuerst der Namespace 'lotto' angelegt werden. Danach wird jeweils ein Deployment pro Image angelegt.
⚠ Achtung: Der Imagename muss nun auch den Namen der Container Registry enthalten, in der die Images liegen.

- Namespace erstellen

    Bash/PowerShell:
    ```bash
    kubectl create namespace lotto
    ```

- ACR Server in Variable speichern

    Bash:
    ```bash
    ACR_SERVER=$(az acr show -g $RESOURCE_GROUP -n $ACR_NAME --query 'loginServer' -o tsv)
    ```

    PowerShell:
    ```powershell
    $ACR_SERVER=(az acr show -g $RESOURCE_GROUP -n $ACR_NAME --query 'loginServer' -o tsv)
    ```

- Deployment für *RandomNumberService*

    > Image name sample: myacr.azurecr.io/daniellindemann/lotto-randomnumberservice:1.0.0

    Bash:
    ```bash
    kubectl create deployment random-number-service --image=$ACR_SERVER/$IMAGE_RANDOMNUMBERSERVICE:1.0.0 --replicas=3 -n lotto
    ```

    PowerShell:
    ```powershell
    kubectl create deployment random-number-service --image="$ACR_SERVER/$IMAGE_RANDOMNUMBERSERVICE`:1.0.0" --replicas=3 -n lotto --replicas=3 -n lotto
    ```

- Deployment für *LottoService*

    > Image name sample: myacr.azurecr.io/daniellindemann/lotto-lottoservice:1.0.0

    Bash:
    ```bash
    kubectl create deployment lotto-service --image=$ACR_SERVER/$IMAGE_LOTTOSERVICE:1.0.0 --replicas=3 -n lotto
    ```

    PowerShell:
    ```PowerShell
    kubectl create deployment lotto-service --image="$ACR_SERVER/$IMAGE_LOTTOSERVICE`:1.0.0" --replicas=3 -n lotto
    ```


- Deployment für *Web*

    > Image name sample: myacr.azurecr.io/daniellindemann/lotto-web:1.0.0

    Bash:
    ```bash
    kubectl create deployment lotto-web --image=$ACR_SERVER/$IMAGE_WEB:1.0.0 --replicas=3 -n lotto
    ```

    PowerShell:
    ```powershell
    kubectl create deployment lotto-web --image="$ACR_SERVER/$IMAGE_WEB`:1.0.0" --replicas=3 -n lotto
    ```

- Deployments prüfen
    ```
    kubectl get pods,deployments -n lotto
    ```
- *Web* Pods inspizieren
    ```
    kubectl describe pod lotto-web-<pod name postfix> -n lotto
    ```
- *Web* Pods logs einsehen
    ```
    kubectl logs -n lotto lotto-web-<pod name postfix>
    ```
- *Web* Pods löschen
    ```
    kubectl delete pod <pod name> -n lotto
    ```
- Pods prüfen
    ```
    kubectl get pods -n lotto
    ```
