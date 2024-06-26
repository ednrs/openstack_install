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
Create extenal network `10.18.0.0/24` as a VLAN, name `ext_knet`, `tag 18` (it is configured on ProxMox Open vSwitch). 
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
```
Check networks with command `openstack network list`.
| ID                                   | Name        | Subnets                              |
|--------------------------------------|-------------|--------------------------------------|
| 264a9f6e-53fd-4810-8f1b-cbe7a2d478d9 | kypo        | 14a4f1b9-2fb5-4bf0-96bd-e7041ddecd7a |
| 56b3c018-7b4e-4722-87f3-d73b76c4e626 | ext_net     | f5510973-6589-4ea8-985b-6dbb49edbaea |
| 609d74a2-c57e-4c55-83db-e0703f11a657 | ext_knet    | 3030c5b4-f622-4518-a860-9c16b85360a0 |
| 9f4ae404-3e9a-41a6-9d93-5f412ff56b50 | kube        | d65bd2d7-ea97-41a1-80d9-843e08716f4c |
| b5e92609-7a1c-4a99-9a62-27423ac43b53 | lb-mgmt-net | 61f44676-91b3-4e2e-acfa-bc7962548d2c |

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
#!/usr/bin/env bash

export OS_AUTH_TYPE=v3applicationcredential
export OS_AUTH_URL=https://10.6.0.52:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_REGION_NAME="RegionOne"
export OS_INTERFACE=public
export OS_APPLICATION_CREDENTIAL_ID=b2c0732db0bf4c399554b1036f0283f2
export OS_APPLICATION_CREDENTIAL_SECRET=clzl5yUcPLJh2fd2eYGtuS8bkxpMPQdlqc5qh4lRyZuTtSyH2yuc8nOZaG_WR0wIO3vpj70UW2W4SITXKuzGcw
export OS_CACERT=/root/snap/openstackclients/common/root-ca.crt
EOF
```
Open new terminal, ssh to your machine again to clear all previous credential and use kypo-application file.
```
source ~/kypo-application
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
Change the name for the external network in `tfvars/deployment.tfvars`
```
sed -i 's/external_network_name = \"\"/external_network_name = \"ext_knet\"/g' tfvars/deployment.tfvars
```
Change `openstack provider` to use insecure connection.
```
sed -i 's/provider \"openstack\" {}/provider \"openstack\"  {\ninsecure=true\n}/g' provider.tf
```
Initialise terraform plan.
```
terraform init
```
If you want you can change additional varibles in `tfvars/vars-all.tfvars` file.
Install KYPO's Openstack base resources.
```
terraform apply -var-file tfvars/deployment.tfvars -var-file tfvars/vars-all.tfvars --auto-approve
```
Results ...
```
cluster_ip = "10.18.0.162"
kubernetes_client_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIyekNDQVdLZ0F3SUJBZ0lSQUtFTDJRZ3Uwa2FIZ1B3dGgvdFlHc013Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBMk1UTXhNREkxTkRGYUZ3MHpOREEyTVRNeQpNakkxTkRGYU1ESXhGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1SY3dGUVlEVlFRREV3NTBaWEp5CllXWnZjbTB0ZFhObGNqQjJNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQWlBMklBQkp1SVFrVGErMk9xRTYzRXNGQnYKajY0VnRrUTJ4ellpN3NSeGxmTUxMOWNtczRlWk5qaGpWNkFuRnF2N3N6SE9ySmwyMVd0ak5HeXBwN3JhVi81awpxT3JnekxKOGtOMnNWdkl5K1d3VHk3VDRKaC9tRDBmZXBEWm5pOTBtZGgzWDdLTldNRlF3RGdZRFZSMFBBUUgvCkJBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGoKQkJnd0ZvQVVQcm90aFpkN3VIWTYyNjJhYjhlNStGK1ZKa0V3Q2dZSUtvWkl6ajBFQXdNRFp3QXdaQUl3WG5ZdQpOQzVSWGZHd1psZFNxNVlVeDI1MEN0SlNneGpHVVE0TW9VemVIRHdmOW44VHoySGdFSW1KN0N2VE5KSFZBakIxCmdBMTd6UXVZTlhSeFR0cGVHL3hNOGp1cktVTkhtaTQvaGVOMHVwM2JYZTE2a01qVlhwcGllY0tzMk5YbjN4WT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
kubernetes_client_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRBc2hMZWIwZFdkRVZRa0ZwcDBWODVBYWRzelVIdkl0c3pUeTd0eU9nNkNkOUJ4VFNVc2QwbFAKR3NHWTJ1cURmMWVnQndZRks0RUVBQ0toWkFOaUFBU2JpRUpFMnZ0anFoT3R4TEJRYjQrdUZiWkVOc2MySXU3RQpjWlh6Q3kvWEpyT0htVFk0WTFlZ0p4YXIrN014enF5WmR0VnJZelJzcWFlNjJsZitaS2pxNE15eWZKRGRyRmJ5Ck12bHNFOHUwK0NZZjVnOUgzcVEyWjR2ZEpuWWQxK3c9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
proxy_host = "10.18.0.165"
proxy_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRDU0MxdXNmektEb1NLZ01jaEJFakFFbXM4VWZwekFlVkRaeXFkdEVoVFlDaTg2Yzh6Yml4NUYKbTlPL0xCNnRIUUtnQndZRks0RUVBQ0toWkFOaUFBUitqTWwzUmZqVi91Nk5GWjJxbE8rMjlKK1RlR3NPZEQvMgpsaTVMa3ZaN3V3YVNhRi9VdWFNcld3TytScWtJUm5VNlJsb2hLaVpIMDRrK1h2QW1YN0tnZExwQzNVSml5aUtsCitzcnp4Q2FpV2ZRTGhlTngrMm9Ld01ZeDQwVmY2cUU9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
```
... use the `config` file in the current directory `/opt/kypo-crp-tf-deployment/tf-openstack-base`.
The file:
```
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://10.18.0.162:6443
  name: default
contexts:
- context:
    cluster: default
    namespace: kypo
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIyekNDQVdLZ0F3SUJBZ0lSQUtFTDJRZ3Uwa2FIZ1B3dGgvdFlHc013Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBMk1UTXhNREkxTkRGYUZ3MHpOREEyTVRNeQpNakkxTkRGYU1ESXhGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1SY3dGUVlEVlFRREV3NTBaWEp5CllXWnZjbTB0ZFhObGNqQjJNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQWlBMklBQkp1SVFrVGErMk9xRTYzRXNGQnYKajY0VnRrUTJ4ellpN3NSeGxmTUxMOWNtczRlWk5qaGpWNkFuRnF2N3N6SE9ySmwyMVd0ak5HeXBwN3JhVi81awpxT3JnekxKOGtOMnNWdkl5K1d3VHk3VDRKaC9tRDBmZXBEWm5pOTBtZGgzWDdLTldNRlF3RGdZRFZSMFBBUUgvCkJBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGoKQkJnd0ZvQVVQcm90aFpkN3VIWTYyNjJhYjhlNStGK1ZKa0V3Q2dZSUtvWkl6ajBFQXdNRFp3QXdaQUl3WG5ZdQpOQzVSWGZHd1psZFNxNVlVeDI1MEN0SlNneGpHVVE0TW9VemVIRHdmOW44VHoySGdFSW1KN0N2VE5KSFZBakIxCmdBMTd6UXVZTlhSeFR0cGVHL3hNOGp1cktVTkhtaTQvaGVOMHVwM2JYZTE2a01qVlhwcGllY0tzMk5YbjN4WT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRBc2hMZWIwZFdkRVZRa0ZwcDBWODVBYWRzelVIdkl0c3pUeTd0eU9nNkNkOUJ4VFNVc2QwbFAKR3NHWTJ1cURmMWVnQndZRks0RUVBQ0toWkFOaUFBU2JpRUpFMnZ0anFoT3R4TEJRYjQrdUZiWkVOc2MySXU3RQpjWlh6Q3kvWEpyT0htVFk0WTFlZ0p4YXIrN014enF5WmR0VnJZelJzcWFlNjJsZitaS2pxNE15eWZKRGRyRmJ5Ck12bHNFOHUwK0NZZjVnOUgzcVEyWjR2ZEpuWWQxK3c9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K
```
### Get ssh access
You can get KYPO kunernetes cluster machine by executing.
```
terraform show --json | grep "BEGIN EC PRIVATE KEY"
```
Get the key. Paste it in file `~/kypo-ssh` and log to the machine.
```
chmod 600 ~/kypo-ssh
ssh -i ~/kypo-ssh ubuntu@10.18.0.47
```
User `ubuntu` is a sudoer with no-password sudo access.
