# Important Metrics

We are tracking a number of metrics at our operations dashboard. What follows are the distinct categories of metrics

## Service Level Indicators

SLIs are carefully defined quantitative measures of some aspect of the level of service that is provided. You can find more information on SLIs in the [Site Reliability Engineering Book](https://landing.google.com/sre/sre-book/chapters/service-level-objectives).

| SLI                                    | Query                                                                                                     | Status                        |
|----------------------------------------|-----------------------------------------------------------------------------------------------------------|-------------------------------|
| Availability                           | min(up{job="ArgoCD Metrics"})                                                                             | Done                          |
| Reconciliation Performance             | sum(increase(argocd_app_reconcile_bucket{dest_server=\~"$Server", job="ArgoCD Metrics"}[10m])) by (le)    | Needs Update to use Histogram |
| Length of Work Queue                   | increase(workqueue_unfinished_work_seconds{}[10m])                                                        | Done                          |

### SLI Descriptions

1. **Availability:** This tells us whether important ArgoCD pods are up.
2. **Reconciliation Performance:** For ArgoCD, Reconciliation refers to making the state of the application match what is stored in git. This is usually done by managing Kubernetes objects. This SLI lets us see how much time our reconciliations are taking, and whether the system is performing in a degraded state/ needs to be scaled up.
3. **Length of Work Queue:** This tells how many seconds of work are in progress but have not been observed by work_duration. Large values usually indicate stuck threads.

## Usage Metrics

These metrics are being used to track usage of the ArgoCD instance

| Usage Metric              | Query                                                                 | Status   |
|---------------------------|-----------------------------------------------------------------------|----------|
| Number of Managed Servers | count(count by (dest_server) (argocd_app_info{project=\~"$Project"})) | Done     |
| Number of Managed Apps    | count((argocd_app_info{project=\~"$Project"}))                        | Done     |
| Number of Source Repos    | count(count by (repo) (argocd_app_info{project=\~"$Project"}))        | Done     |
| Rate of Change            | To Be Implemented                                                     | Not Done |

## Service Level Indicators for App Owners

These metrics are possible SLIs for the App Owners

| SLI               | Query                                                                                                               | Status |
|-------------------|---------------------------------------------------------------------------------------------------------------------|--------|
| App Sync Failures | ceil(increase(argocd_app_sync_total{dest_server=\~"\$Server", project=\~"$Project", phase=\~"Error\|Failed"}[10m])) | Done   |

## Possible Service Level Indicators

These metrics are interesting to look at but their usefulness has not yet been determined.

| Metric                                 | Query                                                                                                                                                                                                                             | Possible Usage | Status                  |
|----------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------|-------------------------|
| Percentage of Apps in Sync             | sum(argocd_app_info{sync_status="Synced", dest_server=\~"\$Server", dest_namespace=\~"\$Namespace", project=\~"\$Project"})/sum(argocd_app_info{dest_server=\~"\$Server", dest_namespace=\~"\$Namespace", project=\~"\$Project"}) | App Owner SLI  | Done                    |
| Cumulative GRPC Success Rate           | sum(grpc_server_handled_total{grpc_code="OK",job="ArgoCD Metrics"})/sum(grpc_server_handled_total{job="ArgoCD Metrics"})                                                                                                          | SRE SLI        | Done                    |
| Application Time Spent out of Sync     | To be Implemented                                                                                                                                                                                                                 | App Owner SLI  | Not Done                |
| App Reconciliation Work Duration       | sum(increase(workqueue_work_duration_seconds_bucket{name="app_reconciliation_queue"}[10m])) by (le)                                                                                                                               | SRE SLI        | Update to use Histogram |
| App Operation Processing Work Duration | sum(increase(workqueue_work_duration_seconds_bucket{name="app_operation_processing_queue"}[10m])) be (le)                                                                                                                         | SRE SLI        | Update to use Histogram |

### Metric Descriptions

1. **Percentage of Apps in Sync:** Having a large percentage of apps be out of sync for a long duration indicates that there are some issues going on. Since the issues can be caused by both app-owners and the SREs it doesn't make sense to have this as an SRE SLI.
2. **Cumulative GRPC Success Rate:** Seeing a large drop in the success rate could indicate issues with the underlying system.
3. **Application Time Spent out of Sync:** Similar to *1*, this can have multiple causes so it is difficult to define an SRE SLI based on this.
4. **App Reconciliation Work Duration:** This gives us a histogram of how much time (in seconds) it takes for ArgoCD to process items in the App Reconciliation queue.
5. **App Operation Processing Work Duration:** This gives us a histogram of how much time (in seconds) it takes for ArgoCD to process items in the App Processing queue.
