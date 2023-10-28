# Development Cluster

## Secrets

### Gitlab Deploy Key
```sh
ssh-keygen -t ecdsa -f github
kubectl -n flux-system create secret generic sops-age --from-file=age.agekey=age.agekey --dry-run=client -o yaml >age-key.yaml
sops --encrypt github-deploy-key.yaml > kubernetes/bootstrap/flux/github-deploy-key.sops.yaml
sops --decrypt kubernetes/bootstrap/flux/github-deploy-key.sops.yaml | oc apply -f -
```

add key to Gitlab repo

### SOPS AGE encryption key
```sh
age-keygen -o age.agekey
kubectl -n flux-system create secret generic sops-age --from-file=age.agekey=age.agekey --dry-run=client -o yaml >age-key.yaml
sops --encrypt age-key.yaml > kubernetes/bootstrap/flux/age-key.sops.yaml
sops --decrypt kubernetes/bootstrap/flux/age-key.sops.yaml | oc apply -f -
```

## Flux

### Install Flux

get your token after logging in to the cluster

```sh
oc login --token=sha256~pTLPMEs_...snip..._pthkBiyfHTS0E --server=https://api.openshift.your.domain:6443
oc apply --server-side --kustomize ./kubernetes/bootstrap/flux
```

```sh
oc apply --server-side --kustomize ./kubernetes/flux/config
```

```sh
FLUX_CONTROLLERS=(
"source-controller"
"kustomize-controller"
"helm-controller"
"notification-controller"
"image-reflector-controller"
"image-automation-controller"
)

for i in ${!FLUX_CONTROLLERS[@]}; do
  oc adm policy add-scc-to-user nonroot system:serviceaccount:${FLUX_NAMESPACE}:${FLUX_CONTROLLERS[$i]}
done
```
