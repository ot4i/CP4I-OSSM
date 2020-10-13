# ACE app test: designerflows-no-tracing
- Use `SimpleAPI` test API: https://github.com/ClaudioTag/CP4I-OSSM/blob/master/ace/testAPIs/simpleAPI.bar
- Deploy from the ACE Dashboard
- Confirm that API cannot be invoked
- Enable Istio sidecar injection to `designerflows-no-tracing` server by editing the deployment YAML manually: add line `sidecar.istio.io/inject: 'true'` to deployment in **spec > template > metadata > annotations**
- Create Istio Gateway: `designerflows-no-tracing-gateway.yaml`
- Create Virtual Service using the `istioselector` label: `designerflows-no-tracing-virtual-service.yaml`
- Create Destination Rule: `designerflows-no-tracing-destionation-rule.yaml`
- Confirm configuration in Kiali:
![designerflows-no-tracing-kiali](https://github.com/ClaudioTag/CP4I-OSSM/blob/master/images/designerflows-no-tracing-kiali.png)
- Ping API on http://designerflows-no-tracing.icp4i-istio-2-de84486a60b3b0d17097c54f4549015a-0000.eu-gb.containers.appdomain.cloud/simpleAPI/developer/John
