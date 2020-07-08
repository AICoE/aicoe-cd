## Related Issues

Link related issues here.

## This introduces a breaking change

- [ ] Yes
- [ ] No

## This Pull Request implements

Explain your changes.

## Description

Describe your changes in detail.

## Checklist for migrating an Application to ArgoCD

When migrating an application's deployment to be managed by ArgoCD use the following checklist to verify your process.

Items to complete before making a migration PR
- [ ] Ensure your application manifests can be built using Kustomize.
- [ ] If using secrets, make sure to include the .sops.yaml file in your repository.
- [ ] Make sure to create the role granting access to namespace. See [here](cluster_ns_management.md#deploying-to-a-namespace) for more info. This role is tracked in your application manifests.

Items to include within the migration PR
- [ ] Ensure your namespace exists in the cluster spec in the [prod-vars.yaml](../vars/prod-vars.yaml) file (for the appropriate cluster).
- [ ] If you are switching between ArgoCD managed namespaces, and that namespace was deleted in OCP, then ensure it is also removed from [prod-vars.yaml](../vars/prod-vars.yaml).
- [ ] Ensure the application repository is added in the `argo_cm` file in [prod-vars.yaml](../vars/prod-vars.yaml).
- [ ] Ensure that all OCP resources that will be managed by ArgoCD on this cluster are included in the `inclusions` list in [prod-vars.yaml](../vars/prod-vars.yaml). See [here](cluster_ns_management.md#cluster-inclusions) for more info.
- [ ] Create the ArgoCD Application, see [here](application_management.md) for instructions.
