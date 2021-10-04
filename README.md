# Control-M Agent Helm Operator

For more information please refer to
 https://sdk.operatorframework.io/docs/building-operators/helm/tutorial/


## Build Process

### Scaffolding the Operator

Checkout the existing Helm chart present in
 https://github.com/controlm/automation-api-community-solutions/pull/105/files

And then execute

```
operator-sdk init \
    --plugins helm \
    --domain bmc.com \
    --group controlm \
    --version v1alpha1 \
    --kind Agent \
    ----helm-chart ../automation-api-community-solutions/3-infrastructure-as-code-examples/kubernetes-agent-using-helm/control-m-agent
```

The result is an initial repository similar to this one.

### RBAC Permissions

Take care to match the roles and permissions under ```config/rbac ``` with the contents of the Helm chart.

Since the chart creates a role, a statefulset, a pvc and a secret - all are required to be given as permissions to the roles of the Operator.

## Executing installation for a local test 

Deploy the Operator's Controller Manager ```make deploy ```

And then

```kubectl apply -f ``` the example CR ```config/samples/controlm_v1alpha1_agent.yaml ```

The Controller Manager will reconcile installation of the Helm Chart, and create/update the Agent.
