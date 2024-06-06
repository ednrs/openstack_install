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
Change the name for the external network in `tfvars/deployment.tfvars`
```
sed -i 's/external_network_name = \"\"/external_network_name = \"ext_knet\"/g' tfvars/deployment.tfvars
```
Change `openstack provider` to use insecure connection.
```
sed -i 's/provider \"openstack\" {}/provider \"openstack\" = {\ninsecure=true\n}/g' provider.tf
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
Results:
```
cluster_ip = "10.18.0.35"
kubernetes_client_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIzVENDQVdLZ0F3SUJBZ0lSQUpYVVhUODRyWDFFeVNsZjRqQTRFVU13Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBMk1EWXdOVE0xTlRKYUZ3MHpOREEyTURZeApOek0xTlRKYU1ESXhGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1SY3dGUVlEVlFRREV3NTBaWEp5CllXWnZjbTB0ZFhObGNqQjJNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQWlBMklBQktNTW5MMmpLS0xlejZSK2d5blAKYm9oQ2h4R1IvZE1XWWVPRVp6VERYdE94Vm1YWXRTUHNqbnhkZ0Z6S1ZXaXBsRDBQOFVaaDA2dVUzMm1uVlduUwpXaEE2Qnl3S3lUcG5xa3lScG53VmhCUmdIRVY3QXFaeEZ6dGovVnIyNUdjbEFhTldNRlF3RGdZRFZSMFBBUUgvCkJBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGoKQkJnd0ZvQVV3SnhURTByQTEzWWZxeWZXNFNXMjBBQnJ0Y2t3Q2dZSUtvWkl6ajBFQXdNRGFRQXdaZ0l4QUxiSgpTNFNOTU1BNzh6d0N5NzdMUzZwdEtDWDEwQ2VWNVdmTTVua1RLcU5TUEo1N3JLY3U4ZENSQzYrSlA3NXNmUUl4CkFMSGxjSFc0ZVZlNDQ3ZlRKWUtsaytVOUFoY3Z3NkJWK3VWU0Zwdm9qNjRLVkFwVjlNZ3FibENQSGhUR01yQXgKVkE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
kubernetes_client_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRDUXpZWFJFbDVxZk1DYVhYVGlmV1lSN3A5Y0xibkpTNUlIb0d4Q2JxMjNXcEpydmUwdjlyU2QKV2RjQ05UWnJnODJnQndZRks0RUVBQ0toWkFOaUFBU2pESnk5b3lpaTNzK2tmb01wejI2SVFvY1JrZjNURm1IagpoR2MwdzE3VHNWWmwyTFVqN0k1OFhZQmN5bFZvcVpROUQvRkdZZE9ybE45cHAxVnAwbG9RT2djc0NzazZaNnBNCmthWjhGWVFVWUJ4RmV3S21jUmM3WS8xYTl1Um5KUUU9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
proxy_host = "10.18.0.240"
proxy_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRCNmJxK0NEcnQ3aDRrbjd4ZG10RFdTU21SWEF0T3NoOExpQ3ZBOWxlbzRoSVNGaitpWHI1WUwKQ3U1aWllOEt2VUNnQndZRks0RUVBQ0toWkFOaUFBVDhXRWNTVFFndEZtZ1k0Q3hMcHF6SEkwZVFxUzhFOFJPMQpubUpPOThJek5XcUVWMllzbkx5NVJEZVd6bFdydkswcHlTcDdFck9LOS9HZ1JpRFpJMTQycFppWVR4VStOcmI1ClQ5Yk5QamMwRHNaWjRTRU9OaFMzb1ZGaHNnMjBiTDQ9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
```
