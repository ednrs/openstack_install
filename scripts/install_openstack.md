# Install Openstack with bundle and overlay

## Boostrap JUJU
```
juju bootstrap --bootstrap-series=jammy --constraints tags=juju maas-one maas-controller
```
### Add model with specification
```
juju add-model --config default-series=jammy --config default-space=public openstack
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
**NB:!!!** Save all vault tokens and other results and do not lose them! Otherwise you will be not able to start OpenStack after shutdown, restart or outage.

When ```nova-cloud-controller``` and ```keystone``` became active and idle unseal the vault with following commands:

Set address of the vault as environment variable (get the address of the vault from ```juju status```
```
export VAULT_ADDR="http://10.6.0.41:8200"
```
Initiate vault with five keys (three main and two backup keys)
```
vault operator init -key-shares=5 -key-threshold=3
```
Use three of them with command ```vault operator unseal```
```
vault operator unseal Vf8aA7BB/YQ82byasAriQt5wfadXtCa32uJRZiVojnx0
vault operator unseal HqX2dD5+dg9YvsJrkZ83vwUZY8HPjxNo1HtM1eputmQf
vault operator unseal J1RK74jMYw9XuO0aLwzOwirGoGQnM1r88Z5dri1GAgyg
```
... and save the other two and the ```Initial Root Token```
```
Unseal Key 4: vUFb9LkU5PzM2sEIevsI181kIwhRoYowdwFOK4VsetOk
Unseal Key 5: Ty1YThrl8RzeAkrAP08WI0oPBfnZgat31Aqh11YI9O1R

Initial Root Token: s.9O0GbH389QL2YaHtH3FMZz5d
```
Export the ```Initial Root Token```
```
export VAULT_TOKEN=s.9O0GbH389QL2YaHtH3FMZz5d
```
Then create the root token. 
```
vault token create -ttl=20m
```
And save the resuts:
```
token                s.zzIbSAi5U9iE860GYYGOEmk3
token_accessor       Qqj28WACPPgRgrGXXc5lMhdd
```
Finally, authorise the vault charm with the token ...
```
juju run vault/leader authorize-charm token=s.zzIbSAi5U9iE860GYYGOEmk3
```
... and generate root certificate for the Openstack
```
juju run vault/leader generate-root-ca
```
Save the results and wait untill **ALL** services shown in ```juju status``` became active and idle (except ```octavia``` and ```glance-simplestreams-sync```)
```
output: |-
  -----BEGIN CERTIFICATE-----
  MIIDazCCAlOgAwIBAgIUHdNtS3Jx8B8eMhCqnrJF3Aj/GmEwDQYJKoZIhvcNAQEL
  BQAwPTE7MDkGA1UEAxMyVmF1bHQgUm9vdCBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkg
  KGNoYXJtLXBraS1sb2NhbCkwHhcNMjQwNTMwMTkwODI5WhcNMzQwNTI4MTgwODU5
  WjA9MTswOQYDVQQDEzJWYXVsdCBSb290IENlcnRpZmljYXRlIEF1dGhvcml0eSAo
  Y2hhcm0tcGtpLWxvY2FsKTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
  ALzyeyGAufiBRHGCllTDZH0htd6SXA3L6HDewmH3mxje+Sx3pv/Y9RkTpMqFLi1t
  mVS8EU5Mrdx6/VIn5hwBDU3JxVvLwlBX25uXqK1MceLEb51+7frqLtROJWQ6DnJd
  zikmYlayP0eIXNkTYk44iApCfK6CZ0N0GX0NAwCBb5PLf8R1SDoevEPMBcECqJpy
  OACcLXGfdAdkjn53xULv/MwbhebQwaG/iG0gDmYuPcm5FPmvx98Yj+VWpNtmdEvz
  Fhvh9+WdITuEYOoMXTcOXGPAaz7e8hLFpDoXUI7O6ZCTWWQHoj5Y5fSwXn7fCT8O
  thm/HByvBBMy9q2Q2fY/i4UCAwEAAaNjMGEwDgYDVR0PAQH/BAQDAgEGMA8GA1Ud
  EwEB/wQFMAMBAf8wHQYDVR0OBBYEFKMr+FiA/dN6b5E6j/s1f5riIsN/MB8GA1Ud
  IwQYMBaAFKMr+FiA/dN6b5E6j/s1f5riIsN/MA0GCSqGSIb3DQEBCwUAA4IBAQAP
  myJMq5RiORteVZdKnKrolBpzlmhWqjyZSFDLNChiWc5HHjh34vePJ2TjKxAOwprX
  3U8MX7yQwzfplv1oBMXAVrisJvK6Ey3Bpx/wZ7NqF03AWBPk5iaJi4p5/8m5BRi0
  kCBjiyY/kQBwO3WNo282nhiS/0UNsjqzfX26mWFTgVuObIkck91k47F7QWfklUXv
  BDiH6XgxJOX4oxwiO5Fla2xQGr2+Hm8M09IP9+Jz8DwAOQBSc4Hw1rbQjiMMvBmS
  8+WCzDoUB7s/XGpD9zY18IB2sXRIbyw/omyppbQT0+ofHRnmUOJkaQuVRJ8bC5BU
  +iB9TtnJ9+ljmpTyqOIo
  -----END CERTIFICATE-----
```
