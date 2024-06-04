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
### Create KYPO application credential
Create specific non-user related credential for KYPO k8s with the following command.
```
openstack application credential create --secret clzl5yUcPLJh2fd2eYGtuS8bkxpMPQdlqc5qh4lRyZuTtSyH2yuc8nOZaG_WR0wIO3vpj70UW2W4SITXKuzGcw --role Admin --role reader --unrestricted unwe-kypo
```
Result is as follows.
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

