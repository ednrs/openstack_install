# Install KYPO resources

## Create Domain, Project and user for KYPO
TBD

### Open KYPO Project credential
```
source kypo-project
```
Ensure that project credential are loaded.
```
env | grep OS_
OS_REGION_NAME=RegionOne
OS_AUTH_VERSION=3
OS_CACERT=/root/snap/openstackclients/common/root-ca.crt
OS_AUTH_URL=https://10.6.0.52:5000/v3
OS_PROJECT_DOMAIN_NAME=cyber_range
OS_AUTH_PROTOCOL=https
OS_USERNAME=admcr
OS_AUTH_TYPE=password
OS_USER_DOMAIN_NAME=cyber_range
OS_PROJECT_NAME=kypo
OS_PASSWORD=123456
OS_IDENTITY_API_VERSION=3
```
## Create external and iternal network for KYPO 
Create extenal network `10.18.0.0/24` as a VLAN, name `ext_knet`, `tag 18` (it is configured on ProxMox Open vSwitch) and internal Opnestack's network `10.25.0.0/24`, name 'kypo_net`. 
```
openstack network create --external --share \
   --provider-network-type vlan --provider-segment 18 \
   --provider-physical-network physnet1 \
   ext_knet

openstack subnet create --network ext_knet --no-dhcp  \
   --gateway 10.18.0.1 --subnet-range 10.18.0.0/24 \
   --allocation-pool start=10.18.0.10,end=10.18.0.250 \
   --dns-nameserver 10.6.0.10 \
   ext_ksubnet

openstack network create --internal kypo

openstack subnet create --network kypo --dns-nameserver 10.6.0.10 \
   --subnet-range 10.25.0/24 \
   --allocation-pool start=10.25.0.20,end=10.25.0.100 \
   kypo_subnet
```
Create router between two networks.
```
openstack router create kypo_router
openstack router add subnet kypo_router kypo_subnet
openstack router set kypo_router --external-gateway ext_knet
```

### Create KYPO application credential
Create specific non-user related credential for KYPO k8s with the following command.
```
openstack application credential create --secret clzl5yUcPLJh2fd2eYGtuS8bkxpMPQdlqc5qh4lRyZuTtSyH2yuc8nOZaG_WR0wIO3vpj70UW2W4SITXKuzGcw --role Admin --role reader --unrestricted unwe-kypo
```
Result from the cmmand.

| Field        | Value                                                                                  |
|--------------|----------------------------------------------------------------------------------------|
| description  | None                                                                                   |
| expires_at   | None                                                                                   |
| id           | e9c87d6055ac46b29c749b6b308550c9                                                       |
| name         | unwe-kypo                                                                              |
| project_id   | c02dd12876754820b97b8b0ffc6ed406                                                       |
| roles        | Admin reader                                                                           |
| secret       | clzl5yUcPLJh2fd2eYGtuS8bkxpMPQdlqc5qh4lRyZuTtSyH2yuc8nOZaG_WR0wIO3vpj70UW2W4SITXKuzGcw |
| system       | None                                                                                   |
| unrestricted | True                                                                                   |
| user_id      | 363b80d43c8841fdbacd16d1f85e350c                                                       |

With this information (`id` and `secret`) create file with KYPO application credential

```
cat << EOF >  kypo-application
#!/bin/bash

export OS_AUTH_TYPE=v3applicationcredential
export OS_AUTH_URL=https://10.3.0.52:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_REGION_NAME=RegionOne
export OS_INTERFACE=public
export OS_APPLICATION_CREDENTIAL_ID=e9c87d6055ac46b29c749b6b308550c9
export OS_APPLICATION_CREDENTIAL_SECRET=clzl5yUcPLJh2fd2eYGtuS8bkxpMPQdlqc5qh4lRyZuTtSyH2yuc8nOZaG_WR0wIO3vpj70UW2W4SITXKuzGcw
export OS_CACERT=/root/snap/openstackclients/common/root-ca.crt
EOF
```
Open new terminal, ssh to your machine again to clear all previous credential and use kypo-application file.
```
source kypo-application
```


## Get the KYPO installation files
Clone github repository of KYPO
```
cd /opt 
git clone https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-crp-tf-deployment.git
```
Enter directory for base OpenStack resources installation
```
cd kypo-crp-tf-deployment/tf-openstack-base/
```
Copy vars tmplate to vars file
```
cp tfvars/deployment.tfvars-template tfvars/deployment.tfvars
```

```
