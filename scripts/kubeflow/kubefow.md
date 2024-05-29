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
juju config dex-auth public-url=http://10.10.0.145.nip.io
juju config oidc-gatekeeper public-url=http://10.10.0.145.nip.io
```
Finally configure user and password.
```
juju config dex-auth static-username=admin
juju config dex-auth static-password=admin
```
and point your browser to `http://10.10.0.143.nip.io. Maybe you need to open access to the LoadBalancer in its Security Group in OpenStack.
## Configure access with NodePort
If you want to configure access as a NodePort Service with ingress you can execute the following commands:
```
export EDITOR=nano
kubectl edit svc istio-ingressgateway-workload -n kubeflow
```
Change `type: LoadBalancer` to `type: NodePort` and then:
```
cat <<EOF | kubectl apply -f -
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubeflow-ing
  namespace: kubeflow
  labels:
    app: kubeflow-gateway
  annotations:
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  ingressClassName: nginx-ingress-controller
  rules:
  - host: 10.10.0.45.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: istio-ingressgateway-workload 
            port:
              number: 80
EOF
```
The address `10.10.0.45` is the workers leader IP address.
Configure the dex service:
```
juju config dex-auth public-url=http://10.10.0.45.nip.io
juju config oidc-gatekeeper public-url=http://10.10.0.45.nip.io
```

