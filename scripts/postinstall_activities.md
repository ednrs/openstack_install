# OpenStack Postinstall
## Configure OpenStack Client
Get and prepare openstack source script.
```
wget -c https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/_downloads/c894c4911b9572f0b5f86bdfc5d12d8e/openrc
sed -i 's\mkdir $_snap_user_dir\mkdir -p $_snap_user_dir\g' openrc
```
