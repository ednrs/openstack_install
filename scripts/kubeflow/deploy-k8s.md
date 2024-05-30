# Deploy and configure the cluster
## Deploy
### Prepare flavors
**NB:** The requirements for workers according to [Kubeflow charmed howto](https://charmed-kubeflow.io/docs/get-started-with-charmed-kubeflow#heading--deploy-charmed-kubeflow) is to have at least 4 cores, 32GB RAM and 50GB of disk space available. Otherwise the deployment will fail.

You can change the flavors according your resources. The contraints in [bundle and overlays files](/scripts/bundles/kubeflow) have to be changed according to your flavors set-up.
```
openstack flavor create min.juju.machine --id min.juju --ram 2048 --disk 8 --vcpus 1
openstack flavor create small.juju.machine --id small.juju --ram 8192 --disk 16 --vcpus 1
openstack flavor create medium.juju.machine --id medium.juju --ram 16384 --disk 32 --vcpus 2
openstack flavor create big.juju.machine --id big.juju --ram 32768 --disk 64 --vcpus 8
openstack flavor create lb.juju.machine --id lb.juju --ram 4096 --disk 16 --vcpus 1
```
### Set-up the quotas
```
openstack quota set --instances 32 --class default
openstack quota set --ram 204800 --class default
openstack quota set --cores 48 --class default
openstack quota set --volumes 100 --class default
PROJ_ID=$(openstack project show admin -f yaml -c id | awk '{print $2}')
openstack quota set --secgroups 128 $PROJ_ID
```
### Add juju model to the cloud *kube*
The configuration of the model set use of floating IPs from the external network, external and internal network IDs.   
```
juju add-model --config default-series=jammy kube

juju model-defaults use-floating-ip=true
juju model-defaults allocate-public-ip=true
juju model-defaults network=$INT_NET_ID
juju model-defaults external-network=$FIP_ID
juju set-model-constraints allocate-public-ip=true
```
### Deploy the cluster
```
juju deploy ./flow-kube.yaml --overlay openstack-integrator-overlay.yaml --trust --overlay ovn-kube-overlay.yaml
```
When command finisheds configure `openstack-integrator`
```
juju trust openstack-integrator
juju config openstack-integrator lb-floating-network=$FIP_ID lb-subnet=$SUB_KUBE_ID manage-security-groups=true
```
Configure ingress access to workers with the following commands:
```
juju config kubernetes-worker ingress=true
juju config kubernetes-worker ingress-use-forwarded-headers=true
```
Watch progress of the deployment of the machine with 
```
watch -c juju machines --color
```
### Prepare machines for Kubeflow requirements 
**NB:** At least workers should be setup with specific `inotify` values according to [Charmed Kubeflow Deplyment Guide](https://charmed-kubeflow.io/docs/get-started-with-charmed-kubeflow#heading--deploy-charmed-kubeflow).
And after all machines are started, log to machine of worker.
```
juju ssh 3
```
Beccome *root*
```
sudo -s
```
And execute the following commands:
```
sysctl fs.inotify.max_user_instances=1280
sysctl fs.inotify.max_user_watches=655360
echo "fs.inotify.max_user_instances=1280" >> /etc/sysctl.conf
echo "fs.inotify.max_user_watches=655360" >> /etc/sysctl.conf
```
You can do this on all 7 machines if you wish.
Then watch the status of model with:
```
watch -c juju status --color
```
## Configure
Get the cluster `config` file
```
mkdir -p ~/.kube
juju ssh kubernetes-control-plane/leader -- cat config > ~/.kube/config
```
Check whether all pods are running.
```
watch kubectl get pod -A
```
### Configure ngnx ingress to the dashboard
The installed nginx pods and services on the workers can not choose the leader and you need to apply the following change of configuration of the configmap for the coordination apiGroups: 
```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app.kubernetes.io/name: ingress-nginx-kubernetes-worker
    app.kubernetes.io/part-of: ingress-nginx-kubernetes-worker
  name: nginx-ingress-role-kubernetes-worker
  namespace: ingress-nginx-kubernetes-worker
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - pods
  - secrets
  - namespaces
  verbs:
  - get
- apiGroups:
  - ""
  resourceNames:
  - nginx-ingress-role-kubernetes-worker
  resources:
  - configmaps
  verbs:
  - get
  - update
- apiGroups:
  - coordination.k8s.io
  resources:
  - leases
  verbs:
  - list
  - watch
  - create
  - get
  - update
EOF
```
Now it is possible to add ingress configurations.
The following configuration setup the infgress for host `cloud.e-dnrs.org`.
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: HTTPS
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
  name: dashboard-local
  namespace: kubernetes-dashboard
spec:
  ingressClassName: nginx-ingress-controller
  rules:
  - host: cloud.e-dnrs.org
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: kubernetes-dashboard
              port:
                number: 443  
EOF
```
Then execute command
```
watch kubectl get ing -A
```
and wait IP addresses of three workers to appear.
### Configure cinder-cni storage class
In order to make cinder-cni storage class default for the cluster execute the following command:
```
kubectl patch storageclass csi-cinder-default -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
With this patch the volumes at the OpenStack Cinder will be created automatically when request from a pod appear.


