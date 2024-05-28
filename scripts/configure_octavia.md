# Configure Octavia loadbalancer service
## Configure Neutron ML2 security
```
juju config neutron-api enable-ml2-port-security=True
```

In order to configure Octavia with its own certificates and password ```foobar``` execute the following commands:

## Create Octavia keys
```
mkdir -p certs && cd certs
mkdir -p demoCA/newcerts
touch demoCA/index.txt
touch demoCA/index.txt.attr

openssl genpkey -algorithm RSA -aes256 -pass pass:foobar -out issuing_ca_key.pem
openssl req -x509 -passin pass:foobar -new -nodes -key issuing_ca_key.pem \
    -config /etc/ssl/openssl.cnf \
    -subj "/C=US/ST=Somestate/O=Org/CN=www.example.com" \
    -days 365 \
    -out issuing_ca.pem

openssl genpkey -algorithm RSA -aes256 -pass pass:foobar -out controller_ca_key.pem
openssl req -x509 -passin pass:foobar -new -nodes \
        -key controller_ca_key.pem \
    -config /etc/ssl/openssl.cnf \
    -subj "/C=US/ST=Somestate/O=Org/CN=www.example.com" \
    -days 365 \
    -out controller_ca.pem
openssl req \
    -newkey rsa:2048 -nodes -keyout controller_key.pem \
    -subj "/C=US/ST=Somestate/O=Org/CN=www.example.com" \
    -out controller.csr
openssl ca -passin pass:foobar -config /etc/ssl/openssl.cnf \
    -cert controller_ca.pem -keyfile controller_ca_key.pem \
    -create_serial -batch \
    -in controller.csr -days 365 -out controller_cert.pem
cat controller_cert.pem controller_key.pem > controller_cert_bundle.pem
```
### Configure juju charm
```
juju config octavia \
    lb-mgmt-issuing-cacert="$(base64 issuing_ca.pem)" \
    lb-mgmt-issuing-ca-private-key="$(base64 issuing_ca_key.pem)" \
    lb-mgmt-issuing-ca-key-passphrase=foobar \
    lb-mgmt-controller-cacert="$(base64 controller_ca.pem)" \
    lb-mgmt-controller-cert="$(base64 controller_cert_bundle.pem)"
```
## Configure resources
**NB!!!** Patiently wait untill ```watch -c juju status octavia --color``` became active, idle and the "Unit is ready"
```
juju run octavia/0 configure-resources
```
## Create Amphorae image for Octavia loadbalancers
```
juju run glance-simplestreams-sync/0 sync-images
```
Watch juju satus and wait until ```glance-simplestreams-sync``` is finished. Then create Octavia Amphorae image.
```
juju run octavia-diskimage-retrofit/leader retrofit-image
```
When juju status states that the task is finished you are ready.
OpenStack is installed in base version with loadbalancer Octavia added.
