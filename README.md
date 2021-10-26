# Control-M Agent Helm Operator (Draft)

## What is a Helm Operator?

A Helm Operator is used to deploy a Helm Chart based on a CRD.
In our case we will create an Operator that installs the new Helm Chart of BMC Control-M Agent.

For more information please refer to
 https://sdk.operatorframework.io/docs/building-operators/helm/tutorial/

## Folder description

```
agent-docker-manifests --> container manifests (dockerfile and entrypoint) used to build the UBI-based Agent.
bundle --> Operator Bundle resources
config --> SDK-generated configurations.  also contains a running Custom Resource for testing, under /samples.
helm-charts --> the up to date chart.  This is newer than the one inside the "automation community" repo. contains a fix for SCC on Openshift.  
```

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

## Building Images

### The ControlM Agent Image (Application)

```
cd agent-docker-manifests
docker build \
       --build-arg AAPI_END_POINT=<API ENDPOINT>/automation-api \
       --build-arg AAPI_USER=<CTRLM_USER> \
       --build-arg AAPI_PASS=<CTRLM_PASS> \
       --build-arg AGENT_IMAGE_NAME=Agent_Image.Linux \
       --build-arg SUB_USER=<RH_SUBSCRIPTION_USER> \ 
       --build-arg SUB_PWD=<RH_SUBSCRIPTION_PASS> \
       . -t <bmc-repository>/controlmagent:<tag>
cd ..
```

### The Helm Operator Image

Inspect `IMG` inside `Makefile` and then run `make docker-build && make docker-push`

### The Bundle Image

See `Bundle and OperatorHub integration` below

## Executing installation for a local test 

Deploy the Operator's Controller Manager ```make deploy ```

And then

```kubectl apply -f ``` the example CR ```config/samples/controlm_v1alpha1_agent.yaml ```

The Controller Manager will reconcile installation of the Helm Chart, and create/update Agents.

## Cleaning up

Remove the installed agent by ```oc delete -f config/samples/controlm_v1alpha1_agent.yaml ```

Run ```make undeploy ``` to remove the CRD and the Operator.

## Bundle and OperatorHub integration

The Operator can be installed to a cluster with OLM integration,  using the following command -
```operator-sdk run bundle quay.io/itroyano/controlmagent-helm-bundle:0.0.2```

and then 

```oc apply -f ``` the example CR ```config/samples/controlm_v1alpha1_agent.yaml ```

The Controller Manager will reconcile installation of the Helm Chart, and create/update Agents.

### How was this created?
Check the makefile for the following targets

```make bundle```

And then

```make bundle-build```

```make bundle-push ```

## Next Steps

1. Create an official, certified image of the Agent by moving dockerFile and Entrypoints from the above PR into a CI inside BMC and and official image repo.
2. Bundle this repo with OLM, move to an official CI inside BMC, and certify.
3. Finally, upload to OperatorHub.
