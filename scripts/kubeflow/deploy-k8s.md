# Deploy the cluter
## Prepare flavors
You can change the flavors according your resources. The contraints in [bundle and overlays files](/scripts/bundles/kubeflow) have to be changed according to your flavors set-up.
```
openstack flavor create min.juju.machine --id min.juju --ram 2048 --disk 8 --vcpus 1
openstack flavor create small.juju.machine --id small.juju --ram 8192 --disk 16 --vcpus 1
openstack flavor create medium.juju.machine --id medium.juju --ram 16384 --disk 64 --vcpus 2
openstack flavor create big.juju.machine --id big.juju --ram 24576 --disk 64 --vcpus 8
openstack flavor create lb.juju.machine --id lb.juju --ram 4096 --disk 16 --vcpus 2
```
