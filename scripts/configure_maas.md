# MAAS Configuration
## Network configuration of the MAAS controller
The following is the configuration file for the MAAS controller, file ```/etc/netplan/maas_net.yaml```
```
network:
  ethernets:
    ens18:
      addresses:
      - 10.6.0.10/24
      nameservers:
        addresses:
        - 8.8.8.8
        search: []
      routes:
      - to: default
        via: 10.6.0.1
  version: 2
  vlans:
    ens18.7:
      addresses:
      - 10.7.0.10/24
      id: 7
      link: ens18
    ens18.8:
      addresses:
      - 10.8.0.10/24
      id: 8
      link: ens18
```

## Image to download and sync in MAAS repository
![](/scripts/images/image.png)
## Settings
### Commissioning
The commisionning image
![](/scripts/images/commissioning.png)
### Deploy
The deployment image
![](/scripts/images/deploy.png)
### Storage
Storage configuration - LVM Layout
![](/scripts/images/storage.png)
### DNS
Internal and external DNS
![](/scripts/images/dns.png)

## Subnets
![](/scripts/images/subnets.png)

