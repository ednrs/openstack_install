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
export VAULT_ADDR="http://10.6.0.12:8200"
```
Initiate vault with five keys (three main and two backup keys)
```
vault operator init -key-shares=5 -key-threshold=3
```
Use three of them with command ```vault operator unseal```
```
vault operator unseal tPKaXkJuGTvVA3miVgflNTkmgKuXNQ5HGYkMchHf+ahE
vault operator unseal vhaM7JI/cf/irV/G3CyoaeeiIQ7B/0Z/iWkwJ+0xhOMT
vault operator unseal /2cXbxO1HCi/2zgALUZKWkY3nEQEudp/Vt6GYg1Dgm5V
```
... and save the other two and the ```Initial Root Token```
```
Unseal Key 4: G/6MvoI6lW8RpE/IWiRMCJqvBe1lWmmuPWDkl7KikBLz
Unseal Key 5: 3HIGGEvjuNfw/HZEHyY9u5LcbCUanJDZAawrkyiqtLVi

Initial Root Token: s.S4pALZuoq8GjiAniSaWwvU0s
```
Export the ```Initial Root Token```
```
export VAULT_TOKEN=s.S4pALZuoq8GjiAniSaWwvU0s
```
Then create the root token. 
```
vault token create -ttl=20m
```
And save the resuts:
```
token                s.rWp9SY21DuLIIKA219FseWo0
token_accessor       2GsYhO8LrQP6bVGtvP2Yopyi
```
Finally, authorise the vault charm with the token ...
```
juju run vault/leader authorize-charm token=s.rWp9SY21DuLIIKA219FseWo0
```
... and generate root certificate for the Openstack
```
juju run vault/leader generate-root-ca
```
Save the results and wait untill **ALL** services shown in ```juju status``` became active and idle (except ```octavia``` and ```glance-simplestreams-sync```)
```
output: |-
  -----BEGIN CERTIFICATE-----
  MIIDazCCAlOgAwIBAgIUPEvn8wYpCroAnFm7QL4K2/y4YgEwDQYJKoZIhvcNAQEL
  BQAwPTE7MDkGA1UEAxMyVmF1bHQgUm9vdCBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkg
  KGNoYXJtLXBraS1sb2NhbCkwHhcNMjQwNTI4MTkzNzAwWhcNMzQwNTI2MTgzNzMw
  WjA9MTswOQYDVQQDEzJWYXVsdCBSb290IENlcnRpZmljYXRlIEF1dGhvcml0eSAo
  Y2hhcm0tcGtpLWxvY2FsKTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
  ANz5CKIHxG3kkW29PDVF0doAq+3QzuBG1u2qxufPPqgZXiG85s0U71UZXzuFZxox
  RG3mi7kP/hjlEo+oi0JQAXJmjiDdSLPoVkTpwtfM0q+zcnWj0tAhni1PR8gInam3
  LHyMHUaEn2QBslCR+zdFyx3pw3vbCi7IPKzBtuOxIEVpoAdhlL0BYgOQfixYxaM5
  Y18aBNozfjzAVJTN8id7zpZIe76uF+xRB/+dzWBFgT7N+MoVORc2INIY/ylJTB4M
  fq2sLNtYZV27uTEggIo92RZS4pL7CKp1NSDJNguE93cJXWwDiYsGNLOLU8Z84LKI
  GdGyPdHC1/+agS2LSJd21oMCAwEAAaNjMGEwDgYDVR0PAQH/BAQDAgEGMA8GA1Ud
  EwEB/wQFMAMBAf8wHQYDVR0OBBYEFHJzDeCDG56+ejFOcKJmA3cByONjMB8GA1Ud
  IwQYMBaAFHJzDeCDG56+ejFOcKJmA3cByONjMA0GCSqGSIb3DQEBCwUAA4IBAQCl
  n5ueryNcMVxgD8VuClFXo6L7HslG0N36/3jxzjfNGTFHQnh68pUFUAFcDjjZpuJw
  fbsfTv6ba5J0BvwFrZuKmbPoNblDT5az5HaHcMLqDhw4AZgYP9Bo71x+eEi7e8C2
  4AgY0K/QQk9DjOUsaz7B5iXBsZyyaSOPY/own48vnIgHfURycXS6uV0oLrpkQaPW
  4w56oj4WYgCkkLvukaEAunt3VDbFBAMmYeMys4z8SXeJGfHzoZUDrXjvityqeV/s
  /mlvSzpVN/iD/AZyJ4x9Eu9/XffcP+VPlii+jvs+KrOtj7+aup5IDtpIhKRdsuPZ
  1IsJ1dFvEDisa3dtxXFn
  -----END CERTIFICATE-----
```
