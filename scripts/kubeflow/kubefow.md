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
