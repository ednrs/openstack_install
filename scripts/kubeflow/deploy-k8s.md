# Deploy the cluter
## Prepare flavors
You can change the flavors according your resources. The contraints in [bundle and overlays files](/scripts/bundles/kubeflow) have to be changed according to your flavors set-up.
```
openstack flavor create min.juju.machine --id min.juju --ram 2048 --disk 8 --vcpus 1
openstack flavor create small.juju.machine --id small.juju --ram 8192 --disk 16 --vcpus 1
openstack flavor create medium.juju.machine --id medium.juju --ram 16384 --disk 32 --vcpus 2
openstack flavor create big.juju.machine --id big.juju --ram 32768 --disk 64 --vcpus 8
openstack flavor create lb.juju.machine --id lb.juju --ram 4096 --disk 16 --vcpus 1
```
## Set-up the quotas
```
openstack quota set --instances 32 --class default
openstack quota set --ram 204800 --class default
openstack quota set --cores 48 --class default
PROJ_ID=$(openstack project show admin -f yaml -c id | awk '{print $2}')
openstack quota set --secgroups 128 $PROJ_ID
```
## Add juju model to the cloud *kube*
The configuration of the model set use of floating IPs from the external network, external and internal network IDs.   
```
juju add-model --config default-series=jammy kube

juju model-defaults use-floating-ip=true
juju model-defaults allocate-public-ip=true
juju model-defaults network=$INT_NET_ID
juju model-defaults external-network=$FIP_ID
juju set-model-constraints allocate-public-ip=true
```
## Deploy the cluster
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
And after all machines are started, watch the status
```
watch -c juju status --color
```
