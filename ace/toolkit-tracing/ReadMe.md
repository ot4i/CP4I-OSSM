# ACE app test: toolkit-tracing
This example deploys an ACE application designed via the ACE toolkit, with Operational Dashboard sidecars, but no designer flow sidecars.

This example also includes A/B testing instructions.


## Sidecar injection
To enable Istio sidecar injection, at deployment time you can add a custom annotation to the ACE operator.
- Create a new *Toolkit Integration* service from the ACE Dashboard
- Use the *Server Ping* test service: https://github.com/ot4i/CP4I-OSSM/blob/main/ace/testAPIs/serverPing.bar
- Enable Operations Dashboard tracing
- Enable *Advanced Settings*
- Add an *Advanced: Annotation*
  - operand_create_name: `sidecar.istio.io/inject`
  - operand_create_value: `true`

![toolkit-tracing-annotation](https://github.com/ot4i/CP4I-OSSM/blob/main/images/toolkit-tracing-annotation.png)


This will add the annotation `sidecar.istio.io/inject: 'true'` to the ACE deployment metadata, which in turn will allow for envoy sidecar injection.

## Remove ACE network policy
The ACE operator creates a Network Policy for each new ACE deployment. This Network Policy overrides the Network Policy implemented by OSSM (which blocks all non-Istio traffic to the namespace) and allows direct access to the ACE pods via an OpenShift Route (also created by the ACE operator).

To test the ACE service via this Route (without going via the OSSM):
- In the OpenShift console, select the project where you have deployed the ACE service (e.g. `ace-istio`)
- Select *Networking* > *Routes*
- Locate the Route which matches that of the ACE service you have deployed (e.g. `toolkit-tracing-http`)
- Click the URL in the *Location* column
- Append `/ping_test/v1/server` to the URL and call it from the web browser
- Access the Kiali dashboard from the installed operators in the `istio-system` project, to validate that the service access bypasses the mesh:


![toolkit-tracing-direct](https://github.com/ot4i/CP4I-OSSM/blob/main/images/toolkit-tracing-direct.png)

To use the Istio service mesh as it's intended, however, no direct access to Kubernetes services should be allowed. For a proper usage of the Service Mesh, the **ACE server Network Policy needs to be removed**, and additional resources need to be created to expose the service outside of the service mesh. To remove the Network Policy:
- Select the project where the ACE server has been deployed (e.g. `ace-istio`)
- Navigate to *Networking* > *Network Policies* and remove the Network Policy associated with the deployment.

## Deploy v2 for A/B testing
Deploying a second version of the same ACE server will allow to use the Istio Virtual Service to split traffic across two versions: A/B testing.
- Create a new *Toolkit Integration* service from the ACE Dashboard
- Use the *Server Ping v2* test service: https://github.com/ot4i/CP4I-OSSM/blob/main/ace/testAPIs/serverPingv2.bar
- Follow the same instructions to deploy *Server Ping*

## Expose ACE service via the mesh ingress
For the ACE service to be exposed via the mesh ingress, additional Istio resources need to be created: an Istio Gateway, a Virtual Service, one (or more) Destination Rules.

The yaml files in this folder can be used directly in the OpenShift console to create the additional resources required by Istio, once the correct project has been selected (e.g. `ace-istio`).


![ocp-add-resource](https://github.com/ot4i/CP4I-OSSM/blob/main/images/ocp-add-resource.png)
- Create Istio Gateway: `toolkit-tracing-gateway.yaml`
  - To automatically generate an OpenShift route, customise the `host` field with FQDN resolvable to your cluster.
- Create Virtual Service: `toolkit-tracing-virtual-service.yaml`
  - Note that this virtual service has two destinations, with traffic split 50% each
  - Make sure that the `host` fields in the two `destinations` clauses point at the names of the two Kubernetes service deployed by the ACE operator.
- Create Destination Rule: `toolkit-tracing-destionation-rules.yaml`
  - Make sure that the `host` fields in the two `destinations` clauses point at the names of the two Kubernetes service deployed by the ACE operator.
  - Note that this file contains two resources, which might have to be created once at a time.

Once the additional resources have been deployed, their correct configuration can be checked in Kiali (accessible from the installed operators in the `istio-system` project):
![toolkit-tracing-kiali-config](https://github.com/ot4i/CP4I-OSSM/blob/main/images/toolkit-tracing-kiali-config.png)

At this point the services are solely accessible from the Istio gateway, and the automatic route creation also exposes an OpenShift Route pointing directly at the gateway endpoint for this service.

- In the OpenShift console, navigate to the `istio-system` project
- Select to *Networking* > *Routes*
- Locate the Route which matches the host defined in `toolkit-tracing-gateway.yaml`: e.g. `ace-istio-toolkit-tracing-gw-xxxx`
- Copy the URL in the *Location* column
- Open this postman collection and select the *Toolkit Tracing Test*: https://github.com/ot4i/CP4I-OSSM/blob/main/ace/testAPIs/Istio-ACE.postman_collection.json
- Replace the URL with the one copied from the Route above, making sure to keep the `/ping_test/v1/server` suffix
  -  This test performs a call adding the header `X-ABTEST:TEST`. This header is mapped to the virtual service, and the traffic gets consequently split across the two ACE server versions.
- From the Kiali dashboard, it will be possible to see the service call routed from the Istio ingress via the mesh, and routed across the two server versions:


![toolkit-tracing-kiali](https://github.com/ot4i/CP4I-OSSM/blob/main/images/toolkit-tracing-kiali.png)


- Confirm that tracing is working in the Operations Dashboard


![tracing-2versions](https://github.com/ot4i/CP4I-OSSM/blob/main/images/tracing-2versions.png)
![pingtest-tracing](https://github.com/ot4i/CP4I-OSSM/blob/main/images/pingtest-tracing.png)
