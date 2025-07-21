# Insights Proxy ACM Addon (for disconnected clusters)

You can find some prior work at [PoC.md](PoC.md).

## Install the addon

```shell
cd deploy
oc import-image insights-proxy/insights-proxy-container-rhel9:1.5.3 --from=registry.redhat.io/insights-proxy/insights-proxy-container-rhel9:1.5.3 --confirm
oc apply -f .
```
