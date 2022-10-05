
## Create a resource group

````
RG=kafka-workshop
az group create --name $RG --location eastus
````
## Configure Kubernetes cluster

```
az aks create  --resource-group $RG  --name Cluster01  --node-count 3  --generate-ssh-keys  --node-vm-size Standard_B2s  --enable-managed-identity --enable-addons monitoring --enable-msi-auth-for-monitoring
```

## Configure kubectl 

Once you setup the cluster, you can easily configure kubectl to point to it

```az aks get-credentials --name Cluster01 --resource-group $RG```

## Get nodes 

```kubectl get nodes```
