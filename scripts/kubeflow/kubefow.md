# Install Kubeflow
## Bootstrap Juju controller inside k8s cluster
New cloud of type *k8s* should to be added.
```
juju add-k8s mlkube --client
```
With controller *kubeflow-controller*
```
juju bootstrap mlkube kubeflow
```
New pod named *controller-kubeflow* appear.
## Deploy model and Kubeflow bundle
```
juju add-model kubeflow
```
And then deploy charmed Kubeflow.
```
juju deploy kubeflow --trust  --channel=latest/stable	
```
Watch `juju status` and wait untill all services are installed. It takes time.

## Configure access
Get the LoadBalancer address.
```
kubectl -n kubeflow get svc istio-ingressgateway-workload -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```
Configure some address. It can be domain, just the IP of the LoadBalancer or combination of the IP as a subdomain of a fake domain.
```
juju config dex-auth public-url=http://10.10.0.143.nip.io
juju config oidc-gatekeeper public-url=http://10.10.0.143.nip.io
```
Finally configure user and password.
```
juju config dex-auth static-username=admin
juju config dex-auth static-password=admin
```
and point your browser to `http://10.10.0.143.nip.io`
