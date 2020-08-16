# Give ArgoCD access to your project

ArgoCD uses an SA named `argocd-manager` to deploy resources to another cluster/namespace. These SAs need access to the resources it will be deploying, this is done via roles and rolebindings.

In your namespace, you will need to deploy a rolebinding like the one below:

```yaml
apiVersion: authorization.openshift.io/v1
kind: RoleBinding
metadata:
  name: argocd-manager-rolebinding
  namespace: <application_namespace>
roleRef:
  name: <role>
subjects:
  - kind: ServiceAccount
    name: argocd-manager
    namespace: <sa_namespace>
```

Fill out `application_namespace`, `role`, and `sa_namespace`.

> **`application_namespace`**: This is your project namespace.
>
> **`sa_namespace`**: On PSI OCP4 this will be `aicoe-argocd` and on Data-Hub clusters it will be `argocd-manager`.
>
> **`role`**: must be a project `admin` role.
