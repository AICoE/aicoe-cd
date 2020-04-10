# AICoE's ArgoCD container image

This repository is containing additions to the ArgoCD container image:

* KSOPS

## KOPS

To add KOPS support we just follow [the README](https://github.com/viaduct-ai/kustomize-sops#argo-cd-integration-)

## Releases

The container image is build manually and pushed to [Quay](https://quay.io/repository/aicoe/argocd)

## GCP credentials

To decrypt secrets we use some GCP key store, therefore we need to add the default application credentials that will
enable ArgoCD to access the keystore:

```
oc -n argocd create secret  generic  gcp --from-file=application-default-credentials=application_default_credentials.json
```
