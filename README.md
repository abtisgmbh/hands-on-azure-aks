# hands-on-azure-aks
A collection of hands on labs to get familiar with Azure Kubernetes Services and Kubernetes on Azure

## Hands on labs

The hands on labs in in german. You can find alle Challenges in the `challenges` files.

1. [Challenge 01 - Kubernetes Cluster erstellen](challenge-01.md)
2. [Challenge 02 - Kubernetes Cluster über Azure CLI erstellen](challenge-02.md)
3. [Challenge 03 - Eigene Applikation in AKS bereitstellen](challenge-03.md)
4. [Challenge 04 - Applikation über Services veröffentlichen](challenge-04.md)
5. [Challenge 05 - Monitoring und Logging](challenge-05.md)
6. [Challenge 06 - Horizontal Pod Autoscaling](challenge-06.md)


## Clean up
Don't forget to clean up the resources you created during this lab, otherwise can generate costs you're not aware of.

```
az group delete --name $RESOURCE_GROUP --yes --no-wait
```