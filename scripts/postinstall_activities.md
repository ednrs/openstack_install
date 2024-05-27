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
