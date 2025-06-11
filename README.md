# Insights Proxy

```
oc patch clusterversion/version --type merge -p '{"spec":{"capabilities":{"additionalEnabledCapabilities":["openshift-samples","marketplace","Console","MachineAPI","ImageRegistry","DeploymentConfig","Build","OperatorLifecycleManager","Ingress", "Insights"]}}}'

oc import-image insights-proxy/insights-proxy-container-rhel9:1.5.3 --from=registry.redhat.io/insights-proxy/insights-proxy-container-rhel9:1.5.3 --confirm

oc apply insights-proxy.yaml
```

You can now try the Insights Proxy:

```
[root@curlpod /]# curl --proxy http://insights-proxy-service.openshift-insights.svc:3128 https://console.redhat.com/api/insights-results-aggregator/v2/reports
{"errors":[{"meta":{"response_by":"gateway"},"status":401,"detail":"Missing Authentication"}]}
```