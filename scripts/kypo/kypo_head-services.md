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
head_ip                       = "10.18.0.224"
kubernetes_host               = "10.18.0.224"
kubernetes_client_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIzRENDQVdLZ0F3SUJBZ0lSQUs4dktwaTRqN1NHRVFSNnh4NTVON2t3Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBMk1Ea3hNREkwTkRCYUZ3MHpOREEyTURreQpNakkwTkRCYU1ESXhGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1SY3dGUVlEVlFRREV3NTBaWEp5CllXWnZjbTB0ZFhObGNqQjJNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQWlBMklBQkdvM2U4SVB3ZHFrZ0J5U2VQTU8KRGRTTHRHVjM5ODhSaFdTbWd6ckRHWmptUnZMdmI0NVJjSTR0YzBZczhmbWkwTlExcDMwUkYreGxDbEgyNnpOQwptYnlnNHNIRm1rckFzVittR3E4TzJKREUzQVMxSFMwY0hIRDJzc2VnWUNZQ1NhTldNRlF3RGdZRFZSMFBBUUgvCkJBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGoKQkJnd0ZvQVVQSTFoZFROZHJLbXdlVVRHZlE1TnFmMlppVUF3Q2dZSUtvWkl6ajBFQXdNRGFBQXdaUUl4QUtVWAoyclZIWUkwdWx2a3dEMWZIRnEvQVBRcHR5WDhJMkM1dFJhSnVkSFQ1TW9ockxlRU80djZMZ2VETC9iUDJLZ0l3ClpXOXJUZVdGRnN1NC95TCtSQldEWW5wZThTVld0eDhybXRFSHh1Z0FpbGMzWDBLRGp0Y2hBNGtSa0kvZjFSVWsKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
kubernetes_client_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRDNEdKcUhacjVlZDVlcXl1NmdiQzJBRE9la0xZVVpZWHByek0xMWNmbVRQT3ZYVzhCVmhBRC8KZUk3ODR6ODZKb1NnQndZRks0RUVBQ0toWkFOaUFBUnFOM3ZDRDhIYXBJQWNrbmp6RGczVWk3UmxkL2ZQRVlWawpwb002d3htWTVrYnk3MitPVVhDT0xYTkdMUEg1b3REVU5hZDlFUmZzWlFwUjl1c3pRcG04b09MQnhacEt3TEZmCnBocXZEdGlReE53RXRSMHRIQnh3OXJMSG9HQW1Ba2s9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
kypo_crp_head_version         = "3.1.11"
kypo_gen_users_version        = "2.0.1"
kypo_postgres_version         = "2.1.0"
man_image                     = "debian-11-man-preinstalled"
os_auth_url                   = "https://10.6.0.52:5000/v3"
openid_configuration_insecure = true
proxy_host                    = "10.18.0.51"
proxy_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRBTGZzOStyL01IZ2RjV2FaRVNuYkMrdklzRkh1Mzh6M0dvVVRsdVpkUGVCeXVXbysvQXd6emQKbWp4TTFkODNJeEdnQndZRks0RUVBQ0toWkFOaUFBUmVXektoeGtaYk01L2JFbzB6OEdvK3RONW10c2k3ZXgxOApuOXlraEZWcnVVYlBIdk44YWZZQjhFVjY0TzBWTm9ZOSs5cm5BZlM2WVF6STBEMjRoTGhaVDN0c0lYVDkwRnU2CkRzWFpRaTRzZXIydk5MU21JSVpGdC9Ec3RDSHFjNEk9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
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
