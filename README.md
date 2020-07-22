# ArgoCD deployment

This repository contains the necessary artifacts to deploy ArgoCD to dev and prod.

## Deployment Instructions

### Prerequisites

We use [pipenv](https://pipenv.readthedocs.io/en/latest/) to manage our
python requirements and to ensure we're all using consistent versions of
packages. Install pipenv by running the following command:

```bash
pip install --user pipenv
```

### Deploying to Development

Run the following command from the root of this repository to deploy the
ArgoCD project in a development environment:

```bash
pipenv shell
pipenv install
ansible-playbook --ask-vault-pass playbook.yaml \
  -e kubeconfig=$HOME/.kube/config \
  -e target_env=dev
```

### Deploying to Production

Run the following command from the root of this repository to deploy the
ArgoCD project to production (need cluster-admin):

```bash
pipenv shell
pipenv install
ansible-playbook --ask-vault-pass playbook.yaml \
  -e kubeconfig=$HOME/.kube/config \
  -e target_env=prod
```

### Ignore cluster objects

If you want to just update deployment/configs or other namespace scoped objects
without requiring cluster-admin, just add the additional `deploy_cluster_objects=False` var:

```bash
pipenv shell
pipenv install
ansible-playbook --ask-vault-pass playbook.yaml \
  -e cluster_admin=False \
  -e kubeconfig=$HOME/.kube/config \
  -e target_env=prod
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
