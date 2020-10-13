# ACE app test: toolkit-tracing
- Use `Transformation_Map` test service: https://github.com/ClaudioTag/CP4I-OSSM/blob/master/ace/testAPIs/Transformation_Map.bar
- Deploy from the ACE Dashboard
- Confirm that service cannot be invoked
- Enable Istio sidecar injection to `toolkit-tracing` server by editing the deployment YAML manually: add line `sidecar.istio.io/inject: 'true'` to deployment in **spec > template > metadata > annotations**
- Create Istio Gateway: `toolkit-tracing-gateway.yaml`
- Create Virtual Service using the `istioselector` label: `toolkit-tracing-virtual-service.yaml`
- Create Destination Rule: `toolkit-tracing-destionation-rule.yaml`
- Test the API using a Postman test passing the `X-ABTEST:TEST` header: https://github.com/ClaudioTag/CP4I-OSSM/blob/master/ace/testAPIs/Istio.postman_collection.json
- Confirm that tracing is working in the Operations Dashboard:
![pingtest-tracing](https://github.com/ClaudioTag/CP4I-OSSM/blob/master/images/pingtest-tracing.png)
