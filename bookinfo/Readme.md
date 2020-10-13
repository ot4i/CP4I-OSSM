# Bookinfo

Deplyoment of bookinfo to a namespace in the service mesh member roll.
In this example: `istio-test-ns-01`
Instructions based on OCP documentation: https://docs.openshift.com/container-platform/4.3/service_mesh/service_mesh_day_two/ossm-example-bookinfo.html

### Deploy Bookinfo
- Deploy single bookinfo yaml from Maistra GitHub repo:
```
kubectl apply -n istio-test-ns-01 -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/platform/kube/bookinfo.yaml
```
- Test from container in the same namaspace:
```
curl http://productpage.istio-test-ns-01.svc.cluster.local:9080
```
-> It should return the prodcutpage html
- Test from container in another namespace:
```
curl http://productpage.istio-test-ns-01.svc.cluster.local:9080
```
-> It should not return anything

### Deploy Gateway
- Deploy the gateway yaml from Maistra GitHub repo:
```
kubectl apply -n istio-test-ns-01 -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/networking/bookinfo-gateway.yaml
```
**Note:** it didn't work from terminal, but worked from KUI

- Customise the Istio Gateway to add hosts. See example here: https://github.com/ClaudioTag/CP4I-OSSM/blob/master/bookinfo/bookinfo-gateway.yml
This will automatically generate routes in OpenShift if IOR is enabled.

- Export value of gateway URL
```
export GATEWAY_URL=$(oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}')
echo $GATEWAY_URL
```

### Deploy Destination Rules
- Deploy the Destination Rules yaml from Maistra GitHub repo without TLS:
```
kubectl apply -n istio-test-ns-01 -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/networking/destination-rule-all.yaml
```
**OR**
- Deploy the Destination Rules yaml from Maistra GitHub repo with TLS:
```
kubectl apply -n istio-test-ns-01 -f https://raw.githubusercontent.com/Maistra/istio/maistra-1.1/samples/bookinfo/networking/destination-rule-all-mtls.yaml
```

### Test deployment
- The bookinfo app will be exposed at ```http://$GATEWAY_URL/productpage```
- If automatic route creation (IOR) is enabled, there will also be a route created for the app in the `istio-system` namespace
