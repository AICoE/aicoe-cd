# Get ArgoCD to manage your Application

When migrating an application's deployment to be managed by ArgoCD use the following checklist to verify your process.

- Ensure your application manifests can be built using Kustomize.
- If using secrets, make sure to include the .sops.yaml file in your repository.
    - See [here](manage_your_app_secrets.md) for more info.
- Create the role granting access to namespace.
    - See [here](give_argocd_access_to_your_project.md) for more info.
    - This role should be tracked in your application manifest repository.

The following items require a PR to the `aicoe-cd` repository, and require an `aicoe-sre` team member to merge/deploy changes:

- Ensure the application repository is added in the `repository` file in [repositories](https://github.com/AICoE/aicoe-cd/blob/master/manifests/overlays/prod/configs/argo_cm/repositories).
- Ensure that all OCP resources that will be managed by ArgoCD on this cluster are included in the `inclusions` list in [resource.inclusions](https://github.com/AICoE/aicoe-cd/blob/master/manifests/overlays/prod/configs/argo_cm/resource.inclusions).
    - See [here](inclusions_explained.md) for more info.
- Create the ArgoCD Application manifest
    - See [here](create_argocd_application_manifest.md) for more info.

The following require an `aicoe-sre` team member to make the changes:

- Ensure your namespace exists in your cluster's spec see [here](admin/add_new_cluster_spec.md) for details.
- If you are switching between ArgoCD managed namespaces, and that namespace was deleted in OCP, then ensure it's also removed from your cluster's credentials found here [clusters](https://github.com/AICoE/aicoe-cd/tree/master/manifests/overlays/prod/secrets/clusters).
