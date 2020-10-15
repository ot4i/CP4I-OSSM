# Cloud Pak for Integration and Red Hat OpenShift Service Mesh

[//]: 0 "Add index"

## Introduction

In this repo we provide instructions on how to set up the Red Hat OpenShift Service Mesh - based on Istio - on an OpenShfit cluster running the Cloud Pak for Integration.

We also describe 4 test use cases to use ACE together with the OpenShift Service Mesh, providing details on how ACE can be enabled to work together with Istio.

It is worth nothing that:
- An installation of Red Hat OpenShift Service Mesh differs from upstream Istio community installations in multiple ways: https://docs.openshift.com/container-platform/4.3/service_mesh/service_mesh_arch/ossm-vs-community.html#ossm-vs-community
- OCP uses an opinionated version of Istio called Maistra: https://maistra.io/

## Cluster set-up

Provision Cloud Pak for Integration 2020.3 cluster on OpenShift Container Platform 4.4 or 4.5: https://www.ibm.com/support/knowledgecenter/SSGT7J_20.3/install/sysreqs.html.

***Note:*** The instructions in this repo specifically refer to these versions of the CP4I and OCP.

## Service Mesh set-up

### Service Mesh Installation
- Refer to instructions at: https://docs.openshift.com/container-platform/4.4/service_mesh/service_mesh_install/installing-ossm.html
- Versions if on OpenShfit 4.4:
  - Red Hat Service Mesh 1.1.9
  - Istio 1.4.8
  - Jaeger 1.17.6
  - Kiali 1.12.15
  - Elasticsearch 4.4.x
- Versions if on OpenShfit 4.5:
  - Red Hat Service Mesh 1.1.9
  - Istio 1.4.8
  - Jaeger 1.17.6
  - Kiali 1.12.15
  - Elasticsearch 4.5.x
- Starting with Red Hat OpenShift Service Mesh 1.1.3, you must install the Elasticsearch Operator, the Jaeger Operator, and the Kiali Operator before the Red Hat OpenShift Service Mesh Operator can install the control plane.
- Install operators from OperatorHub in the following order
  - Install Elasticsearch (select the Update Channel that matches your OpenShift Container Platform installation)
    - Select the `A specific namespace on the cluster` option and then select `openshift-operators-redhat` from the menu
  - Install the Red Hat version of the Jaeger operator (not community)
    - Select `1.17-stable`
    - Select *All namespaces on the cluster*, which will install the Operator in the `openshift-operators` project
  - Install the Red Hat version of Kiali (not community)
    - Select `stable`
    - Select *All namespaces on the cluster*, which will install the Operator in the `openshift-operators` project
  - Install Red Hat OpenShift Service Mesh
    - Select `stable`
    - Select *All namespaces on the cluster*, which will install the Operator in the `openshift-operators` project

### Service Mesh - Control Plane deployment
- Create a project named `istio-system`
- Navigate to *Installed Operators* in the `istio-system` project, and make sure that all the cluster-wide operators are available and in *Succeeded* state
- Create *ServiceMeshControlPlane* in the `istio-system` namespace, from the Installed Operators tab, and using the `all-in-one` yaml template without customisation: `cp4i-istio-controplane`
- Create a *ServiceMeshMemberRoll* called `default`, with a sample empty project (e.g. `istio-test`) added to the members:
```
apiVersion: maistra.io/v1
kind: ServiceMeshMemberRoll
metadata:
  name: default
  namespace: istio-system
spec:
  members:
    - istio-test
```

Once the Service mesh is successfully deployed, it generates by default network policies which prevent accessing pods via the OpenShift Routes, unless specifically exposed via pre-existing Network polices (which is the case for the ACE Dashboard and ACE Designer).

### Service Mesh - Control Plane configuration
#### Mixer policy enforcement
- Change mixer policy enforcement from `disablePolicyChecks: true`  to `disablePolicyChecks: false`.
#### Automatic Route Creation
The OpenShift Service Mesh provides a feature called *Automatic Route Creation*. If this feature is enabled, every time an Istio Gateway is created, updated or deleted inside the service mesh, an OpenShift route is created, updated or deleted.

To leverage this feature:
- Enable Automatic Route Creation (IOR)
  - In the Service Mesh Control plane, change from `ior_enabled:false` to `ior_enabled:true`.
- For full instructions refer to: https://docs.openshift.com/container-platform/4.4/service_mesh/service_mesh_day_two/ossm-auto-route.html  
#### Sidecar Injection
With the OpenShift Service Mesh, for Envoy proxy sidecars to be automatically injected into pods at deployment time, each Kubernetes deployment needs to have a special annotation: the annotation `sidecar.istio.io/inject: 'true'` needs to be present in the deployment spec's template metadata.
- For deployments created from a YAML file or a Helm package, this annotation can be manually added to the spec.
- For deployments created by an Operator (like all CP4I runtimes), this annotation needs to be present in the Operator spec.


To test that sidecar injection is working:
- Create a generic test deployment via YAML.
- To enable sidecar injection, add line: `sidecar.istio.io/inject: 'true'` to the test deployment in **spec > template > metadata > annotations**
- Test that Istio injection works for selected deployments: a sidecar container called `istio-proxy` is added to the pod
- For full instructions refer to: https://docs.openshift.com/container-platform/4.4/service_mesh/service_mesh_day_two/prepare-to-deploy-applications-ossm.html

## Applications

### Bookinfo
A more articulate example is given by the Bookinfo application. In this example, deployments are  created via YAML files.
- Refer to: https://github.com/ot4i/CP4I-OSSM/tree/master/bookinfo

### ACE Server
In this example we will examine how to use AppConnect Enterprise together with the OpenShift Service Mesh.
- Create a dedicated OpenShift project: e.g. `ace-istio`.
- Deploy the AppConnect Dashboard to this project.
- Add this project to the ServiceMeshMemberRoll in the OSSM operator instance: this will create two new network policies in the `ace-istio` project to prevent access to anything in that namespace:
![Istio network policies](https://github.com/ot4i/CP4I-OSSM/blob/dev/images/istio-netpols.png)

***Note:*** existing Network policies will take precedence on the newly created ones: e.g. the ACE Dashboard network policies.

ACE servers, like other ACE runtimes, are deployed via Kubernetes Operators. In Cloud Pak for Integration 2020.3.1, it is possible to add custom annotations to the operators at deployment time, like the one needed to enable sidecar injection: https://www.ibm.com/support/knowledgecenter/SSTTDS_11.0.0/com.ibm.ace.icp.doc/certc_install_integrationserveroperandreference.html#crvalues.

In this example we will use this feature via the ACE Dashboard User Interface.

This repo provides 4 ACE server examples to test ACE + Istio functionality:
1. `toolkit-no-tracing`: deployment with no OD and Designer sidecars
2. `toolkit-tracing`: deployment with OD sidecars, but no Designer sidecars - includes A/B test
3. `designerflows-no-tracing`: deployment with Designer sidecars, but no OD sidecars
4. `designerflows-tracing`: deployment with both OD and Designer sidecars - includes A/B test

Test cases 2 and 4 also implement A/B testing via the Istio Service Mesh.

Detailed instructions for each test case are available here: https://github.com/ot4i/CP4I-OSSM/tree/dev/ace.

If all 4 configurations are deployed, the Kiali dashboard will display them similarly to the picture below:

![complete-configuration](https://github.com/ot4i/CP4I-OSSM/blob/dev/images/complete-configuration-kiali.png)

## Remove the OpenShift Service Mesh
To uninstall the OpenShift Service Mesh, use the script available here: https://github.com/ot4i/CP4I-OSSM/blob/master/remove-service-mesh/remove-mesh-script.sh
