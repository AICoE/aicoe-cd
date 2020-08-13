# Add namespace to cluster

## Prerequisites

* sops 3.6+
* imported aicoe-sre gpg key

## Instructions

Namespaces are added to ArgoCD by altering the corresponding cluster spec. Cluster specs are defined within the [clusters](../../manifests/overlays/prod/secrets/clusters) folder.

Open the file in the sops editor, for example if updating the cluster spec `apps.ocp.prod.psi.redhat.com.enc.yaml` you would execute:
```bash
# From repo root
$ cd manifests/overlays/prod/resources/secrets/clusters
$ sops apps.ocp.prod.psi.redhat.com.enc.yaml
```
This should open the decrypted form of the cluster spec. Update the namespace field by appending your namespace (comma-separated, no spaces, if there are multiple namespaces).

```yaml
...
 10 stringData:
 11     name: apps.ocp.prod.psi.redhat.com
 12     config: ...
 14     namespaces: namespace-1,namespace-2 # Update this field by appending the new namespace
 15     server: ...
```

Once done, submit a PR with this change.

If your cluster isn't there then it will need to be added by a member of the aicoe-sre team. See [here](add_new_cluster_spec.md) for more info.
