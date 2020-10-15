# Cloud Pak for Integration and Red Hat OpenShift Service Mesh

[//]: 0 "Add index"

[//]: 1 "Need more context: the top level CP4I-OSSM readme needs to better explain what it is, why it’s there for people who come to it without reading this post. "

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
- Navigate to `Installed Operators` in the `istio-system` project, and make sure that all the cluster-wide operators are available and in `Succeeded` state
- Create `ServiceMeshControlPlane` in the `istio-system` namespace, from the Installed Operators tab, and using the `all-in-one` yaml template without customisation: `cp4i-istio-controplane`
- Create a `ServiceMeshMemberRoll` called `default`, with a sample empty project (e.g. `istio-test`) added to the members:
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
#### Automatic Route Creation
[//]: 5 "Explain what IOR is."
- Enable automatic route creation (IOR)
  - In the Service Mesh Control plane change `ior_enabled` from `false` to `true`:
  ```
  ior_enabled: true
  ```
- For full instructions refer to: https://docs.openshift.com/container-platform/4.4/service_mesh/service_mesh_day_two/ossm-auto-route.html  
#### Sidecar Injection
[//]: 2 "Explain that to enable sidecar injection a particular annotation needs to be added to the deployment. This is done differently if the deployment is managed by an operator or not."
- To enable sidecar injection, add line: `sidecar.istio.io/inject: 'true'` to a test deployment in **spec > template > metadata > annotations**
- Change mixer policy enforcement from `disablePolicyChecks: true`  to `disablePolicyChecks: false`.
- Test that Istio injection works for selected deployments
- For full instructions refer to: https://docs.openshift.com/container-platform/4.4/service_mesh/service_mesh_day_two/prepare-to-deploy-applications-ossm.html

## Applications

### Bookinfo
[//]: 3 "Say something about this example, and the fact that we're going to annotate the deployment directly."
- Refer to: https://github.com/ClaudioTag/CP4I-OSSM/tree/master/bookinfo

### ACE Server
- Add the namespace where you intend to deploy ACE (e.g. `ace-istio`) to the ServiceMeshMemberRoll, which creates two new network policies which prevent access to anything in that namespace:
![Istio netowrk policies](https://github.com/ClaudioTag/CP4I-OSSM/blob/master/images/Istio-network-policies.png)

Note that existing Network policies will take precedence on the newly created ones.

[//]: 4 "Say something about editing the operator's annotations (https://www.ibm.com/support/knowledgecenter/SSTTDS_11.0.0/com.ibm.ace.icp.doc/certc_install_integrationserveroperandreference.html#crvalues), and how we're going to do it"
- This repo provides 4 ACE server examples to test ACE + Istio functionality:
  1. `toolkit-no-tracing`: deployment with no OD and Designer sidecars
  2. `toolkit-tracing`: deployment with OD sidecars, but no Designer sidecars - includes A/B test
  3. `designerflows-no-tracing`: deployment with Designer sidecars, but no OD sidecars
  4. `designerflows-tracing`: deployment with both OD and Designer sidecars - includes A/B test
- Detailed instructions for each test case are available here: https://github.ibm.com/claudio-tag/Istio-PoC/tree/master/ace
- Test cases 2 and 4 also implement A/B testing via the Istio Service Mesh.

If all 4 configurations are deployed, the Kiali dashboard will display them similarly to the picture below:

![complete-configuration](https://github.com/ot4i/CP4I-OSSM/blob/dev/images/complete-configuration-kiali.png)

## Remove the OpenShift Service Mesh
To uninstall the OpenShift Service Mesh, use the script available here: https://github.com/ot4i/CP4I-OSSM/blob/master/remove-service-mesh/remove-mesh-script.sh
