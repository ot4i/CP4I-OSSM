# ACE app test: toolkit-no-tracing
To enable Istio sidecar injection, at deployment time you can add a custom annotation to the ACE operator.
- Create a new service from the ACE Dashboard
- Use `Server Ping` test service: https://github.com/ClaudioTag/CP4I-OSSM/blob/master/ace/testAPIs/serverPing.bar
- Enable `Advanced Settings`
- Add an `Advanced: Annotation`
  - operand_create_name: `sidecar.istio.io/inject`
  - operand_create_value: `true`

This will add the annotation `sidecar.istio.io/inject: 'true'` to the ACE deployment metadata, which in turn will allow for envoy sidecar injection.

Additional resources need to be now created to expose the service outside of the Istio Gateway:
- Create Istio Gateway: `toolkit-no-tracing-gateway.yaml`
- Create Virtual Service using the `istioselector` label: `toolkit-no-tracing-virtual-service.yaml`
- Create Destination Rule: `toolkit-no-tracing-destionation-rule.yaml`
- Confirm configuration in Kiali:
![toolkit-no-tracing-kiali](https://github.com/ClaudioTag/CP4I-OSSM/blob/master/images/toolkit-no-tracing-kiali.png)
- More on testing
- Test flow: http://toolkit-no-tracing.icp4i-istio-2-de84486a60b3b0d17097c54f4549015a-0000.eu-gb.containers.appdomain.cloud/ping_test/v1/server
- Need to delete network policy
