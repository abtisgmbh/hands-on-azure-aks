# Lösung Challenge 02 - Kubernetes Cluster über Azure CLI erstellen

## 1. Mit Azure verbinden

Verbinde dich mit Azure und wähle die richtige Subscription für das Deployment aus

```
az login
az account list
az account set -s <SUBSCRIPTION_ID>
```

## 2. Cluster erstellen

### 2.1 Allgemeine Variable festlegen

Erstellen von Variablen die zur Ausführung der nächsten Schritte notwendig sind

Bash:
```bash
AKS_RANDOM=$RANDOM
REGION_NAME=northeurope
RESOURCE_GROUP=rg-abtis-workshop-$AKS_RANDOM
```

PowerShell:
```powershell
$AKS_RANDOM=Get-Random -Maximum 99999
$REGION_NAME="northeurope"
$RESOURCE_GROUP="rg-abtis-workshop-$AKS_RANDOM"
```

### 2.2 Resource Group erstellen

Bash:
```bash
az group create \
    --name $RESOURCE_GROUP \
    --location $REGION_NAME
```

PowerShell:
```powershell
az group create `
    --name $RESOURCE_GROUP `
    --location $REGION_NAME
```

### 2.3 AKS Konfiguration

Die Konfiguration von AKS wird natürlich über Variablen gesteuert. Also noch mehr davon.
Prüft die IP Adressbereiche, dass sie auch so möglich sind.

Bash:
```bash
SUBNET_NAME=snet-abtis-workshop-$AKS_RANDOM-workloads
VNET_NAME=vnet-abtis-workshop-$AKS_RANDOM
VNET_ADDRESS_PREFIX=10.100.0.0/16
VNET_ADRESS_SUBNET_PREFIX=10.100.0.0/24
AKS_CLUSTER_NAME=aks-abtis-workshop-$AKS_RANDOM
AKS_SERVICE_CIDR=10.100.254.0/24
AKS_DNS_SERVICE_IP=10.100.254.254
```

PowerShell:
```powershell
$SUBNET_NAME="snet-abtis-workshop-$AKS_RANDOM-workloads"
$VNET_NAME="vnet-abtis-workshop-$AKS_RANDOM"
$VNET_ADDRESS_PREFIX="10.100.0.0/16"
$VNET_ADRESS_SUBNET_PREFIX="10.100.0.0/24"
$AKS_CLUSTER_NAME="aks-abtis-workshop-$AKS_RANDOM"
$AKS_SERVICE_CIDR="10.100.254.0/24"
$AKS_DNS_SERVICE_IP="10.100.254.254"
```

### 2.4 VNET Konfiguration für Workload-Netzwerk

In einer Best Practices Konfiguration haben Pods ihr eigenes Netzwerk. Das muss angelegt werden.

Bash:
```bash
az network vnet create \
    --resource-group $RESOURCE_GROUP \
    --location $REGION_NAME \
    --name $VNET_NAME \
    --address-prefixes $VNET_ADDRESS_PREFIX \
    --subnet-name $SUBNET_NAME \
    --subnet-prefixes $VNET_ADRESS_SUBNET_PREFIX
SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)
```

PowerShell:
```powershell
az network vnet create `
    --resource-group $RESOURCE_GROUP `
    --location $REGION_NAME `
    --name $VNET_NAME `
    --address-prefixes $VNET_ADDRESS_PREFIX `
    --subnet-name $SUBNET_NAME `
    --subnet-prefixes $VNET_ADRESS_SUBNET_PREFIX
$SUBNET_ID=(az network vnet subnet show `
    --resource-group $RESOURCE_GROUP `
    --vnet-name $VNET_NAME `
    --name $SUBNET_NAME `
    --query id -o tsv)
```

### 2.5 AKS Cluster erstellen

Cluster erstellen. Endlich ...

Bash:
```bash
VERSION=$(az aks get-versions \
    --location $REGION_NAME \
    --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' \
    --output tsv)

az aks create \
    --resource-group $RESOURCE_GROUP \
    --name $AKS_CLUSTER_NAME \
    --vm-set-type VirtualMachineScaleSets \
    --node-count 3 \
    --node-vm-size Standard_B2ms \
    --load-balancer-sku standard \
    --location $REGION_NAME \
    --kubernetes-version $VERSION \
    --network-plugin azure \
    --vnet-subnet-id $SUBNET_ID \
    --service-cidr $AKS_SERVICE_CIDR \
    --dns-service-ip $AKS_DNS_SERVICE_IP \
    --docker-bridge-address 172.17.0.1/16 \
    --generate-ssh-keys
```

PowerShell:
```powershell
$VERSION=(az aks get-versions `
    --location $REGION_NAME `
    --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' `
    --output tsv)

az aks create `
    --resource-group $RESOURCE_GROUP `
    --name $AKS_CLUSTER_NAME `
    --vm-set-type VirtualMachineScaleSets `
    --node-count 3 `
    --node-vm-size Standard_B2ms `
    --load-balancer-sku standard `
    --location $REGION_NAME `
    --kubernetes-version $VERSION `
    --network-plugin azure `
    --vnet-subnet-id $SUBNET_ID `
    --service-cidr $AKS_SERVICE_CIDR `
    --dns-service-ip $AKS_DNS_SERVICE_IP `
    --docker-bridge-address 172.17.0.1/16 `
    --generate-ssh-keys
```

### 2.6 Kubernetes CLI installieren

Falls noch nicht vorhanden, muss die Kubernetes CLI kubectl installiert werden.

Bash/PowerShell:
```bash
az aks install-cli
```

### 2.7 In Kubernetes Cluster einloggen

Bash/PowerShell:
```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME
```

Ein kleiner Test zur Prüfung des Cluster ist nicht verkehrt.

Bash/PowerShell:
```bash
kubectl get nodes
```

Ein Zweiter ist auch nicht schlecht

Bash/PowerShell:
```bash
kubectl get pods --all-namespaces
```

## 3. Erste Workloads bereitstellen

nginx bereitstellen, um die Lauffähigkeit zu prüfen

Bash/PowerShell:
```bash
kubectl run nginx --image=nginx
kubectl get pods
kubectl exec nginx -- nginx -v
```

Erstelle einen Port-forward und öffne die nginx Website über Port `8080`.

Bash/PowerShell:
```bash
kubectl port-forward pod/nginx 8080:80
```

> ℹ️ <http://localhost:8080>

## 4. Extra: Yaml Datei Deployment

kubectl kann dabei unterstützen yaml-Dateien zu erstellen.

Bash/PowerShell:
```bash
kubectl run nginx2 --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

Schau dir die Datei an.

Bash/PowerShell:
```bash
cat nginx-pod.yaml
```

Erstelle einen neuen nginx Pod über das Manifest.

Bash/PowerShell:
```bash
kubectl apply -f nginx-pod.yaml
```
