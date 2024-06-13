# Create KYPO kubernetes resources &mdash; kypo-head-services
```
cd /opt/kypo-crp-tf-deployment/tf-head-services/
```
Copy file with variables for install. Look [here](https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-crp-tf-deployment/-/blob/master/HELM.md) and [here](https://gitlab.ics.muni.cz/muni-kypo-crp/devops/kypo-crp-tf-deployment/-/tree/master/tf-openstack-base) for variables exlanation. 
```
cp tfvars/deployment.tfvars-template tfvars/deployment.tfvars
```

Prepare the file with your passwords. Copy the keys and sertificates for KYPO kubernetes from previous stage of installing the Openstack KYPO's resources. Decide whether to use own GITLAB repo or local github.
**NB:** Prepare your FDQN record for the KYPO in your registrar or dns proxy (if you using one). If you prefer not to use FDQN you can use kubernetes IP as a `head_host`.

Ready file:
```
acme_contact                  = "gpenchev@gmail.com"
application_credential_id     = "8b2b28edcca64d16a6617780ff1df696"
application_credential_secret = "clzl5yUcPLJh2fd2eYGtuS8bkxpMPQdlqc5qh4lRyZuTtSyH2yuc8nOZaG_WR0wIO3vpj70UW2W4SITXKuzGcw"
enable_monitoring             = true
gen_user_count                = "2"
git_config = {
     type                 = "INTERNAL"
     server               = "git-internal.kypo"
     sshPort              = 22
     restServerUrl        = "http://git-internal.kypo:5000/"
     user                 = "git"
     privateKey           = ""
     accessToken          = "no-gitlab-token"
     ansibleNetworkingUrl = "git@git-internal.kypo:/repos/backend-python/ansible-networking-stage/kypo-ansible-stage-one.git"
     ansibleNetworkingRev = "v1.0.14"
}
guacamole_admin_password      = "5YDBzE2SlpM7k9nnevsqig=="
guacamole_user_password       = "Ut9ghn7NjaW5VUUhuuH4rA=="
head_host                     = "fcr.e-dnrs.org"
head_ip                       = "10.18.0.162"
kubernetes_host               = "10.18.0.162"
kubernetes_client_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIyekNDQVdLZ0F3SUJBZ0lSQUtFTDJRZ3Uwa2FIZ1B3dGgvdFlHc013Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBM>
kubernetes_client_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRBc2hMZWIwZFdkRVZRa0ZwcDBWODVBYWRzelVIdkl0c3pUeTd0eU9nNkNkOUJ4VFNVc2QwbFAKR3NHWTJ1cURmMWVnQndZRks0RUVBQ0toWkFOaUFBU2JpRUpFMnZ0anFoT>
kypo_crp_head_version         = "3.1.11"
kypo_gen_users_version        = "2.0.1"
kypo_postgres_version         = "2.1.0"
man_image                     = "debian-11-man-preinstalled"
os_auth_url                   = "https://10.6.0.52:5000/v3"
openid_configuration_insecure = true
proxy_host                    = "10.18.0.165"
proxy_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRDU0MxdXNmektEb1NLZ01jaEJFakFFbXM4VWZwekFlVkRaeXFkdEVoVFlDaTg2Yzh6Yml4NUYKbTlPL0xCNnRIUUtnQndZRks0RUVBQ0toWkFOaUFBUitqTWwzUmZqVi91Nk5GWjJxbE8rM>
users = {
    "kypo-admin" = {
      iss              = "https://fcr.e-dnrs.org/keycloak/realms/KYPO"
      keycloakUsername = "kypo-unwe"
      keycloakPassword = "pass123456"
      email            = "admin@example.com"
      fullName         = "Demo Admin"
      givenName        = "Demo"
      familyName       = "Admin"
      admin            = true
    }
}
```
Then init terraform for KYPO head sevices.
```
terraform init
```
And apply the configuration.
```
terraform apply -var-file tfvars/deployment.tfvars --auto-approve
```
Watch progress in another terminal ...
```
kubectl get pod -A -w
```
... or ...
```
watch kubectl get pod -A
```
... or watch events in cluster ...
```
kubectl get events -A -w
```
## Get access and add certificates
### Keycloak and Grafana (monitoring)
Get *Keycloak* and *Grafana* passwords.
```
terraform output --json
```
the results:
```
{
  "keycloak_password": {
    "sensitive": true,
    "type": "string",
    "value": "kPLSzv5mSNWGBgCtfVIU"
  },
  "monitoring_admin_password": {
    "sensitive": true,
    "type": "string",
    "value": "yWuC5mJOeAoNu8Bzxpun"
  }
}
```
Open *Keykloak* with following address [https://fcr.e-dnrs.org/keycloak/](https://fcr.e-dnrs.org/keycloak/) and log in with `admin` user.
You can edit users and other staff in regard to KYPO and Keykloack.
Monitoring of the cyber range can be found on the address [https://fcr.e-dnrs.org/grafana/](https://fcr.e-dnrs.org/grafana/). Log in with user `admin`.

### Add certificates
**NB:** This is a workarround. Not stable solution. If you need to stop or to restart the KYPO kubernetes, then you need do the same again.
KYPO do not use certificates for authentication to the OpenStack, so you need to add OpenStack client certificate to sandbox-service.
Find the name of the pod
```
kubectl get pod -A | grep sandbox-service
```
Result:
```
kypo           sandbox-service-6dc4bff4dc-rt8s9                         1/1     Running            0               68m
kypo           sandbox-service-worker-ansible-5d44dbc8cc-4n7g5          1/1     Running            0               50m
kypo           sandbox-service-worker-default-785c5b9bf8-rz9wr          1/1     Running            0               50m
kypo           sandbox-service-worker-openstack-86d46cf46f-h6sfk        1/1     Running            0               50m
kypo           sandbox-service-worker-openstack-86d46cf46f-9vc7l        1/1     Running            0               50m
kypo           sandbox-service-worker-ansible-5d44dbc8cc-6kx2v          1/1     Running            0               50m
kypo           sandbox-service-worker-default-785c5b9bf8-ndxdq          1/1     Running            0               50m
kypo           sandbox-service-worker-ansible-5d44dbc8cc-dt729          1/1     Running            0               49m
kypo           sandbox-service-worker-default-785c5b9bf8-sb2hq          1/1     Running            0               49m
kypo           sandbox-service-worker-openstack-86d46cf46f-nd6tb        1/1     Running            0               49m
kypo           sandbox-service-worker-ansible-5d44dbc8cc-7nnhf          1/1     Running            0               49m
kypo           sandbox-service-worker-default-785c5b9bf8-lszw8          1/1     Running            0               49m
kypo           sandbox-service-worker-openstack-86d46cf46f-dhndr        1/1     Running            0               49m
```
Copy certificates from the pod.
```
kubectl cp sandbox-service-6dc4bff4dc-rt8s9:/etc/ssl/certs/ca-certificates.crt ~/ca-certificates.crt
```
Add Openstack client certificate.
```
cat /root/snap/openstackclients/common/root-ca.crt >> ~/ca-certificates.crt
```
Add new certicate to the sandbox-service pod.
```
kubectl cp ~/ca-certificates.crt sandbox-service-6dc4bff4dc-rt8s9:/etc/ssl/certs/ca-certificates.crt
```
### Change type of the connection between KYPO's Ansible scripts and Openstack
**NB:** This is a workarround. Not stable solution. If you need to stop or to restart the KYPO kubernetes, then you need do the same again.
KYPO uses Ansible for pool creation and these applications do not have access to the Openstack resources. So, you need to setup insecure connection by adding new environment variable.
```
kubectl set env deployment/sandbox-service-worker-ansible OS_INSECURE=true
kubectl set env deployment/sandbox-service-worker-default OS_INSECURE=true
kubectl set env deployment/sandbox-service-worker-openstack OS_INSECURE=true
```

Now you are **ready to use the cyber range KYPO**!
