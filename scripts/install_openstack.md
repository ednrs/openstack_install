# Install Openstack with bundle and overlay

## Bootrap JUJU
```
juju bootstrap --bootstrap-series=jammy --constraints tags=juju maas-one maas-controller
```
### Add model with specification
```
add-model --config default-series=jammy --config default-space=public openstack
juju model-defaults default-space=public
juju model-config default-space=public
juju set-model-constraints spaces=admin,internal,public
```
## Deploy OpenStack bundle with octavia overlay
```
juju deploy ./bundle.yaml --overlay octavia-overlay.yaml
```
### Watch progress
Machines
```
watch -c juju machines --color
```
Watch the following command until ```nova-cloud-controller``` and ```keystone``` are active and idle and ```vault``` is in status blocked.
```
watch -c juju status vault keystone neutron-api nova-cloud-controller --color
```
## Unlock the vault
When ```nova-cloud-controller``` and ```keystone``` became active and idle unseal the wault with following commands:

Set address of the vault as environment variable (get the address of the vault from ```juju status```
```
export VAULT_ADDR="http://10.6.0.25:8200"
```
Initiate vault with five keys (three main and two reserve eys)
```
vault operator init -key-shares=5 -key-threshold=3
```
