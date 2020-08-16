# ArgoCD Permissions

Currently there are 3 OpenShift groups (same as ldap) with access:

- `data-hub-openshift-admins`
- `aicoe-thoth-devops`
- `aicoe-aiops-devops`

There are 3 ArgoCD roles with defined policies:

- `thoth-admin`
- `data-hub-admin`
- `aiops-admin`

OpenShift groups have the following ArgoCD role associations:

| OC Group                    | ArgoCD Role      |
|:--------------------------- |:---------------- |
| `aicoe-thoth-devops`        | `thoth-admin`    |
| `data-hub-openshift-admins` | `data-hub-admin` |
| `aicoe-aiops-devops`        | `aiops-admin`    |

ArgoCD roles have policies that define their access to ArgoCD resources, they are defined in [policy.csv](../manifests/overlays/prod/configs/argo_rbac_cm/policy.csv).

Read about how these policies and roles work [here](https://argoproj.github.io/argo-cd/operator-manual/rbac/).

In summary this is what you can and cannot do as a `<project>-admin`:

- get/create/update argocd clusters, certs, repos, projects, accounts
- cannot delete argocd clusters, certs, repos, projects, accounts
- this means `<project>-admin` can do things like `argocd cluster add ...` or `argocd proj list ...`
- `<project>-admin` can do pretty much anything including delete argocd application in their ArgoCD `<project>`
- `<project>-admin` cannot delete non application resources

All this applies to OpenShift Authentication, ArgoCD admin account retains unrestricted access.
