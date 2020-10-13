# ACE app test: designerflows-tracing
- Use `SimpleAPI` test API: https://github.com/ClaudioTag/CP4I-OSSM/blob/master/ace/testAPIs/simpleAPI.bar
- Deploy from the ACE Dashboard
- Confirm that API cannot be invoked
- Enable Istio sidecar injection to `designerflows-tracing` server by editing the deployment YAML manually: add line `sidecar.istio.io/inject: 'true'` to deployment in **spec > template > metadata > annotations**
- Create Istio Gateway: `designerflows-tracing-gateway.yaml`
- Create Virtual Service using the `istioselector` label: `designerflows-tracing-virtual-service.yaml`
- Create 2 Destination Rules: `designerflows-tracing-destionation-rule.yaml`
- Test the API using a Postman test passing the `X-ABTEST:TEST` header: https://github.com/ClaudioTag/CP4I-OSSM/blob/master/ace/testAPIs/Istio.postman_collection.json
- Confirm that tracing is working in the Operations Dashboard:
![simpleAPI-tracing](https://github.com/ClaudioTag/CP4I-OSSM/blob/master/images/simpleAPI-tracing.png)
- Kiali picks traffic flowing out of the ACE server (to the tracing sidecars)
![simpleAPI-kiali-tracing](https://github.com/ClaudioTag/CP4I-OSSM/blob/master/images/simpleAPI-kiali-tracing.png)
