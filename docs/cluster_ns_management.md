## Cluster & Namespace Management

We add [clusters declaratively](https://argoproj.github.io/argo-cd/operator-manual/declarative-setup/#clusters) in ArgoCD. 

The ansible playbook will auto generate the cluster spec that is defined within
the [prod-vars.yaml](https://github.com/AICoE/aicoe-cd/blob/master/vars/prod-vars.yaml).


## Deploying to a namespace
In order for ArgoCD to be able to deploy to your namespace you must allow ArgoCD access to your namespace. This can be done by doing the following: 

1. Give the `argocd-manager` service account (SA) a role with access to the namespace. 
2. Add the namespace to the cluster spec in ArgoCD 


### 1. Add role to SA 
ArgoCD uses a SA named `argocd-manager` to deploy resources to another cluster/namespace. These SAs need access to the resources it will be deploying, this is done via roles and rolebindings. 

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
> **`role`**: must be a project `admin` role, or a role that, _at the minumum_ has 
>read access to all resources listed in `resource.inclusions` in 
>[prod-vars.yaml](https://github.com/AICoE/aicoe-cd/blob/master/vars/prod-vars.yaml).


### 2. Add namespace to ArgoCD Cluster spec 

Namespaces are added to ArgoCD by altering the corresponding cluster spec. Cluster specs are defined within the [prod-vars.yaml](https://github.com/AICoE/aicoe-cd/blob/master/vars/prod-vars.yaml) under the `clusters` field. To add a namespace, find `your_cluster` and append the `namespaces` list with your namespace, see below for an example. 
 
```yaml

clusters:
  - name: your_cluster_spec_name.com
    data:
      name: your_cluster_context/service_account
      namespaces:
        - Namespace_1
        - Namespace_2
        - ...
        - <ADD_NEW_NAMESPACE_HERE> 
      server: your_cluster_url
      config:
        bearerToken: argocd_manager_sa_token 
        tlsClientConfig:
          insecure: true
```

Once done, submit a PR with this change. If you are merging such a PR, run the playbook afterwards to bring the update live. 

If your cluster isn't there then it will need to be added by a member of the aicoe-sre team. See below.

## Adding a new Cluster 

Note: Must be done by a member of the aicoe-sre team. 

ArgoCD will need a service account present on the cluster for deployments. Where the SA is located is irrelevant, though it's advised to have it be located in it's own indepenent namespace. For consistency name this service account `argocd-manager`. 


This workflow may look like this:
```
$ oc login <your_cluster> 
$ oc new-project argocd-manager
$ oc create sa argocd-manager
```

Get the token for this SA
```
$ SA_TOKEN=`oc sa get-token argocd-manager -n argocd-manager`
```
Encrypt the SA token using Ansible-Vault: 

```
SA_TOKEN_ENCRYPTED=`echo $SA_TOKEN | ansible-vault encrypt`
```


Get the server hostname: 
```
$ SERVER=`oc whoami --show-server`
```

Next you will need to append to the `clusters` field in [prod-vars.yaml](https://github.com/AICoE/aicoe-cd/blob/master/vars/prod-vars.yaml). 

See below for an example with comments on how to fill out the spec. 

```yaml

clusters:
  - ...
  # Add your cluster here
  - name: cluster-<cluster_hostname>    # Name of the cluster spec secret
    data:
  name: mycluster.com                   # Human readable name for your cluster
      namespaces:
        - <namespace_1> 
        - <namespace_2>
        - ...
      server: <cluster_hostname>        # Value of ${SERVER}
      config:
        bearerToken: <sa_token>         # Value of ${SA_TOKEN_ENCRYPTED}
        tlsClientConfig:
          insecure: true
```

Once done, submit a PR. After merge, run the playbook to bring the changes live. 


## Cluster Inclusions 

It is likely that your team does not have `get` access to all namespace scoped resources.
This can be an issue when deploying apps to a namespace in a cluster, because ArgoCD will
attempt to discover all namespace scoped resourced and be denied. To avoid this, we limit
ArgoCD to discover the resources that are available to project admins, these are added 
under the `resource.inclusions` ArgoCD configurations in
[prod-vars.yaml](https://github.com/AICoE/aicoe-cd/blob/master/vars/prod-vars.yaml).