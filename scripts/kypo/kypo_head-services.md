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
acme_contact                  = "gpenchev@gpenchev@gmail.com"
application_credential_id     = "e9c87d6055ac46b29c749b6b308550c9"
application_credential_secret = "clzl5yUcPLJh2fd2eYGtuS8bkxpMPQdlqc5qh4lRyZuTtSyH2yuc8nOZaG_WR0wIO3vpj70UW2W4SITXKuzGcw"
enable_monitoring             = true
gen_user_count                = "8"
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
  # Example of GitLab
  #   type                 = "GITLAB"
  #   server               = "example.com"
  #   sshPort              = 22
  #   restServerUrl        = "https://example.com/"
  #   user                 = "git"
  #   privateKey           = "`base64 -w0 ssh_private.key`"
  #   accessToken          = "alpha-num-string"
  #   ansibleNetworkingUrl = "git@example.com:kypo-ansible-stage-one.git"
  #   ansibleNetworkingRev = "master"
}
guacamole_admin_password      = "5YDBzE2SlpM7k9nnevsqig=="
guacamole_user_password       = "Ut9ghn7NjaW5VUUhuuH4rA=="
head_host                     = "fcr.e-dnrs.org"
head_ip                       = "10.18.0.235"
kubernetes_host               = "10.18.0.235"
kubernetes_client_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIzRENDQVdLZ0F3SUJBZ0lSQU9oT3R5Y2FPUFJkdTVZanQ1ajdvOWN3Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBMk1EWXdOelEzTWpWYUZ3MHpOREEyTURZeApPVFEzTWpWYU1ESXhGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1SY3dGUVlEVlFRREV3NTBaWEp5CllXWnZjbTB0ZFhObGNqQjJNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQWlBMklBQkV0Rkx5c1dKWlZ5QVlzWmFIWloKMFFJVWx2cEJGclNvRCt6MUdtUkZVUS9MQ1oveWE1Z3BOSHpmVmRwaTRsNWsrbGpwSlF2YmJBWkhHVEFIV29CYQozYklEUFB2ZnY3bHdyWVhtZFhiK015RDc5VTVDUVdyalpFa05HOVBpNE9HNTVxTldNRlF3RGdZRFZSMFBBUUgvCkJBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGoKQkJnd0ZvQVV0YnU2Qmt6VmkzN0hOdy9KdkhIeVZVUnArbXd3Q2dZSUtvWkl6ajBFQXdNRGFBQXdaUUl4QU11ego1aU5aOWxTME5MbnpYNGdycVdHQXdEdWh5YysyOTRFVWxUNUUvOHBzZE5XbHpjSEh5b3hPb2JXSmdUK0FMQUl3CkpCaGVnZWJDajZDUWpVSU1HdFdWbCtpRHVaTzhJUEhNb2daMkNpaUhSQmU3SEViZG1QUHN5V3Y4eFQ2cEFYVHMKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
kubernetes_client_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRDL1hHMUR0V0M4VklwejNiSExPSGl6UU91RFh1UDZFMjNydWZNMDc2SnlHSXRRdERvYXE3aFQKRko2QTFQZEl1Ym1nQndZRks0RUVBQ0toWkFOaUFBUkxSUzhyRmlXVmNnR0xHV2gyV2RFQ0ZKYjZRUmEwcUEvcwo5UnBrUlZFUHl3bWY4bXVZS1RSODMxWGFZdUplWlBwWTZTVUwyMndHUnhrd0IxcUFXdDJ5QXp6NzM3KzVjSzJGCjVuVjIvak1nKy9WT1FrRnE0MlJKRFJ2VDR1RGh1ZVk9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
kypo_crp_head_version         = "3.1.11"
kypo_gen_users_version        = "2.0.1"
kypo_postgres_version         = "2.1.0"
man_image                     = "debian-11-man-preinstalled"
os_auth_url                   = "https://10.6.0.52:5000/v3"
openid_configuration_insecure = true
proxy_host                    = "10.18.0.43"
proxy_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRDelMwYXdEU3czRjFJN0I3d2tYOWJ1VnVYUG9tWE4xZEhSSDlZUXExWGNVWk1FTUNuQlVpaTEKYXdacmIrQVorVU9nQndZRks0RUVBQ0toWkFOaUFBVHZNdkRDRWRoRnVUdFBISHZGWmM3QVNFZTQwM1FMRkNDRwp3YjdKank2N1lJOEUxaUhUM1VkWEVDdkNyS2c4N2dqT1N1UFRid1JtQnRhUk5TT2dSQXpWL2xoRkZQZjN2cURhCml2WkxXZDc0eS9PWXJnY1NCUVhLOWVLMVhacHdIck09Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
users = {
    "kypo-admin" = {
      iss              = "https://fcr.e-dnrs.org/keycloak/realms/KYPO"
      keycloakUsername = "kypo-admin"
      keycloakPassword = "pass123456"
      email            = "admin@example.com"
      fullName         = "Demo Admin"
      givenName        = "Demo"
      familyName       = "Admin"
      admin            = true
    }
}
```
