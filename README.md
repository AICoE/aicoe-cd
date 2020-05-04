# ArgoCD deployment (v1.5.2)

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
without requiring cluster-admin, just add the additional `cluster_admin=False` var: 

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

To add KSOPS support we just follow [the README](https://github.com/viaduct-ai/kustomize-sops#argo-cd-integration-)

### Releases

The container image is build manually and pushed to [Quay](https://quay.io/repository/aicoe/argocd)

### GCP credentials

To decrypt secrets we use some GCP key store, therefore we need to add the default application credentials that will
enable ArgoCD to access the keystore:

```bash
oc -n argocd create secret  generic  gcp --from-file=application-default-credentials=application_default_credentials.json
```

### Creating a gpg secret

Export the key

```bash
gpg --export-secret-keys "${KEY_ID}" | base64 > private.asc
```

Create a `${target-env}-key.yaml` and copy the contents of `private.asc` onto the field.

`${target-env}-key.yaml`:

```yaml
gpg_key: |
    # Contents of private.asc
```

Encrypt `${target-env}-key.yaml` using ansible-vault:

```yaml
ansible-vault encrypt ${target-env}-key.yaml
```

Use the appropriate vault pass when prompted to encrypt the file and store
the file in `vars/gpg-keys`.

### Logging into ArgoCD

```bash
ARGOCD_ROUTE=$(oc get route argocd-server -o jsonpath='{.spec.host}')
# Login to ArgoCD as admin user
argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --username admin --password ${ARGOCD_SERVER_PASSWORD}
# Login to ArgoCD via SSO
argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --sso
```

### Change Admin Password

```bash
argocd --insecure --grpc-web --server ${ARGOCD_ROUTE}:443 account update-password --current-password ${ARGOCD_SERVER_PASSWORD} --new-password
```
