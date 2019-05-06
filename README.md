# Install Selfhosted Stacks on Docker

The following Apps are implemended:

- Nextcloud + Collabora + Talk
- Joomla
- Wordpress
- Bookstack
- Grav
- Rocket.Chat
- Wallabag
- Portainer
- Adminer
- restic & rclone Backup

Right now this will run on Ubuntu 16/18, Debian 9, CentOS 7, Amazon Linux 2.

The playbook runs on x86_64. It's tested on both architektures on AWS EC2 and Scaleway Server. Arm server are not yet tested. But may work.

This is work in progress. Not everything is tested. Before you use this in production check the installation very carefully.

## Preparation

Clone this repo and change into the directory `selfhosted_on_docker`.

```bash
git clone https://github.com/ReinerNippes/selfhosted_on_docker
cd selfhosted_on_docker
```

Install [Ansible](https://www.ansible.com/) and some needed tools by running the following command with a user that can sudo or is root.

```bash
./prepare_ansible.sh
```

Note that root must have also sudo right otherwise the script will complain. Some hoster use distros where is is not in the sudoers file. In this case you have to add `root ALL=(ALL) NOPASSWD:ALL` to /etc/sudoers.

Prepare your OS to run the playook and install Docker on your server.

```bash
./prepare_os.sh
```

You may skip this steps if you have already a running docker installation. Nevertheless you may miss some python packages. You have to install them manually.

## Configuration

Now you can configure the whole thing by editing the file `selfhosted_sites.yml` and some other files.

### Preliminary variables

First of all you must configure the base system.

```yaml

# Define your the top level domain for for selfhosted sites
base_domain: selfhosted.example.tld

# Your email adresse for letsencrypt
ssl_cert_email: selfhosted@.example.tld

# set this to true if you want to enable a postfix mail relay for all your selfhosted services
# edit group_vars/mail_relay.yml to set user/password for your upstream mail server
# all container can use this relay server to send mail
mailrelay_enabled: true

# your maildomain
selfhosted_mail_domain: '{{ base_domain }}'

# Enable to backup your server
backup_enabled: false

# adminer is a webfront end for your database at https://{{ base_domain }}/adminer/
adminer_enabled: false

# portainer is a webfront end for docker host at https://{{ base_domain }}/portainer/
portainer_enabled: true

# user for traefik dashboard at https://{{ base_domain }}/traefik/
traefik_api_enabled: false

```

Now you can define the apps you want to run by editing the variable `selfhosted_sites` in the file `selfhosted_sites.yml`

### The base system

Lets start with the base setup

```yaml
selfhosted_sites:
  - name: base
    type: base
    server_fqdn: {{ base_domain }}
```

Don't edit or delete this. It's the base setup. It provides the basic container to run your apps. If you delete this the playbook will abort with an error.

### Turnserver

If you want to use Talk with Nextcloud setup a turnserver. All Nextcloud instances will share the same turnserver.
The server_fqdn must be the same as the base setup. Otherwise no certificates wil be created.

```yaml
  - name: talk
    type: turnserver
    server_fqdn: {{ base_domain }}
```

### An app

If you want to start an Joomla app named myjoomla at joomla.selfhosted.example.tld it would like this

```yaml
  - name: myjoomla
    type: joomla
    server_fqdn: joomla.{{ base_domain }}
```

The name variable is used as the container name. So be carefull with special charaters.

### Some more apps

If you want to run several joomla installations just copy and change `name` and `server_fqdn`. `type` must be joomla

```yaml
  - name: myjoomla
    type: joomla
    server_fqdn: joomla.{{ base_domain }}

  - name: another_joomla
    type: joomla
    server_fqdn: another-joomla.{{ base_domain }}
```

### Nextcloud is differnet

Nextcloud has four additional variables. You can define if you want to install Collabora and/or Talk additional.
If you want to install Collabora you can choice your dictionaries as well.

```yaml
  - name: nextcloud
    type: nextcloud
    server_fqdn: nextcloud.{{ base_domain }}
    collabora:   true
    collabora_dictionaries: "de_DE en_GB en_US es_ES fr_FR it nl pt_BR pt_PT ru"
    talk:        true
    talk_server: {{ base_domain }}
```

### All working apps (so far)

This a list of all available apps:

```yaml
  - name: nextcloud
    type: nextcloud
    server_fqdn: nextcloud.{{ base_domain }}
    collabora:   true
    collabora_dictionaries: "de_DE en_GB en_US es_ES fr_FR it nl pt_BR pt_PT ru"
    talk:        true
    talk_server: {{ base_domain }}

  - name: joomla
    type: joomla
    server_fqdn: joomla.{{ base_domain }}

  - name: wordpress
    type: wordpress
    server_fqdn: wordpress.{{ base_domain }}

  - name: bookstack
    type: bookstack
    server_fqdn: bookstack.{{ base_domain }}

  - name: grav
    type: grav
    server_fqdn: grav.{{ base_domain }}

  - name: rocketchat
    type: rocketchat
    server_fqdn: rocketchat.{{ base_domain }}

  - name: wallabag
    type: wallabag
    server_fqdn: wallabag.{{ base_domain }}
```

How many apps you can host on one machine depends on your machine and your users. Well. What else.

## Install the apps

Run

```bash
./prepare_site.yml
```

## Access your apps

You'll find all passwords in /opt/selfhosted/secrets. To get a complete list run `sudo grep -r '' /opt/selfhosted/secrets`.

```text
/opt/selfhosted/secrets/bookstack_database_user_secret:kAO4pbmWbcalPq7fsp3z5PQLZbDD5C2n
/opt/selfhosted/secrets/bookstack_mysql_root_secret:36OeJ3KNjGQLOz7HqLsjNA1Rkr2uBuD6
/opt/selfhosted/secrets/joomla_database_user_secret:hpfWmTQCT3gXkbjl05NwMcXZrcnnuQOa
/opt/selfhosted/secrets/joomla_mysql_root_secret:ap0QPc8IKNDnSmhBHosNrT8deZWXPqGo
/opt/selfhosted/secrets/nextcloud_admin_secret:TQMdYgRC6KQ2w3MAdcqZsXtaZsh1iv5k
/opt/selfhosted/secrets/nextcloud_collabora_secret:GDCOq0qUOrucUGstiawb2Z9XFinG4Lt1
/opt/selfhosted/secrets/nextcloud_db_user_secret:YFzKJFHGu8kh3JkzRPGks47Fb24dSS02
/opt/selfhosted/secrets/nextcloud_redis_secret:yyaFBfdqgKelK9fujgto2KiCE4FXfILh
/opt/selfhosted/secrets/portainer_admin_secret:EhfKBfVsKsaFqgcvVjWloQ9b6zAP3vZN
/opt/selfhosted/secrets/talk_secret:VHMKK5buOUkYW0bmOcdiEhZ6rlRQ6s5r
/opt/selfhosted/secrets/wallabag_database_user_secret:XHBOeYPivzHCCCoHiSoY6iVVkylRZXLA
/opt/selfhosted/secrets/wallabag_db_user_secret:xqUysULX9waN9Yzhfq8nzC4hDk4s3Bxq
/opt/selfhosted/secrets/wallabag_mysql_root_secret:cBz1uTHccrOr0CN3Q36V5ji3DlP5GEhV
```

Nextcloud is ready to run. You can login as admin and start work.

All other apps have to be configured via their web interfaces. Please note that some apps are open to world until you configure them. E.g. `grav`.

The name of the database host for all apps is `{{ app_name }}-db`. In the above example of myjoomla it would be `myjoomla-db`. The db user you'll find in `group_vars/joomla.yml`.

### Mail relay

Enter the credentials of your upstream smtp server in `group_vars/mailrelay.yml`

```yaml
upstream_smtp_host:          ''
upstream_smtp_username:      ''
upstream_smtp_password:      ''
upstream_smtp_from_address:  ''
```

Configure your mail-relay container in your app. Below is the example for Nextcloud. Other apps must be configure through there web gui.

```yaml
# Nextcloud mail setup
app_configure_mail:               true
app_mail_from:                    'nextcloud'
app_mail_domain:                  'exmaple.tld'
app_mail_smtpmode:                'smtp'
app_mail_smtpauthtype:            ''
app_mail_smtpsecure:              ''
app_mail_smtpauth:                '0'
app_mail_smtphost:                'mail-relay'
app_mail_smtpport:                '25'
app_mail_smtpname:                ''
app_mail_smtppwd:                 ''
```

### Fine tuning

That's a long story. It's good to know yaml, jinja2 and ansible.

To start look at the yaml files in `group_vars/*.yml`.

If you don't like `/opt/selfhosted` as the base directory you may change it in `group_vars/all.yml`. If your favorite Postgrsql version isn't '10-alpine' you may change it in this file as well.

But. That's not tested. You may try.

```yaml
selfhosted_base_dir:            '/opt/selfhosted'
selfhosted_credential_store:    '{{ selfhosted_base_dir }}/secrets'

docker_default_postgres_image:  '10-alpine'
docker_default_mysql_image:     'latest'
docker_default_nginx_image:     'alpine'
docker_default_redis_image:     'alpine'
docker_default_mongodb_image:   'latest'
```

If you want to fine tune the apps you have look into each yml file.

For example if you want to change the mysql image for bookstack you may edit bookstack.yml and change `app_docker_mysql_image:  '10.4.4'`

```yaml
---
# bookstack varibales

# docker image version
app_docker_image:                  'latest'

app_docker_mysql_image:            '10.4.4'

# Enable/disable auto container updates via watchtower
app_watchtower:                    'false'
app_db_watchtower:                 'false'

# data dir
app_base_dir:                      '{{ selfhosted_base_dir }}/{{ app_name }}'
app_data_dir:                      '{{ app_base_dir }}/data'
app_datadump_dir:                  '{{ app_base_dir }}/database_dump'

# database settings (choose one)
app_db_type:                       'mysql'        # (MariaDB)

# options for mariadb or postgres
app_db_name:                       'bookstack'
app_db_user:                       'bookstack'
app_db_passwd:                     ''              # leave empty to generate random password
```

### Watchtower

The watchtower container takes care about your container and can update them automatic to a newer version if a newer images is pushed to hub.docker.io. Per default all container are set to `app_watchtower: false`. You have to change this to your needs.

## Remove everything

If you want to get rid of the containers, networks and volumes run the following command.

```bash
sudo ./scripts/remove_all_container.sh
```

Your data won't be deleted. You have to do this with.

```bash
sudo rm -rf /opt/selfhosted
```

If you test your setup it's wise to have a backup of `/opt/selfhosted/traefik`. Letsencrypt limits the number of certificate request to there server.
