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
OS_AUTH_URL=https://10.6.0.52:5000/v3
OS_PROJECT_DOMAIN_NAME=admin_domain
OS_AUTH_PROTOCOL=https
OS_USERNAME=admin
OS_AUTH_TYPE=password
OS_USER_DOMAIN_NAME=admin_domain
OS_PROJECT_NAME=admin
OS_PASSWORD=gi9woo2aeciePeep
OS_IDENTITY_API_VERSION=3
```
## OpeStack endpoints
Get the endpoints of OpenStack services.
```
openstack endpoint list --interface admin
```
And save the results somvewhere in some file.

| ID                               | Region    | Service Name | Service Type    | Enabled | Interface | URL                                     |
| ---------------------------------- | ----------- | -------------- | ----------------- | --------- | ----------- | -----------------------------------------|
| 070e2a74d400470c9d0067d6e02db610 | RegionOne | swift        | object-store    | True    | admin     | https://10.6.0.49:443/swift             |
| 1d2f8a049b6d4e3f83f6f018bca89aca | RegionOne | cinderv3     | volumev3        | True    | admin     | https://10.6.0.44:8776/v3/$(tenant_id)s |
| 581887b012d74d3093c950cc959a035c | RegionOne | keystone     | identity        | True    | admin     | https://10.6.0.52:35357/v3              |
| 5de255a92345463995b4c9c92e6e9a9d | RegionOne | image-stream | product-streams | True    | admin     | http://10.6.0.50                        |
| 71201cb695b64ccc8019bcc8a3bce3bd | RegionOne | nova         | compute         | True    | admin     | https://10.6.0.39:8774/v2.1             |
| 9bed71c3783e4e889c55b68934551105 | RegionOne | s3           | s3              | True    | admin     | https://10.6.0.49:443/                  |
| 9fe008255de646e487c4862ed17719c8 | RegionOne | placement    | placement       | True    | admin     | https://10.6.0.47:8778                  |
| d501c6fe11f14476905fb752ba9f0459 | RegionOne | barbican     | key-manager     | True    | admin     | https://10.6.0.55:9312                  |
| d9bd273ae1fa4f27bc2208cdf3a75b20 | RegionOne | octavia      | load-balancer   | True    | admin     | https://10.6.0.34:9876                  |
| f235724993a345f6ba6cc1b8e2db1c5a | RegionOne | neutron      | network         | True    | admin     | https://10.6.0.45:9696                  |
| f7b8093aa1f6484295dbfb86c5b4bbaf | RegionOne | glance       | image           | True    | admin     | https://10.6.0.35:9292                  |


## Access admin OpenStack Dashboard
Get the address of the dashboard
```
juju status --format=yaml openstack-dashboard | grep public-address | awk '{print $2}' | head -1
```
Open Horizon dahboard
```
https://10.6.0.43/horizon
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
