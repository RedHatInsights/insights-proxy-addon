# Insights Proxy ACM Addon (for disconnected clusters)

You can find some prior work at [PoC.md](PoC.md).

## Install the addon

1. Have access to a hub cluster
2. Have access to a managed cluster
3. The managed cluster can be disconnected from the internet, but it must have access to the Insights Proxy load balancer that will be created by the addon.
4. The managed cluster must have the label `should-use-insights-proxy` set to `true`, configured when importing the cluster to the hub cluster.
5. Log into the hub cluster and run the following commands:
```shell
oc import-image insights-proxy/insights-proxy-container-rhel9:1.5.3 --from=registry.redhat.io/insights-proxy/insights-proxy-container-rhel9:1.5.3 --confirm
find deploy/ -name "*.yaml" ! -name "disconnected-insights-addon-template.yaml" -exec oc apply -f {} \;
```
(we need to exclude the disconnected-insights-addon-template.yaml because we first need to know the Insights Proxy load balancer URL)
6. Get the Insights Proxy load balancer address:
```shell
oc get svc insights-proxy-service -n insights-proxy
```
You can check the URL is working with
```shell
curl --proxy http://{{URL}}:80 https://console.redhat.com/api/insights-results-aggregator/v2/info
```
7. Set the `INSIGHTS_PROXY_URL` in [disconnected-insights-addon-template.yaml](deploy/disconnected-insights-addon-template.yaml) variable to the Insights Proxy load balancer IP address.
8. Run the following command to deploy the disconnected insights addon:
```shell
oc apply -f deploy/disconnected-insights-addon-template.yaml
```
9. Check the Insights Operator logs in the managed cluster to see if the configuration `insights-config` was created successfully in the `openshift-insights` namespace. Then check the `insights-operator` pod logs to see if the configuration was applied. Otherwise, delete the pod and force a configuration reload (although it should be automatic).

## Uninstall the addon

```shell
cd deploy
oc delete -f .
```
