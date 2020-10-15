# ACE app test: toolkit-no-tracing
This example deploys an ACE application designed via the ACE toolkit, with no Operational Dashboard nor Designer sidecars.

## Sidecar injection
To enable Istio sidecar injection, at deployment time you can add a custom annotation to the ACE operator.
- Create a new service from the ACE Dashboard
- Use the `Server Ping` test service: https://github.com/ot4i/CP4I-OSSM/blob/master/ace/testAPIs/serverPing.bar
- Enable `Advanced Settings`
- Add an `Advanced: Annotation`
  - operand_create_name: `sidecar.istio.io/inject`
  - operand_create_value: `true`


![toolkit-no-tracing-annotation](https://github.com/ot4i/CP4I-OSSM/blob/dev/images/toolkit-no-tracing-annotation.png)


This will add the annotation `sidecar.istio.io/inject: 'true'` to the ACE deployment metadata, which in turn will allow for envoy sidecar injection.

## Out-of-the-box ACE service
Deploying the ACE server automatically creates a Kubernetes service and an OpenShift route for that server. As the ACE operator creates a Network Policy which overrides the Istio one, the service is directly accessible from the link created in *Networking* > *Routes*. To access it:
- Click on the service location (e.g. `toolkit-no-tracing-http`)
- Append `/ping_test/v1/server` to the URL to test the service functionality.
- Access the Kiali dashboard from the installed operators in the `istio-system` project, to validate that the service access bypasses the mesh:
![toolkit-no-tracing-direct](https://github.com/ot4i/CP4I-OSSM/blob/dev/images/toolkit-no-tracing-direct.png)

To use the Istio service mesh as it's intended, no direct access to Kubernetes services should be allowed. For a proper usage of the Service Mesh, the **ACE server Network Policy needs to be removed**, and additional resources need to be created to expose the service outside of the service mesh. To remove the Network Policy:
- Select the project where the ACE server has been deployed (e.g. `ace`)
- Navigate to *Networking* > *Network Policies* and remove the Network Policy associated with the deployment.

## Expose ACE service via the mesh ingress
For the ACE service to be exposed via the mesh ingress, additional Istio resources need to be created: an Istio Gateway, a Virtual Service, one (or more) Destination Rules.

The yaml files in this folder can be used directly in the OpenShift console to create the additional resources required by Istio, once the correct project has been selected (e.g. `ace`):
- Create Istio Gateway: `toolkit-no-tracing-gateway.yaml`
  - To automatically generate an OpenShift route, customise the `host` field with FQDN resolvable to your cluster.
- Create Virtual Service: `toolkit-no-tracing-virtual-service.yaml`
  - Make sure that the `host` field in the `route` clause points at the name of the Kubernetes service deployed by the ACE operator.
- Create Destination Rule: `toolkit-no-tracing-destionation-rule.yaml`
  - Make sure that the `host` field in the `spec` clause points at the name of the Kubernetes service deployed by the ACE operator.

Once the additional resources have been deployed, their correct configuration can be checked in Kiali (accessible from the installed operators in the `istio-system` project):
![toolkit-no-tracing-kiali-config](https://github.com/ot4i/CP4I-OSSM/blob/dev/images/toolkit-no-tracing-kiali-config.png)

At this point the service is solely accessible from the Istio gateway, and the automatic route creation also exposes an OpenShift Route pointing directly at the gateway endpoint for this service.

- Navigate to the `istio-system` project
- Navigate to *Networking* > *Routes*
- Select the location for the route corresponding to the host defined in `toolkit-no-tracing-gateway.yaml`: e.g. `ace-istio-toolkit-no-tracing-gw-xxxx`
- Append `/ping_test/v1/server` to the URL to test the service functionality.
- The correct access mode will be visible in the Kiali dashboard:
![toolkit-no-tracing-kiali](https://github.com/ot4i/CP4I-OSSM/blob/dev/images/toolkit-no-tracing-flow.png)
