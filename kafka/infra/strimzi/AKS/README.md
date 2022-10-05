
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

## Create namespace 
- kafka for our project (operator, cluster...)

```
kubectl create ns kafka
```

## install strimzi 

### Kubectl (first way)

```
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
```

### Helm (Second way)

Installing Strimzi using Helm is pretty easy:
````
# Add the Strimzi Helm Chart repository:
helm repo add strimzi https://strimzi.io/charts/

# To install the chart with the release name strimzi-kafka
helm install strimzi-kafka strimzi/strimzi-kafka-operator --namespace kafka
````

you should get the output : 
```bash
NAME: strimzi-kafka
LAST DEPLOYED: Thu Sep 22 13:55:28 2022
NAMESPACE: kafka
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing strimzi-kafka-operator-0.31.1

To create a Kafka cluster refer to the following documentation.

https://strimzi.io/docs/operators/latest/deploying.html#deploying-cluster-operator-helm-chart-str
```

To confirm that the Strimzi Operator had been deployed, check it’s Pod
```
$ kubectl get pods -l=name=strimzi-cluster-operator -A
NAMESPACE   NAME                                        READY   STATUS    RESTARTS   AGE
kafka     strimzi-cluster-operator-5f78bcb85d-p9sh7   1/1     Running   0          113s
```