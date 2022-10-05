
# Install k3d

```bash
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```

# Create kubernetes cluster

```bash
$ k3d cluster create mycluster --agents 3 -v $HOME/temp/strimzi:/var/lib/rancher/k3s/storage@all 
$ kubectl cluster-info
$ kubectl taint nodes k3d-mycluster-server-0 key1=value1:NoSchedule
$ kubectl create ns kafka
```

## Get nodes

```bash
kubectl get nodes
```
