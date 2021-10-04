# Control-M Agent Helm Operator

## What is a Helm Operator?

A Helm Operator is used to deploy a Helm Chart based on a CRD.
In our case we will create an Operator that installs the new Helm Chart of BMC Control-M Agent.

For more information please refer to
 https://sdk.operatorframework.io/docs/building-operators/helm/tutorial/


## How this was built

First checkout the existing Helm chart present in
 https://github.com/controlm/automation-api-community-solutions/pull/105/files

And then executed

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

Next, verified to match the roles and permissions under ```config/rbac/role.yaml ``` with the contents of the Helm chart.

Since the chart creates a role, a statefulset, a pvc and a secret - all are required to be given as permissions to the roles of the Operator.

## Executing installation for a local test 

Deploy the Operator's Controller Manager ```make deploy ```

And then

```kubectl apply -f ``` the example CR ```config/samples/controlm_v1alpha1_agent.yaml ```

The Controller Manager will reconcile installation of the Helm Chart, and create/update Agents.

## Cleaning up

Remove the installed agent by ```kubectl delete -f config/samples/controlm_v1alpha1_agent.yaml ```

Run ```make undeploy ``` to remove the CRD and the Operator.

## Bundle and OperatorHub integration

To be implemented soon.

## Next Steps

1. Create an official, certified image of the Agent by moving dockerFile and Entrypoints from the above PR into a CI inside BMC and and official image repo.
2. Bundle this repo with OLM, move to an official CI inside BMC, and certify.
3. Finally, upload to OperatorHub.