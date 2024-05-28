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
| ID                                   | Name                                                          | Status |
| ------------------------------------ | ------------------------------------------------------------- | ------ |
| 5492b5f8-0ac5-497c-944a-d9a1986a1ca3 | amphora-haproxy-x86_64-ubuntu-22.04-20240514                  | active |
| 0eda49ef-3d20-45ed-b953-143d85613dd5 | auto-sync/ubuntu-bionic-18.04-amd64-server-20230607-disk1.img | active |
| 9c5d5c6f-9c3c-49ae-87d2-a1499c612ec5 | auto-sync/ubuntu-focal-20.04-amd64-server-20240513-disk1.img  | active |
| f94f1bd6-3ad8-4113-aa35-f862f452d018 | auto-sync/ubuntu-jammy-22.04-amd64-server-20240514-disk1.img  | active |
| d59f2039-b6df-49e6-84f4-bf5b2caa7ccb | auto-sync/ubuntu-trusty-14.04-amd64-server-20191107-disk1.img | active |
| 0edf0547-f2fc-407f-bf85-d428c2dbf83a | auto-sync/ubuntu-xenial-16.04-amd64-server-20211001-disk1.img | active |

Set environment variables for bootstraping with the following commands:
```
IMAGE_ID=$(openstack image show  auto-sync/ubuntu-jammy-22.04-amd64-server-20240514-disk1.img -f yaml -c id | awk '{print $2}')
INT_NET_ID=$(openstack network show kube -f yaml -c id | awk '{print $2}')
FIP_ID=$(openstack network show ext_net -f yaml -c id | awk '{print $2}')

SUB_EXT_ID=$(openstack subnet show  ext_subnet -f yaml -c id | awk '{print $2}')
SUB_KUBE_ID=$(openstack subnet show  kube_subnet -f yaml -c id | awk '{print $2}')
```
### Add Openstack credential for the new Juju controller
```
cat << EOF >  kube-cloud.yaml
clouds:
  kube:
    type: openstack
    auth-types: [access-key, userpass]
    endpoint: https://10.6.0.23:5000/v3
    regions:
      RegionOne:
        endpoint: https://10.6.0.23:5000/v3
    ca-certificates: 
    - |
EOF

cat /root/snap/openstackclients/common/root-ca.crt | sed 's/^/      /g' >> kube-cloud.yaml
```
Apply the file.
```
juju add-cloud --client -f kube-cloud.yaml
```
### Create flavor for Juju controller instance
Two virtual CPUs and 4GB RAM &mdash; you do not need more.
```
openstack flavor create juju.controller --id juju --ram 4096 --disk 40 --vcpus 2
```
### Bootstrap the controller
```
juju bootstrap  kube --bootstrap-image=$IMAGE_ID --bootstrap-base=ubuntu@22.04 --config network=$INT_NET_ID --config external-network=$FIP_ID --bootstrap-constraints arch=amd64 --constraints allocate-public-ip=true 
```

