# dj-gitops

## create cluster in ui
something like this? [link](https://github.com/konstructio/gitops-template/blob/main/digitalocean-github/terraform/digitalocean/modules/workload-cluster/main.tf#L2C1-L22)
```tf
data "digitalocean_kubernetes_versions" "versions" {
  version_prefix = "1.29."
}

resource "digitalocean_kubernetes_cluster" "cluster" {
  name    = var.cluster_name
  region  = lower(var.cluster_region)
  version = data.digitalocean_kubernetes_versions.versions.latest_version

  node_pool {
    name       = "${var.cluster_name}-node-pool"
    size       = var.node_type
    auto_scale = true
    min_nodes  = var.node_count
    max_nodes  = var.node_count
  }
}
```

## get kubeconfig
```bash
# OR 
export DIGITALOCEAN_TOKEN=
doctl auth init --context beta # beta is my project name in the left gutter
# cluster id, project name 
doctl kubernetes cluster kubeconfig save 781eef8e-4ba0-4afc-a9e3-889a0963c799 --context beta 


kubectl kustomize https://github.com/konstructio/manifests/argocd/cloud\?ref\=v1.1.0 | kubectl apply -f -
```

# create secret for external
make sure you set env up above
```bash
kubectl create ns external-\dns
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: external-dns
---
apiVersion: v1
stringData:
  token: $DIGITALOCEAN_TOKEN
kind: Secret
metadata:
  name: external-dns-secrets
  namespace: external-dns
type: Opaque
EOF
```

# apply the ArgoCD registry yaml
```bash
alias k=kubectl
# assumes mac os
k -n argocd get secrets argocd-initial-admin-secret -ojson | jq -r .data.password | base64 -d | pbcopy"
kubectl port-forward service/argocd-server -n argocd 8888:443
# visit localhost:8888
# username: root
# password from previous command / on clipboard
# assumes my upstream and no changes but just the raw link from the file view in github
kubectl apply -f https://raw.githubusercontent.com/jarededwards/dj-gitops/refs/heads/main/registry/clusters/dj-test/registry.yaml
```

# add an app, ask for that next
