# OpenStack Postinstall
## Configure OpenStack Client
Get and prepare openstack source script.
```
wget -c https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/_downloads/c894c4911b9572f0b5f86bdfc5d12d8e/openrc
sed -i 's\mkdir $_snap_user_dir\mkdir -p $_snap_user_dir\g' openrc
```
Add Openstack credential environment variables aand execute following command.
```
source ~/openrc
```
Get environment variables.
```
env | grep OS_
```
And save them.
```
OS_REGION_NAME=RegionOne
OS_AUTH_VERSION=3
OS_CACERT=/root/snap/openstackclients/common/root-ca.crt
OS_AUTH_URL=https://10.6.0.15:5000/v3
OS_PROJECT_DOMAIN_NAME=admin_domain
OS_AUTH_PROTOCOL=https
OS_USERNAME=admin
OS_AUTH_TYPE=password
OS_USER_DOMAIN_NAME=admin_domain
OS_PROJECT_NAME=admin
OS_PASSWORD=AiWioZ1chohroori
OS_IDENTITY_API_VERSION=3
```
## OpeStack endpoints
Get the endpoints of OpenStack services.
```
openstack endpoint list --interface admin
```
And save the results somvewhere in some file.

| ID                               | Region    | Service Name | Service Type    | Enabled | Interface | URL                                     |
|----------------------------------|-----------|--------------|-----------------|---------|-----------|-----------------------------------------|
| 16fbc2a771e6481f9a3ec9d75032ffa2 | RegionOne | swift        | object-store    | True    | admin     | https://10.6.0.28:443/swift             |
| 2b65fdbeb86b4446a135aecfec5990f5 | RegionOne | s3           | s3              | True    | admin     | https://10.6.0.28:443/                  |
| 4ca4ac444415427787164dfa5e938ad3 | RegionOne | neutron      | network         | True    | admin     | https://10.6.0.34:9696                  |
| 6330f0df16cf482a8694afb694d84807 | RegionOne | keystone     | identity        | True    | admin     | https://10.6.0.15:35357/v3              |
| 665669e352ee4a4b911913a0e30723fb | RegionOne | octavia      | load-balancer   | True    | admin     | https://10.6.0.21:9876                  |
| 842144b197b44a59be0b02f099fd79d0 | RegionOne | placement    | placement       | True    | admin     | https://10.6.0.30:8778                  |
| 8476cea6e7234c57afb760797f18d9f3 | RegionOne | image-stream | product-streams | True    | admin     | http://10.6.0.13                        |
| 85b4e8566aa84e85a4ca6db6eacf2bb6 | RegionOne | barbican     | key-manager     | True    | admin     | https://10.6.0.18:9312                  |
| 85fe3e8da9f445819f6b6e6b22b50f50 | RegionOne | cinderv3     | volumev3        | True    | admin     | https://10.6.0.17:8776/v3/$(tenant_id)s |
| dc09beff6812427b90c845567d2e71c8 | RegionOne | glance       | image           | True    | admin     | https://10.6.0.24:9292                  |
| ec684fe0ff9548129d7a5ff622774b90 | RegionOne | nova         | compute         | True    | admin     | https://10.6.0.16:8774/v2.1             |            |

## Access admin OpenStack Dashboard
Get the address of the dashboard
```
juju status --format=yaml openstack-dashboard | grep public-address | awk '{print $2}' | head -1
```
Open Horizon dahboard
```
https://10.6.0.32/horizon
```
Get the admin password
```
juju exec --unit keystone/leader leader-get admin_passwd
```
and authenticate with user ```admin``` on domain ```admin_domain```
