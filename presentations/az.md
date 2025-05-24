# AZ commands
[<img src="../images/back.png">](../README.md)

## AZ AKS
- az aks get-credentials --resource-group <Resource-Group-Name> --name <Cluster-Name>
- az aks get-credentials --resource-group aks-rg1 --name aksdemo1
- az aks get-credentials --resource-group myResourceGroup --name myManagedCluster --admin
- az aks get-credentials --resource-group aks-rg3 --name aksdemo3 --overwrite-existing


## KUBECTL
kubectl config get-contexts, kubectl reads your kubeconfig file, which typically resides in $HOME/.kube/config.

- kubectl cluster-info
- kubectl config view
- kubectl config get-context
- kubectl config set-context
- kubectl use-context
- kubectl current-context

## Configure KUBECTL
```
# Configure kubectl
az aks get-credentials --resource-group aks-rg3 --name aksdemo3 --overwrite-existing

# View Cluster Information
kubectl cluster-info
URL: https://microsoft.com/devicelogin
Code: H8VP9YE7F (Sample)(View on terminal)
Username: user1aksadmin@stacksimplifygmail.onmicrosoft.com 
Password: @AKSADAuth1011
```

## References
- [az aks](https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest)
- [AKS Cheat Sheet](https://gist.github.com/yokawasa/fd9d9b28f7c79461f60d86c23f615677)
- [az aks command](https://learn.microsoft.com/de-de/cli/azure/aks/command?view=azure-cli-latest)
- [kubectl config](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_config/)
- [Use dynamic configuration in Azure Kubernetes Service](https://learn.microsoft.com/en-us/azure/azure-app-configuration/enable-dynamic-configuration-azure-kubernetes-service)
- [Generate ConfigMap from Azure App Configuration](https://learn.microsoft.com/en-us/azure/aks/azure-app-configuration-quickstart)
- [Configuring Azure Kubernetes Services containers to consume ConfigMap and Secret resources](https://medium.com/@bashaus/4-4-configuring-azure-kubernetes-services-containers-to-consume-configmap-and-secret-resources-9a66314adb1e)
- [*Configuring App Configuration to expose environment variables to Azure Kubernetes Services](https://medium.com/@bashaus/2-4-configuring-app-configuration-to-expose-environment-variables-to-azure-kubernetes-services-273664df35e0)
- [*[3/4] Configuring Key Vault to expose environment variables to Azure Kubernetes Services](https://medium.com/@bashaus/3-4-configuring-key-vault-to-expose-environment-variables-to-azure-kubernetes-services-48b633ec9e67)
- [*Use the Azure Key Vault provider for Secrets Store CSI Driver in an Azure Kubernetes Service (AKS) cluster](https://learn.microsoft.com/en-us/azure/aks/csi-secrets-store-driver)
- [Tutorial: Use dynamic configuration in Azure Kubernetes Service](https://learn.microsoft.com/en-us/azure/azure-app-configuration/enable-dynamic-configuration-azure-kubernetes-service)

[<img src="../images/back.png">](../README.md)