## Install Ubuntu 22.04
## Install snapd
```
apt install -y snapd 
```
## Install maas through snap applications
```
snap install maas-test-db
snap install maas
```
## Install juju
```
snap install juju
```
## Install other applications
```
snap install vault  ## The OpenStack vault agent
snap install kubectl --classic 
snap install openstackclients --classic
snap install terraform --classic
```
## Initialise the MAAS Controller
```
maas init region+rack --maas-url http://10.6.0.10:5240/MAAS --database-uri maas-test-db:///
```
## Create admin user and credential
```
maas createadmin --username admin --password YJW59c+FK7h/2dQW240kBg== --email gpenchev@e-dnrs.org 
maas apikey --username admin > ~/admin-api-key
cat ~/admin-api-key
```
### Result for MAAS api-key 
```
qjtBFhH5JJxkjP9Lhy:JK4h8Vru3Bk7aWAb8n:AjrEdTkzEgEJAVRYc2qaruBdvXPb3qx7
```
## Register Juju Cloud and Controller
```
cat << EOF >  maas-cloud.yaml
clouds:
  maas-one:
    type: maas
    auth-types: [oauth1]
    endpoint: http://10.6.0.10:5240/MAAS
EOF
```
```
juju add-cloud --client -f maas-cloud.yaml maas-one
```
## Add credential for Juju to MAAS
```
cat << EOF >  maas-creds.yaml
credentials:	
  maas-one:
    anyuser:
      auth-type: oauth1
      maas-oauth: qjtBFhH5JJxkjP9Lhy:JK4h8Vru3Bk7aWAb8n:AjrEdTkzEgEJAVRYc2qaruBdvXPb3qx7
EOF
```
```
juju add-credential --client -f maas-creds.yaml maas-one
```

########### CONFIGURE MAAS ###############################

## Bootrap JUJU
```
juju bootstrap --bootstrap-series=jammy --constraints tags=juju maas-one maas-controller
```

