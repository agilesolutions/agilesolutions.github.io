# AZ commands
[<img src="../images/back.png">](../README.md)

## AZ AKS
- az aks get-credentials --resource-group <Resource-Group-Name> --name <Cluster-Name>
- az aks get-credentials --resource-group aks-rg1 --name aksdemo1
- az aks get-credentials --resource-group myResourceGroup --name myManagedCluster --admin
- az aks get-credentials --resource-group aks-rg3 --name aksdemo3 --overwrite-existing
- az aks enable-addons
- az aks list-addons
- az aks enable-addons --addons azure-keyvault-secrets-provider --name myAKSCluster --resource-group myResourceGroup
- az aks update -n MyAKSCluster -g MyResourceGroup --enable-secret-store-csi-driver
- kubectl get pods -n kube-system
- kubectl get pods -n kube-system -l 'app in (secrets-store-csi-driver,secrets-store-provider-azure)'


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
- [AZ AKS ADDON](https://learn.microsoft.com/en-us/cli/azure/aks/addon?view=azure-cli-latest)
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

## Azure Secret Store CSI Driver
What is Kubernetes CSI (Container Storage Interface)? CSI is a standard that helps provide drivers as an extension to expose arbitrary block and file storage systems to containerized workloads on Kubernetes,
and establish connectivity between storage systems and Kubernetes. Azure makes this integration straightforward by allowing you to enable the **Secret Store CSI Driver** directly on your AKS cluster.

- [MS: Use the Azure Key Vault provider for Secrets Store CSI Driver in an Azure Kubernetes Service (AKS) cluster](https://learn.microsoft.com/en-us/azure/aks/csi-secrets-store-driver)
- [MS: Connect your Azure identity provider to the Azure Key Vault Secrets Store CSI Driver in Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/csi-secrets-store-identity-access?tabs=azure-portal&pivots=access-with-service-connector)
- [Medium: Azure Key Vault integration with Azure Kubernetes Cluster (AKS)](https://medium.com/@sachin.prabhakar.ingle/azure-key-vault-integration-with-azure-kubernetes-cluster-aks-eba940f845cc)
- [DEV.TO: Securing Kubernetes Secrets in AKS: Using Azure Key Vault with Managed and User Assigned Identities](https://dev.to/hkhelil/securing-kubernetes-secrets-in-aks-using-azure-key-vault-with-managed-and-user-assigned-identities-569k)
- [Medium: Securing Secrets: Building a Private-Access Azure Key Vault Integration with Azure Kubernetes Service Using Terraform](https://medium.com/@newfishg/securing-secrets-building-a-private-access-azure-key-vault-integration-with-azure-kubernetes-d9df939dfd88)
- [Remi: Configuring Secret Store CSI Driver with Terraform: A Guide to Secure Secrets Management in Azure Kubernetes Service](https://www.remiceraline.com/blog/configuring-secret-store-csi-driver-with-terraform-a-guide-to-secure-secrets-management-in-azure-kubernetes-service)

## Azure App Configuration Kubernetes provider
The Azure App Configuration Kubernetes provider runs as a container in your cluster.
- Seamless integration: Pulls data from Azure App Configuration and Key Vault, making them accessible as ConfigMap and Secret without code changes in your workloads.
- Dynamic update: Built-in caching and refreshing capabilities for dynamic configuration, feature flagging, and automatic secret rotation.

### Read
- [Medium: Azure Ap Configuration service](https://medium.com/@barbieri.santiago/azure-app-configuration-service-2ffe9b31e914)
- [Azure App Configuration Kubernetes Provider reference](https://learn.microsoft.com/en-us/azure/azure-app-configuration/reference-kubernetes-provider?tabs=default)
- [Medium: Configuring App Configuration to expose environment variables to Azure Kubernetes Services](https://medium.com/@bashaus/2-4-configuring-app-configuration-to-expose-environment-variables-to-azure-kubernetes-services-273664df35e0)
- [Quickstart: Generate ConfigMap from Azure App Configuration](https://learn.microsoft.com/en-us/azure/aks/azure-app-configuration-quickstart)
- [HELM: Azure App Configuration Kubernetes Provider](https://mcr.microsoft.com/artifact/mar/azure-app-configuration/kubernetes-provider/about)

**NOTE: ** Azure App Configuration Kubernetes Provider is a Kubernetes controller that integrates Azure App Configuration with Kubernetes. The provider gets key-values and Key Vault references from Azure App Configuration and builds them into Kubernetes ConfigMaps and Secrets.
This will configure new services via a helm chart that will be applied to your azappconfig-system namespace. To test if the installation was successful, simply run:
```
helm install azureappconfiguration.kubernetesprovider \
     oci://mcr.microsoft.com/azure-app-configuration/helmchart/kubernetes-provider \
     --namespace azappconfig-system \
     --create-namespace
     
kubectl get azureappconfigurationprovider
```
## Container logs and AZ Monitor
What is Kubernetes CSI (Container Storage Interface)? CSI is a standard that helps provide drivers as an extension to expose arbitrary block and file storage systems to containerized workloads on Kubernetes,
and establish connectivity between storage systems and Kubernetes

1. By default, Spring Boot uses Logback as its logging framework. To switch to Log4j, you'll need to exclude Logback and include Log4j in your project.
2. As a default, Docker uses the json-file logging driver, which caches container logs as JSON internally.
3. Azure Monitor Container insights collect custom metrics for nodes, pods, containers, and persistent volumes.
   ---- Container Insights is a feature of Azure Monitor that collects and analyzes container logs from Azure Kubernetes clusters
4. Container insights store most of its data in a Log Analytics workspace, and you typically use the same log analytics workspace as the resource logs for your cluster.
   --- Kubernetes manages the symlinks in /var/log/container/* when Pod containers start or stop to point to the logfile of the underlying container runtime.

Azure Monitor Container insights collect custom metrics for nodes, pods, containers, and persistent volumes. For more information, see Metrics collected by Container insights.

Azure Monitor is a comprehensive monitoring solution for collecting, analyzing, and responding to monitoring data from your cloud and on-premises environments.
Log Analytics is a tool in the Azure portal to edit and run log queries from data collected by Azure Monitor logs

[<img src="../images/back.png">](../README.md)