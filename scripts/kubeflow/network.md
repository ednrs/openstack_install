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
|--------------------------------------|-------------|--------------------------------------|
| 56b3c018-7b4e-4722-87f3-d73b76c4e626 | ext_net     | f5510973-6589-4ea8-985b-6dbb49edbaea |
| 9f4ae404-3e9a-41a6-9d93-5f412ff56b50 | kube        | d65bd2d7-ea97-41a1-80d9-843e08716f4c |
| b5e92609-7a1c-4a99-9a62-27423ac43b53 | lb-mgmt-net | 61f44676-91b3-4e2e-acfa-bc7962548d2c |

## Setup environment variables for bootstraping of Juju controller inside Openstack
Execute command `openstack image list` to check name of image which you can use.
| ID                                   | Name                                                          | Status |
|--------------------------------------|---------------------------------------------------------------|--------|
| 55884174-cd2d-4233-9714-ac36e4e5d8cd | amphora-haproxy-x86_64-ubuntu-22.04-20240514                  | active |
| ff6d32ab-18ab-48c7-9049-cb5ad2a4f9a1 | auto-sync/ubuntu-bionic-18.04-amd64-server-20230607-disk1.img | active |
| 2635249b-c353-434b-b513-97bfff973886 | auto-sync/ubuntu-focal-20.04-amd64-server-20240513-disk1.img  | active |
| 57be989b-8998-448b-8432-502e6fa8d44e | auto-sync/ubuntu-jammy-22.04-amd64-server-20240514-disk1.img  | active |
| 6b521cc2-a366-4058-a786-df989d4fd994 | auto-sync/ubuntu-trusty-14.04-amd64-server-20191107-disk1.img | active |
| 6958d32a-27c8-467c-bd6c-e081788a6477 | auto-sync/ubuntu-xenial-16.04-amd64-server-20211001-disk1.img | active |

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
Then execute the following comand, select `1` and press `enter`.
```
juju autoload-credentials --client
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

