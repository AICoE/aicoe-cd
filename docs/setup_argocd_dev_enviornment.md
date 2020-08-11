# Deploying a development environment

## Prequisites
* An OCP 4.x Development cluster
* Must have cluster admin

## Instructions

Create the project `aicoe-argocd-dev` and `argocd-test`. The latter wll be used
for deploying a dev application via ArgoCD.

```bash
oc new-project argocd-test
oc new-project aicoe-argocd-dev
```

Deploy ArgoCD
```bash
git clone git@github.com:AICoE/aicoe-cd.git
cd aicoe-cd

# Deploy Cluser objects
kustomize build manifests/crds --enable_alpha_plugins | oc apply -f -

# Deploy Non Cluster objects
kustomize build manifests/overlays/dev --enable_alpha_plugins | oc apply -f -
```

## Configure Auth
Once deployed, there are some additional configurations, run this script:
```bash
examples/configure_development.sh
```
Feel free to look inside the script for detailed comments on what configurations are applied.

## Cleanup
Run the following commands to clean up your environment.

```
kustomize build manifests/overlays/dev --enable_alpha_plugins | oc delete -f -
kustomize build manifests/crds --enable_alpha_plugins | oc delete -f -
oc delete group dev
oc delete project argocd-test
oc delete project aicoe-argocd-dev
```

You may ignore the following error when removing secrets:

```
Error from server (NotFound): error when deleting "STDIN": secrets "argocd-dex-server-oauth-token" not found
Error from server (NotFound): error when deleting "STDIN": secrets "dev-cluster-spec" not found
```
