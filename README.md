# ArgoCD deployment

This repository contains the necessary artifacts to deploy ArgoCD to dev and prod.

## Deployment Instructions

### Prerequisites
Kustomize 3.8.1+
SOPS 3.4.0+
KSOPS 2.1.2+

Ensure you have the key to decrypt secrets. Reach out to members of the Data Hub team for access.

### Deploying to Development

See [the docs](docs/setup_argocd_dev_enviornment.md) for instructions on how to setup a development environment.

### Deploying to Production
To Deploy CRDs run (cluster admin required):
```
kustomize build manifests/crds --enable_alpha_plugins | oc apply -f -
```

To deploy to production run the following:
```
kustomize build manifests/overlays/prod --enable_alpha_plugins | oc apply -f -
```

To deploy ArgoCD applications/projects you will need to login as the `argocd-application-controller` SA first:

```
oc login --token $(oc sa get-token argocd-application-controller)
kustomize build manifests/overlays/prod/privileged_resources/ --enable_alpha_plugins | oc apply -f -
```

## Dev Notes

### AICoE's ArgoCD container image

This repository is containing additions to the ArgoCD container image:

* KSOPS

### KSOPS

To add KSOPS support we just followed [the README](https://github.com/viaduct-ai/kustomize-sops#argo-cd-integration-)

### Releases

The container image is built manually and pushed to [Quay](https://quay.io/repository/aicoe/argocd)

### Logging into ArgoCD

```bash
ARGOCD_ROUTE=$(oc get route argocd-server -o jsonpath='{.spec.host}')
# Login to ArgoCD as admin user
argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --username admin --password ${ARGOCD_SERVER_PASSWORD}
# Login to ArgoCD via SSO
argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --sso
```

Note: You can find your cluster-contexts by going into your kubeconfig or running `kubectl config current-context`
