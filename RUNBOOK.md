# Production Runbook for ArgoCD

## Key Locations

Production Namespace: aicoe-argocd
Production Cluster: This information can be found in the internal Runbooks repository.

## Important Metrics

We have a number of important metrics we are tracking on our operations dashboard.
Information on these metrics can be found at [Important ArgoCD Metrics](docs/argocd_metrics.md).

## Common Debugging Steps

### Application is stuck will not sync and/or is stuck on refresh

Check work queue metric, it is possible that thread count is bottlenecked,
restart the application controller pod.

### Excluded resource warning

This warning indicates that ArgoCD has encountered a resource that is not included as
part of its discovery phase. This resource `kind` and `apigroup` must be added to
the `resource.inclusions` list in [prod-vars.yaml](https://github.com/AICoE/aicoe-cd/blob/main/vars/prod-vars.yaml).

### SA Cannot list resource, no RBAC policy matched

An example error:

```bash
"system:serviceaccount:argocd-manager:argocd-manager" cannot list objectbucketclaims.objectbucket.io in the namespace "example-ns": no RBAC policy matched
```

This error can occur in one of two ways:

- The resource is a `CRD` and was removed from the cluster but exists in the [inclusions list](manifests/overlays/prod/configs/argo_cm/resource.inclusions)
for that cluster.

    Verify this by running `$ kubectl api-resources` and seeing if the APIGroup and Resource shows up.

    **Solution**:
    Remove the cluster from this APIGroup and Resource from the [inclusions list](manifests/overlays/prod/configs/argo_cm/resource.inclusions).
    See [Example PR here](https://github.com/AICoE/aicoe-cd/pull/173#event-3979099664).


- The `serviceaccount` ArgoCD uses to access this cluster does not have read access on this APIGroup and Kind.

    **Solution**:
    the service account being used by ArgoCD (e.g. `argocd-manager`) does not have the appropriate permissions
    to list the resource in the designated name space. Let the application owner know they need to give the SA more permissions.

    If the SA is a project admin that needs permissions to this resource, you can try using [aggregated clusterroles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#aggregated-clusterroles)
    to aggregate permissions for this resource to project admin.
