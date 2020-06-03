# Application Management

Currently application creation/managment happens via the ArgoCD UI. In the future
we will create and manage applications declaratively.

## Deploying Applications with ArgoCD

1. Log into ArgoCD ui at ```echo https://$(oc get routes | grep argocd-server | awk 'NR==1{print $2}')```
2. Click create application
3. Fill the fields out like so:

```text
Application name: <application_name>
Project: <your_team_project>
Sync Policy: Automatic (or manual)
Sync Option: Check Use A Schema...
Repository URL: <git_url>
Revision: <branch_name> or HEAD
Path: <path_to_overlay> or .
Cluster: Your cluster address
Namespace: <application_namespace>
```

Leave the rest blank and click create at the top left.
If everything works the creation slider will go away, refresh the page and the app should be there if isn't already. Click sync and your application should be deployed.

NOTE: You may need to disable schema validation if your deployments are failing. This is due to the ArgoCD api validator having a very strict api spec.
