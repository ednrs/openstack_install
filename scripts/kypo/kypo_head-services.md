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
head_ip                       = "10.18.0.35"
kubernetes_host               = "10.18.0.35"
kubernetes_client_certificate = "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUIzVENDQVdLZ0F3SUJBZ0lSQUpYVVhUODRyWDFFeVNsZjRqQTRFVU13Q2dZSUtvWkl6ajBFQXdNd0dERVcKTUJRR0ExVUVBeE1OYTNWaVpYSnVaWFJsY3kxallUQWVGdzB5TkRBMk1EWXdOVE0xTlRKYUZ3MHpOREEyTURZeApOek0xTlRKYU1ESXhGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1SY3dGUVlEVlFRREV3NTBaWEp5CllXWnZjbTB0ZFhObGNqQjJNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQWlBMklBQktNTW5MMmpLS0xlejZSK2d5blAKYm9oQ2h4R1IvZE1XWWVPRVp6VERYdE94Vm1YWXRTUHNqbnhkZ0Z6S1ZXaXBsRDBQOFVaaDA2dVUzMm1uVlduUwpXaEE2Qnl3S3lUcG5xa3lScG53VmhCUmdIRVY3QXFaeEZ6dGovVnIyNUdjbEFhTldNRlF3RGdZRFZSMFBBUUgvCkJBUURBZ1dnTUJNR0ExVWRKUVFNTUFvR0NDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGoKQkJnd0ZvQVV3SnhURTByQTEzWWZxeWZXNFNXMjBBQnJ0Y2t3Q2dZSUtvWkl6ajBFQXdNRGFRQXdaZ0l4QUxiSgpTNFNOTU1BNzh6d0N5NzdMUzZwdEtDWDEwQ2VWNVdmTTVua1RLcU5TUEo1N3JLY3U4ZENSQzYrSlA3NXNmUUl4CkFMSGxjSFc0ZVZlNDQ3ZlRKWUtsaytVOUFoY3Z3NkJWK3VWU0Zwdm9qNjRLVkFwVjlNZ3FibENQSGhUR01yQXgKVkE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
kubernetes_client_key         = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRDUXpZWFJFbDVxZk1DYVhYVGlmV1lSN3A5Y0xibkpTNUlIb0d4Q2JxMjNXcEpydmUwdjlyU2QKV2RjQ05UWnJnODJnQndZRks0RUVBQ0toWkFOaUFBU2pESnk5b3lpaTNzK2tmb01wejI2SVFvY1JrZjNURm1IagpoR2MwdzE3VHNWWmwyTFVqN0k1OFhZQmN5bFZvcVpROUQvRkdZZE9ybE45cHAxVnAwbG9RT2djc0NzazZaNnBNCmthWjhGWVFVWUJ4RmV3S21jUmM3WS8xYTl1Um5KUUU9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
kypo_crp_head_version         = "3.1.11"
kypo_gen_users_version        = "2.0.1"
kypo_postgres_version         = "2.1.0"
man_image                     = "debian-11-man-preinstalled"
os_auth_url                   = "https://10.6.0.52:5000/v3"
openid_configuration_insecure = true
proxy_host                    = "10.18.0.240"
proxy_key = "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1JR2tBZ0VCQkRCNmJxK0NEcnQ3aDRrbjd4ZG10RFdTU21SWEF0T3NoOExpQ3ZBOWxlbzRoSVNGaitpWHI1WUwKQ3U1aWllOEt2VUNnQndZRks0RUVBQ0toWkFOaUFBVDhXRWNTVFFndEZtZ1k0Q3hMcHF6SEkwZVFxUzhFOFJPMQpubUpPOThJek5XcUVWMllzbkx5NVJEZVd6bFdydkswcHlTcDdFck9LOS9HZ1JpRFpJMTQycFppWVR4VStOcmI1ClQ5Yk5QamMwRHNaWjRTRU9OaFMzb1ZGaHNnMjBiTDQ9Ci0tLS0tRU5EIEVDIFBSSVZBVEUgS0VZLS0tLS0K"
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
