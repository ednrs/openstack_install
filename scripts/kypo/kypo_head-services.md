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
}
guacamole_admin_password      = "5YDBzE2SlpM7k9nnevsqig=="
guacamole_user_password       = "Ut9ghn7NjaW5VUUhuuH4rA=="
head_host                     = "fcr.e-dnrs.org"
head_ip                       = "10.18.0.47"
kubernetes_host               = "10.18.0.47"
kubernetes_client_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIzRENDQVdLZ0F3SUJBZ0lSQU1RSDNJTDZSUmNaZFNrZlh3Y2xvakl3Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBMk1EWXhPREF5TkRkYUZ3MHpOREEyTURjdwpOakF5TkRkYU1ESXhGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1SY3dGUVlEVlFRREV3NTBaWEp5CllXWnZjbTB0ZFhObGNqQjJNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQWlBMklBQkU1SzdtZkNNREY2TURwTk1Oc3IKZWpxVWRkc2d5dW9jOVRKQ29DTDFWNmlQY3h4cmlaNHhYc1VDTHo3YUt4dG9EWHJPWW1MSjAybTRtZWQzTzJtQwpyV3FzSytFby80RE1TRlpBQ2JycXhHSVY4MXVjaDdYYjZXbjRhWjNyeWRZZDRLTldNRlF3RGdZRFZSMFBBUUgvCkJBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGoKQkJnd0ZvQVVTUTNRZW5jNzRYcDlBd2pBV1BocmpsNm9IS013Q2dZSUtvWkl6ajBFQXdNRGFBQXdaUUl4QU5BdgplNXdZdFRHZHoxSVRwL0NJcld2WXluTTJ5UUh4MGFFUUxrMHg5NWh1aU43MnhLeFlQQ1BEQmkxQTR4Q1g5QUl3CkR3WDhVRmpTNk12Ly9EL1Q2ajVEU3FQVHlYaGtNVVlTRlBYNUpXZVZWcWR6Nk13THpHY1MweEgzcTZJNVRvWGIKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
kubernetes_client_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkREYmNVUGRMb2VzRHZIY3ZrL2VHZFVaRTl4TlNsUVg2QXloUlJ2T0pFeEE1SWQ2b2ZzNm9kVEYKVldQSzhEc0d5UVNnQndZRks0RUVBQ0toWkFOaUFBUk9TdTVud2pBeGVqQTZUVERiSzNvNmxIWGJJTXJxSFBVeQpRcUFpOVZlb2ozTWNhNG1lTVY3RkFpOCsyaXNiYUExNnptSml5ZE5wdUpubmR6dHBncTFxckN2aEtQK0F6RWhXClFBbTY2c1JpRmZOYm5JZTEyK2xwK0dtZDY4bldIZUE9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
kypo_crp_head_version         = "3.1.11"
kypo_gen_users_version        = "2.0.1"
kypo_postgres_version         = "2.1.0"
man_image                     = "debian-11-man-preinstalled"
os_auth_url                   = "https://10.6.0.52:5000/v3"
openid_configuration_insecure = true
proxy_host                    = "10.18.0.67"
proxy_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRBWVl0SEtDRVBIbDlEUWRZMXhST1lkcDB3NVppOCtYdk9scFZSNTEzRGZyUW9vUTZUNEwydVcKLys4TjJJa280UVdnQndZRks0RUVBQ0toWkFOaUFBVHdZY1IvSk5JVUZJRncyYnF0YTk2cURVTm4weEN5b2I1VAp5RU9LcjljZ0NKSkptOUs1ZE5YNjRrMGh6V0FaR0lQbzFSRUEwQkNwTVdBTVAySjJkSHcrZjhEK1JySVI5dWcrCmdOZ1JvYmt6K1ZPNmwyTTZ6UTM0US9rQmk0VTg5WG89Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
users = {
    "kypo-admin" = {
      iss              = "https://head_host/keycloak/realms/KYPO"
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
