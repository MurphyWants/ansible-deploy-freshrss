# ansible-deploy-freshrss
Purpose: Setup the freshrss docker container using rootless podman on its own partition

Image: 'lscr.io/linuxserver/freshrss'
tag: latest

This role will:
- Create a new parition for freshrss if it doesn't already exist
  - It can grow the partition if it does exist and needs more space (increase variable FRESHRSS_VOLUME_MIN_SIZE)
- Create a new service account, unprivileged 
- Create a new systemd service from a template to run podman as the service account
- Installs nginx
- If FRESHRSS_SSL_CERT_PATH_COPY or FRESHRSS_SSL_KEY_PATH_COPY are defined, it will copy those files to the system
- Creates a new nginx config from a template to proxy podman and use https

## Requirements/Dependencies
Uses the following modules:
- community.general
- ansible.builtin
- containers.podman
- ansible.posix

## Variables

Variable | Default Value | Description
---|---|---
FRESHRSS_VOLUME_DIR | /mnt/freshrss | The location of the container persistent data. A parition will be created and mounted here if the folder doens't exist. 
FRESHRSS_VOLUME_MIN_SIZE | 2G | How large should the patition be
FRESHRSS_LVM_LV_NAME | lvm_freshrss | The name of the LV to be created
FRESHRSS_LVM_VG_NAME | rhel | The name of the VG to be created on
FRESHRSS_SERVICE_ACCOUNT | srv_freshrss | The name of the service account
FRESHRSS_URL | freshrss.example.com | The URL of your freshrss instance. 
FRESHRSS_ENV_TZ | 'America/New_York' | Docker Time Zones: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
FRESHRSS_SSL_CERT_PATH | '/etc/nginx/certs/freshrss.crt' | Location on the host where the SSL cert will live, for the nginx config
FRESHRSS_SSL_KEY_PATH | '/etc/nginx/certs/freshrss.key' | Location on the host where the SSL key will live, for the nginx config
FRESHRSS_SSL_CERT_PATH_COPY | | Undefined, if defined as part of the run command this file will be copied to the system to service as the SSL cert
FRESHRSS_SSL_KEY_PATH_COPY | | Undefined, if defined as part of the run command this file will be copied to the system to service as the SSL key

## Example Playbook

install_freshrss.yml
```
- hosts: freshrss # Assuming the 'freshrss' group exists
  gather_facts: yes
  become: yes
  roles:
  - role: ansible-deploy-freshrss
```

## Example Inventory

static.ini
```
[all]
rhel-freshrss.domain.example.com

[freshrss]
rhel-freshrss.domain.example.com

[freshrss:vars]
FRESHRSS_URL="freshrss.domain.example.com"
```

## Example Command

Setup With SSL Certs:
```
ansible-playbook -v -i inventories/dev_inventory.ini -u username -k -K -e 'FRESHRSS_SSL_CERT_PATH_COPY=../../nosync.nosync/certs/cert.pem' -e 'FRESHRSS_SSL_KEY_PATH_COPY=../../nosync.nosync/certs/privkey.pem' playbooks/install_freshrss.yml
```

Increase partition size to 5G

```
ansible-playbook -v -i inventories/dev_inventory.ini -u username -k -K -e 'FRESHRSS_VOLUME_MIN_SIZE=5G' playbooks/install_freshrss.yml
```
