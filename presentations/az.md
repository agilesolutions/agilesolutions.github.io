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

## References AZ CLI
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

## Azure Key Vault Secret and App Config AKS providers
- [MS: Use the Azure Key Vault provider for Secrets Store CSI Driver in an Azure Kubernetes Service (AKS) cluster](https://learn.microsoft.com/en-us/azure/aks/csi-secrets-store-driver)
- [MS: Connect your Azure identity provider to the Azure Key Vault Secrets Store CSI Driver in Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/csi-secrets-store-identity-access?tabs=azure-portal&pivots=access-with-service-connector)
- [Medium: Azure Key Vault integration with Azure Kubernetes Cluster (AKS)](https://medium.com/@sachin.prabhakar.ingle/azure-key-vault-integration-with-azure-kubernetes-cluster-aks-eba940f845cc)
- [DEV.TO: Securing Kubernetes Secrets in AKS: Using Azure Key Vault with Managed and User Assigned Identities](https://dev.to/hkhelil/securing-kubernetes-secrets-in-aks-using-azure-key-vault-with-managed-and-user-assigned-identities-569k)
- [Medium: Securing Secrets: Building a Private-Access Azure Key Vault Integration with Azure Kubernetes Service Using Terraform](https://medium.com/@newfishg/securing-secrets-building-a-private-access-azure-key-vault-integration-with-azure-kubernetes-d9df939dfd88)
- [Remi: Configuring Secret Store CSI Driver with Terraform: A Guide to Secure Secrets Management in Azure Kubernetes Service](https://www.remiceraline.com/blog/configuring-secret-store-csi-driver-with-terraform-a-guide-to-secure-secrets-management-in-azure-kubernetes-service)

[<img src="../images/back.png">](../README.md)