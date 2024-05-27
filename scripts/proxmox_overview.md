# Overview of ProxMox

## Machines
- MAAS Controller (VM machine)
- JUJU Controller (VM machine)
- Three VM machines as nodes for the Openstack
- One template for MAAS Controller (VM machine)
- Ansible proxmox container
- Nginx reverse proxy container
![](/scripts/images/pmx.png)
## Network configuration
OVS Bridge (vmbr0) configured with the file ```/etc/network/interfaces``` with 5 internal vlans and one external network (outside).
Four 1GB ports (eno1..3) and two 10GB ports (ens1f0 and ens1f1). Only ens1f0 is added to the bridge.
```
auto lo
iface lo inet loopback

iface eno3 inet manual

iface eno4 inet manual

iface eno2 inet manual

auto ens1f0
iface ens1f0 inet manual
	ovs_type OVSPort
	ovs_mtu 9000
	ovs_bridge vmbr0

iface ens1f1 inet manual

iface eno1 inet manual

auto vlan7i
iface vlan7i inet static
	address 10.7.0.1/24
	ovs_type OVSIntPort
	ovs_bridge vmbr0
	ovs_mtu 9000
	ovs_options tag=7

auto vlan8i
iface vlan8i inet static
	address 10.8.0.1/24
	ovs_type OVSIntPort
	ovs_bridge vmbr0
	ovs_mtu 9000
	ovs_options tag=8

auto vlan10i
iface vlan10i inet static
	address 10.10.0.1/24
	ovs_type OVSIntPort
	ovs_bridge vmbr0
	ovs_mtu 9000
	ovs_options tag=10

auto vlan18i
iface vlan18i inet static
	address 10.18.0.1/24
	ovs_type OVSIntPort
	ovs_bridge vmbr0
	ovs_mtu 9000
	ovs_options tag=18

auto outside
iface outside inet static
	address 194.141.32.111/24
	gateway 194.141.32.1
	ovs_type OVSIntPort
	ovs_mtu 9000
	ovs_bridge vmbr0

auto vlan6i
iface vlan6i inet static
	address 10.6.0.1/24
	ovs_type OVSIntPort
	ovs_bridge vmbr0
	ovs_mtu 9000
	ovs_extra set interface ${IFACE} external-ids:iface-id=$(hostname -s)

auto vmbr0
iface vmbr0 inet manual
	ovs_type OVSBridge
	ovs_ports ens1f0 vlan7i vlan8i vlan10i vlan18i outside vlan6i
	ovs_mtu 9000
#Web Management

source /etc/network/interfaces.d/*
```
Additional iptabbles rules
```
iptables -t nat -A POSTROUTING -o vmbr0 -j MASQUERADE
iptables -t nat -A POSTROUTING -j MASQUERADE
```
![](/scripts/images/pmx_network.png)
## Virtual machines hardware
![](/scripts/images/pmx_hardware.png)

