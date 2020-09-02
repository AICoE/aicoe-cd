# Adding a new cluster spec

## Prerequisites

* sops 3.6+
* imported aicoe-sre gpg key

## Instructions

ArgoCD will need a service account present on the cluster for deployments. Where the SA is located is irrelevant, though it's advised to have it be located in its own independent namespace. For consistency name this service account `argocd-manager`.

This workflow may look like this:

```bash
oc login <your_cluster>
oc new-project argocd-manager
oc create sa argocd-manager
```

Get the token for this SA

```bash
SA_TOKEN=`oc sa get-token argocd-manager -n argocd-manager`
```

Create the cluster spec:
```yaml
# manifests/overlays/prod/resources/secrets/clusters/datahub.psi.redhat.com.yaml
apiVersion: v1
kind: Secret
metadata:
    name: cluster-datahub.psi.redhat.com
    labels:
        argocd.argoproj.io/secret-type: cluster
    annotations:
        managed-by: argocd.argoproj.io
type: Opaque
stringData:
    name:  datahub.psi.redhat.com
    config: |
        {"bearerToken": ${SA_TOKEN}, "tlsClientConfig": {"insecure": true}}
    namespaces: namespace_1,namespace_2
    server: https://datahub.psi.redhat.com:44
```

Let's go over what some of the fields in the `stringData` field refer to:

- `name`: Name for this cluster, appears in the ArgoCD UI

- `config`: The token goes here, replace the contents of ${SA_TOKEN} with the one retrieved earlier.

- `namespace`: List of namespaces ArgoCD has permissions to deploy to, comma-separated, no whitespace

- `server`: Cluster api server hostname, can be retrieved by running `oc whoami --show-server`

SOPS encrypt the manifest and store it in the `clusters` directory.
```
$ cd manifests/overlays/prod/resources/secrets/clusters/
$ sops -e datahub.psi.redhat.com.yaml > datahub.psi.redhat.com.enc.yaml
# Delete the cluster_spec.yaml (decrypted version)
$ rm datahub.psi.redhat.com.yaml
```

Once done, submit a PR.

> Note: DO NOT submit a PR with the decrypted cluster spec secret.
