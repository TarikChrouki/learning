# Start k3s with docker-compose

We use docker-compose file from [k3s](https://github.com/k3s-io/k3s)

## To start 

`K3S_TOKEN=${RANDOM}${RANDOM}${RANDOM} docker-compose up -d`

## Make kubectl refer to the settings of the cluster
`export KUBECONFIG=./kubeconfig.yaml`

## get nodes 

`kubectl get nodes`
 
result : 

```tarikchrouki@DESKTOP-VI10PTI:~/dev/k3s$ kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
8d48d51597d1   Ready    control-plane,master   24h   v1.25.0+k3s1
8ebf84cb26dd   Ready    <none>                 24h   v1.25.0+k3s1
```

## install strimzi

### With Kubectl 

```
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
```

## [Install Kubernetes dashboard](https://rancher.com/docs/k3s/latest/en/installation/kube-dashboard/) (Optional)

### Deploying the Kubernetes Dashboard

```
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
k3s kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml
```

### Dashboard RBAC Configuration

Deploy the admin-user configuration :

`kubectl create -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml`

### Obtain the Bearer Token

The jeton will be used to access to dashboard

`k3s kubectl -n kubernetes-dashboard create token admin-user`

### Local Access to the Dashboard
`kubectl proxy`

The Dashboard is now accessible at:

- http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
- Sign In with the admin-user Bearer Token
