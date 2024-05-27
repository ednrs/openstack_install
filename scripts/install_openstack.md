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
**NB:!!!** Save all vault tokens and other results and do not lose them! Otherwise you will be not able to start OpenStack after shutdown, restart or outage.

When ```nova-cloud-controller``` and ```keystone``` became active and idle unseal the wault with following commands:

Set address of the vault as environment variable (get the address of the vault from ```juju status```
```
export VAULT_ADDR="http://10.6.0.25:8200"
```
Initiate vault with five keys (three main and two reserve eys)
```
vault operator init -key-shares=5 -key-threshold=3
```
Use three of them with command ```vault operator unseal```
```
vault operator unseal Q70dkoSAxW/Sko6kOhSJVRGfenEFGdFIvu/6LooYNQaT
vault operator unseal 1SUIidMByFjV8m/6MJRcCpPKxvJkb35fxDBuj3fZHBSb
vault operator unseal J+73FQD55he0wy8rd7qWBnTtqeDbOiArNGujhCrW3Amt
```
... and save the other two and the ```Initial Root Token```
```
Unseal Key 4: zDgrc23uQaw7D7hzmda1AT/+7wdXwR4GFVCsfd98+2zU
Unseal Key 5: ET0eVCbslSg8OfgtBSFkF3744GYLt8me0K5T/kp9cTCU

Initial Root Token: s.IwKiZaABJhKZDLpaRhOTXioo
```
Export the ```Initial Root Token```
```
export VAULT_TOKEN=s.IwKiZaABJhKZDLpaRhOTXioo
```
Then create the root token. 
```
vault token create -ttl=20m
```
And save the resuts:
```
token                s.1LyBALR4tms9h6NYJjHXDNfw
token_accessor       2XRftrvnNHdw794YY6Y7HO5d
```
Finally, authorise the vault charm with the token ...
```
juju run vault/leader authorize-charm token=s.1LyBALR4tms9h6NYJjHXDNfw
```
... and generate root certificate for the Openstack
```
juju run vault/leader generate-root-ca
```
Save the results and wait untill **ALL** services shown in ```juju status``` became active and idle (except ```octavia``` and ```glance-simplestreams-sync```)
```
## Set Address and Port
### Ge it from juju status
export VAULT_ADDR="http://10.6.0.25:8200"

## Initialise
vault operator init -key-shares=5 -key-threshold=3

## Keys use
export VAULT_ADDR="http://10.6.0.13:8200"
vault operator unseal Q70dkoSAxW/Sko6kOhSJVRGfenEFGdFIvu/6LooYNQaT
vault operator unseal 1SUIidMByFjV8m/6MJRcCpPKxvJkb35fxDBuj3fZHBSb
vault operator unseal J+73FQD55he0wy8rd7qWBnTtqeDbOiArNGujhCrW3Amt

Unseal Key 4: zDgrc23uQaw7D7hzmda1AT/+7wdXwR4GFVCsfd98+2zU
Unseal Key 5: ET0eVCbslSg8OfgtBSFkF3744GYLt8me0K5T/kp9cTCU

Initial Root Token: s.IwKiZaABJhKZDLpaRhOTXioo

## Generate a root token with a limited lifetime (10 minutes here) using the initial root token:
export VAULT_TOKEN=s.IwKiZaABJhKZDLpaRhOTXioo
vault token create -ttl=20m

token                s.1LyBALR4tms9h6NYJjHXDNfw
token_accessor       2XRftrvnNHdw794YY6Y7HO5d

## Authorise the vault charm
juju run vault/leader authorize-charm token=s.1LyBALR4tms9h6NYJjHXDNfw

## Add a CA certificate
juju run vault/leader generate-root-ca

output: |-
  -----BEGIN CERTIFICATE-----
  MIIDazCCAlOgAwIBAgIUUCngwSqISGxyyHXSbB0jabfsaXwwDQYJKoZIhvcNAQEL
  BQAwPTE7MDkGA1UEAxMyVmF1bHQgUm9vdCBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkg
  KGNoYXJtLXBraS1sb2NhbCkwHhcNMjQwNTIzMTYwMDIzWhcNMzQwNTIxMTUwMDUz
  WjA9MTswOQYDVQQDEzJWYXVsdCBSb290IENlcnRpZmljYXRlIEF1dGhvcml0eSAo
  Y2hhcm0tcGtpLWxvY2FsKTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
  AKGqf+/3Xe+58Myw+Op8kihG7LoWgPxrUAZY4mF9/qjl3VQ017Lh/lJJAMevo31Q
  vviIhmVyCEyw3oYHEqwHNRaZCvMSFYg765yaFGViTtnsZVroIcbakcDXcMa+MNE6
  ECZ56OiVLmIPQxeySwiVg8T9RI7mM+Jf1QLO3ASFyhMDhp62VLGsSU6GpRLpB4oc
  foFdMzr7gZlBjwR0ej45wS+NzqJRfe778gg1SvFqSvQiaMuerho0f7ifDc5M91lp
  GfRdF85mOAPM92EFMSiRkidQz9D0UXS/pLKMHMeGxDn1t/EZ2wW2wimmeZRXamph
  dFN7l7EIWmnueF189tfEflkCAwEAAaNjMGEwDgYDVR0PAQH/BAQDAgEGMA8GA1Ud
  EwEB/wQFMAMBAf8wHQYDVR0OBBYEFCK3eNHFCsY47BYritqObWitBBtiMB8GA1Ud
  IwQYMBaAFCK3eNHFCsY47BYritqObWitBBtiMA0GCSqGSIb3DQEBCwUAA4IBAQAS
  1RfjRaaXGHF/fSGstjn5yjlmsK8GpTUj3C3mbxIUBpbJCeR/ohcLgsMh6mRW/vSJ
  GiM2MvhXdFE5pSYUOA0chud8i8a+KTRcO+2Ghc1K11rUxcXCUnznV8enl8rqeRms
  h5L4BIPBgfqM6w/eFyhbmQbjFiCyek57jFw495TJkzwxOIw74wsuZPfAvp9Nkl44
  qT/rce1saA5xF4ooSqbvSlaszUnaFSX0h8QSTlRYZb9QQaFtLcCjV98wQqdD8Hjv
  eRQUFrkRr3TfvG3wfr8/nEUi3nF6v5uAnB2KD6j3ThffC1YTy0vhsssnWDU3Vz+T
  nIM+ncT2iEZjpO4xRBvS
  -----END CERTIFICATE-----
```
