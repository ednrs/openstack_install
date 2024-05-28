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
OS_AUTH_URL=https://10.6.0.23:5000/v3
OS_PROJECT_DOMAIN_NAME=admin_domain
OS_AUTH_PROTOCOL=https
OS_USERNAME=admin
OS_AUTH_TYPE=password
OS_USER_DOMAIN_NAME=admin_domain
OS_PROJECT_NAME=admin
OS_PASSWORD=eilushoobeeThe6Y
OS_IDENTITY_API_VERSION=3
```
## OpeStack endpoints
Get the endpoints of OpenStack services.
```
openstack endpoint list --interface admin
```
And save the results somvewhere in some file.

| ID                               | Region    | Service Name | Service Type    | Enabled | Interface | URL                                            |
|----------------------------------|-----------|--------------|-----------------|---------|-----------|-----------------------------------------|
| 0c8203ca00984e07a56af47a22899c5a | RegionOne | placement    | placement       | True    | admin     | https://10.6.0.14:8778                         |
| 11dfa5f9eb3242f1bbef4caedb02fc5f | RegionOne | image-stream | product-streams | True    | admin     | https://10.6.0.19:443/swift/simplestreams/data |
| 4262548559ec49119d199814e5ef89df | RegionOne | swift        | object-store    | True    | admin     | https://10.6.0.19:443/swift                    |
| 6fcbb8681d9044e8b3a46f752822fb41 | RegionOne | neutron      | network         | True    | admin     | https://10.6.0.27:9696                         |
| 9053e57a047b4fe1879c337deef4e188 | RegionOne | cinderv3     | volumev3        | True    | admin     | https://10.6.0.7:8776/v3/$(tenant_id)s         |
| 94e3243799ab4979aeddd5d1e0186478 | RegionOne | nova         | compute         | True    | admin     | https://10.6.0.24:8774/v2.1                    |
| c70c5f82d1904d88af7c408b3b3c82b1 | RegionOne | barbican     | key-manager     | True    | admin     | https://10.6.0.9:9312                          |
| cb08939b9ce94b2888ea56b711004557 | RegionOne | keystone     | identity        | True    | admin     | https://10.6.0.23:35357/v3                     |
| cf768762fbf14011813452a0d76dba19 | RegionOne | s3           | s3              | True    | admin     | https://10.6.0.19:443/                         |
| e57ffda491f3437c82e3c34cb51e0d88 | RegionOne | glance       | image           | True    | admin     | https://10.6.0.21:9292                         |
| ec9817da21a14ad8ad6c2edd515c44bd | RegionOne | octavia      | load-balancer   | True    | admin     | https://10.6.0.15:9876                         |

## Access admin OpenStack Dashboard
Get the address of the dashboard
```
juju status --format=yaml openstack-dashboard | grep public-address | awk '{print $2}' | head -1
```
Open Horizon dahboard
```
https://10.6.0.28/horizon
```
Get the admin password
```
juju exec --unit keystone/leader leader-get admin_passwd
```
and authenticate with user ```admin``` on domain ```admin_domain```

## Storage CEPH Pool
Create ceph pool of type RDB and share it within hypervisors of Nova service.
```
juju config nova-compute libvirt-image-backend=rbd
juju config nova-compute rbd-pool=cinder-ceph
```
## Configure protocol of the Nova console
```
juju config nova-cloud-controller console-access-protocol=spice
```
With this you finished the basic confguration of the OpenStack services.
