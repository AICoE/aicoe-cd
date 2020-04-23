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
ArgoCD project to production:

```bash
pipenv shell
pipenv install
ansible-playbook --ask-vault-pass playbook.yaml \
  -e kubeconfig=$HOME/.kube/config \
  -e target_env=prod
```

## Dev Notes 

### AICoE's ArgoCD container image

This repository is containing additions to the ArgoCD container image:

* KSOPS

### KOPS

To add KOPS support we just follow [the README](https://github.com/viaduct-ai/kustomize-sops#argo-cd-integration-)

### Releases

The container image is build manually and pushed to [Quay](https://quay.io/repository/aicoe/argocd)

### GCP credentials

To decrypt secrets we use some GCP key store, therefore we need to add the default application credentials that will
enable ArgoCD to access the keystore:

```
oc -n argocd create secret  generic  gcp --from-file=application-default-credentials=application_default_credentials.json
```

### Creating a gpg secret

Export the key
```
gpg --export-secret-keys "${KEY_ID}" | base64 > private.asc
```
Create a `${target-env}-key.yaml` and copy the contents of `private.asc` onto the field. 

`${target-env}-key.yaml`:
```yaml
gpg_key: | 
    # Contents of private.asc
```

Encrypt `${target-env}-key.yaml `using ansible-vault: 
```
ansible-vault encrypt ${target-env}-key.yaml
```

Use the appropriate vault pass when prompted to encrypt the file and store 
the file in `vars/gpg-keys`. 


### Logging into ArgoCD
```
# Login to ArgoCD
ARGOCD_ROUTE=$(oc get route argocd-server -o jsonpath='{.spec.host}')
argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --username admin --password ${ARGOCD_SERVER_PASSWORD}
```

### Change Admin Password
```
argocd --insecure --grpc-web --server ${ARGOCD_ROUTE}:443 account update-password --current-password ${ARGOCD_SERVER_PASSWORD} --new-password
```