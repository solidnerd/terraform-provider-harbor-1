# E2E Tests With Kind

For Quick Develp

## Kind Precondition

Starting the Kind Cluster with [Ingress Support](https://kind.sigs.k8s.io/docs/user/ingress/).

```bash

COPY
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

Using [Nginx](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx) as Ingress Controller.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/ingress-nginx-2.2.0/deploy/static/provider/kind/deploy.yaml
```

**or for a existing Cluster**
kind export kubeconfig

```bash
kubectl create ns harbor
```

## Install the Harbor Chart

Install the Helm Chart from [goharbor/harbor-helm](https://github.com/goharbor/harbor-helm).

```bash
# add helm chart repo (always done if you use the devcontainer)
helm repo add harbor https://helm.goharbor.io

export INGRESSDOMAIN=192-168-178-51.sslip.io 

helm upgrade -i tf-harbor-test harbor/harbor \
    -n harbor \
    --set expose.ingress.hosts.core=harbor.${INGRESSDOMAIN},expose.ingress.hosts.notary=notary.${INGRESSDOMAIN},externalURL=https://harbor.${INGRESSDOMAIN}


# delete the chart
helm delete -n harbor tf-harbor-test
```

## Update the local Provider

```bash
make build
mv terraform-provider-harbor_v1.0_linux_amd64 ~/.terraform.d/plugins/linux_amd64/terraform-provider-harbor
chmod +x ~/.terraform.d/plugins/linux_amd64/terraform-provider-harbor
```