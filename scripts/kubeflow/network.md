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
Check networks with command ```openstack network list```

| ID                                   | Name        | Subnets                              |
| ------------------------------------ | ----------- | ------------------------------------ |
| 639b4740-314d-4b77-bfa7-469b31fc8c91 | kube        | 8dc82ca3-52cf-40c4-b991-d57ec2b15c7d |
| 6b32cd9b-73be-431b-b1d4-c94f71d6d5f1 | ext_net     | 79d0fa18-f27b-418e-891b-c006b29d0f30 |
| a2fcd593-a7f2-40b0-9c69-0f8eafed50b3 | lb-mgmt-net | 84bda264-2590-4f5f-ab15-5b077eeb0f99 |

## Setup environment variables for bootstraping of Juju controller inside Openstack
Execute command `openstack image list` to check name of image which you can use.

Set environment variables for bootstraping 


