# Insights Proxy ACM Addon (for disconnected clusters)

You can find some prior work at [PoC.md](PoC.md).

## Install the addon

1. Have access to a hub cluster
2. Have access to a managed cluster
3. The managed cluster can be disconnected from the internet, but it must have access to the Insights Proxy load balancer that will be created by the addon.
4. The managed cluster must have the label `should-use-insights-proxy` set to `true`, configured when importing the cluster to the hub cluster.
5. The managed cluster must have Insights enabled. Otherwise you can run something like
```shell
oc patch clusterversion/version --type merge -p '{"spec":{"capabilities":{"additionalEnabledCapabilities":["openshift-samples","marketplace","Console","MachineAPI","ImageRegistry","DeploymentConfig","Build","OperatorLifecycleManager","Ingress", "Insights"]}}}'
```
Be careful because this will reset the capabilities to the ones specified in the patch.
6. Enable the addons to control policies on the managed cluster:

First, export the managed cluster name:
```shell
export YOUR_MANAGED_CLUSTER_NAME=$(oc get managedcluster -o jsonpath='{.items[0].metadata.name}')
```
And then enable the addons:
```shell
# Enable governance policy framework
oc create -f - <<EOF
apiVersion: addon.open-cluster-management.io/v1alpha1
kind: ManagedClusterAddOn
metadata:
  name: governance-policy-framework
  namespace: $YOUR_MANAGED_CLUSTER_NAME
spec: {}
EOF

# Enable config policy controller
oc create -f - <<EOF
apiVersion: addon.open-cluster-management.io/v1alpha1
kind: ManagedClusterAddOn
metadata:
  name: config-policy-controller
  namespace: $YOUR_MANAGED_CLUSTER_NAME
spec: {}
EOF
```
7. Log into the hub cluster and run the following commands:
```shell
oc import-image insights-proxy/insights-proxy-container-rhel9:1.5.3 --from=registry.redhat.io/insights-proxy/insights-proxy-container-rhel9:1.5.3 --confirm
oc create namespace insights-proxy
oc apply -f deploy # TODO: I did apply, but I need to try with create
```
8. Ensure the Insights Proxy URL in [insights-proxy-url-updater.yaml](deploy/insights-proxy-url-updater.yaml) variable was correctly used in the ConfigMap: access the managed cluster and check the `insights-config` ConfigMap in the `openshift-insights` namespace.
9.  Check the Insights Operator logs in the managed cluster to see if the configuration `insights-config` was created successfully in the `openshift-insights` namespace. Then check the `insights-operator` pod logs to see if the configuration was applied. Otherwise, delete the pod and force a configuration reload (although it should be automatic).

## Uninstall the addon

```shell
cd deploy
oc delete -f .
```
