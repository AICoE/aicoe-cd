# Production Runbook for ArgoCD

## Key Locations

Production Namespace: aicoe-argocd
Production Cluster: This information can be found in the internal Runbooks repository.

## Common Debugging Steps

### Aplication is stuck will not sync and/or is stuck on refresh

Check work queue metric, it is possible that thread count is bottlenecked,
restart the application controller pod.

### Excluded resource warning

This warning indicates that ArgoCD has encountered a resource that is not included as
part of its discovery phase. This resource `kind` and `apigroup` must be added to
the `resource.inclusions` list in [prod-vars.yaml](https://github.com/AICoE/aicoe-cd/blob/master/vars/prod-vars.yaml).

### SA Cannot list resource, no RBAC policy matched

An example error:

```bash
"system:serviceaccount:argocd-manager:argocd-manager" cannot list objectbucketclaims.objectbucket.io in the namespace "example-ns": no RBAC policy matched
```

This means that the service account being used by ArgoCD (e.g. `argocd-manager`)
does not have the appropriate permissions to list the resource in the designated
name space. Let the application owner know they need to give the SA more permissions.
