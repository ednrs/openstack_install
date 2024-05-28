# All steps to create network k8s cluster and juju controller inside Openstack 
## Create network
External net and subnet as VLAN &mdash; created in ProxMox with DNS from MAAS.
```
openstack network create --external --share \
   --provider-network-type vlan --provider-segment 10 \
   --provider-physical-network physnet1 \
   ext_net

openstack subnet create --network ext_net --no-dhcp  \
   --gateway 10.10.0.1 --subnet-range 10.10.0.0/24 \
   --allocation-pool start=10.10.0.10,end=10.10.0.250 \
   --dns-nameserver 10.6.0.10 \
   ext_subnet
```
Internal net named *kube* and subnet (whatever address range) with DNS from MAAS.
```
openstack network create --internal kube

openstack subnet create --network kube --dns-nameserver 10.6.0.10 \
   --subnet-range 10.12.0/24 \
   --allocation-pool start=10.12.0.20,end=10.12.0.100 \
   kube_subnet
```
Create *router* between external and internal networks.
```
openstack router create kube_router
openstack router add subnet kube_router kube_subnet
openstack router set kube_router --external-gateway ext_net
```





