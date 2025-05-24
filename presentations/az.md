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
