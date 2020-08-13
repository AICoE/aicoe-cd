# Application Management

While ArgoCD allows you to create ArgoCD applications via the UI and CLI, it is our
policy that all applications be created [declaratively](https://argoproj.github.io/argo-cd/operator-manual/declarative-setup/#applications).

This allows us to easily restore your applications should the need arise.

## Pre-requisites
- Kustomize version 3.8+

## Steps for creating an application
For your application to show up to ArgoCD you need to do 2 things:
1. Create the Application yaml in the appropriate path in a fork
2. Submit a PR to this repository

These steps are outlined in detail below:

### Step 1. Create the Application Yaml

Clone the repo and `cd` into where applications are stored:

```bash

$ git clone https://github.com/${GIT_USERNAME}/aicoe-cd.git
$ cd aicoe-cd/manifests/overlays/prod/privileged_resources/applications
```

If your team folder does not exist, create it:

```bash
$ mkdir example_team && cd example_team
$ kustomize create
```

Let's create a sample application called `example-app`.

```yaml
# aicoe-cd/manifests/overlays/prod/privileged_resources/applications/example_team/example_app.yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example_app
spec:
  destination:
    namespace: example_namespace
    server: http://example_server
  project: example_project
  source:
    path: path/to/kustomization
    repoURL: http://path-to-example-repo
    targetRevision: HEAD
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - Validate=false
```
This is a basic minimal example application. For additional fields see [here](https://argoproj.github.io/argo-cd/operator-manual/application.yaml).

Let's go over what some of the fields in the `example_app.yaml` refer to:

- `metadata.name`: the name of the `Application` resource as well as the name as it appears on the ui.

- `spec.project`: the ArgoCD `Project` to which this `Application` belongs, ensure that this `Project` exists in [projects](../manifests/overlays/prod/projects).

- `spec.destination.namespace`: the target namespace for this `Application`'s deployment, ensure that this namespace exists in [clusters](../manifests/overlays/prod/secrets/clusters) for the appropriate cluster/server.

- `spec.destination.cluster`: the target cluster server name for this `Application`'s deployment, ensure this server exists in  [clusters](../manifests/overlays/prod/secrets/clusters) for the appropriate cluster.

- `spec.source.path`: path to the Kustomization.yaml file relative to the repo's root.

- `spec.source.repoURL`: the repository holding the `Application`'s desired state, ensure this repo exists within `argo_cm` [here](../manifests/overlays/prod/configs/argo_cm).

  In case you are adding a private repo:

  1. Please ensure [@sesheta](https://github.com/sesheta) bot has access to your repo (directly or indirectly via [@AICoE/sourceops](https://github.com/orgs/AICoE/teams/sourceops)/[@aicoe-aiops/sourceops](https://github.com/orgs/aicoe-aiops/teams/sourceops) teams)
  2. Reference repository access credentials via SourceOps secret:

      ```yaml
      repositories: |
      ...
      - type: git
        url: https://github.com/<PATH>.git
        usernameSecret:
          key: username
          secret: srcops
        passwordSecret:
          key: token
          secret: srcops
      ...
      ```

> NOTE: You may need to disable schema validation if your deployments are failing. This is due to the ArgoCD api validator having a very strict api spec.

Once you are done add the application yaml to the Kustomization file by running the following _from the repository root directory_.

```
$ cd aicoe-cd/manifests/overlays/prod/privileged_resources/applications/example_team
$ kustomize edit add resource example_app.yaml
```


### Step 2. Make a Pull request

Commit your changes and submit a pr to `aicoe-sre` repository. An AICOE-SRE team
member will review your PR.


#### If your application exists in ArgoCD but not on VCS
We make no guarantees about your application if it does not exist on vcs. Our
policy is "_if it's not on vcs, it does not exist_". Luckily, getting the manifests
for all your applications is easy to do with the argocd cli.

```bash
# Login via cli using sso
$ argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --sso

# Get the application resource details
$ argocd app get ${APP_NAME} -o yaml
```
