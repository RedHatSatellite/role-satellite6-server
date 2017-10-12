Install and configure Satellite 6 on a RHEL 6 or 7 host. 

This is based on the process outlined in the [Red Hat Satellite 6 Installation Guide](https://access.redhat.com/documentation/en/red-hat-satellite/)

=======
Invoke the role using only one of the below three include statements, in order to pass in the data required to register the system with RHN: 

```YAML
- hosts: satellite6
  roles:
    ## use rhn username + password
    - { role: role-satellite6-server rhn_user: my-rhn-username rhn_password: my-rhn-password }
    ## use rhn activation key
    - { role: role-satellite6-server rhn_activationkey: some-key }
    ## use specific pool ids, found via "subscription-manager list --avaialable"
    ##   This is needed when your version of Ansible uses buggy redhat_subscription module prior to PR 1204. Before that, redhat_subscription won't be able to find subs
    - { role: role-satellite6-server rhn_pool_ids: ["somelongpoolid", "someotherlongpoolid"] } 
```

If you don't specify the variable `satellite_version` (6.1 or 6.2), then the latest version is assumed.

## More complete (and complex) setup

If you create an empty directory, where you create all the following files in a similar manner (search for `YOUR` to see where you all need to adapt:

```
$ head -n-0 ansible.cfg credentials.cfg inventory.cfg satellite6.yml roles/requirements.yml 
==> ansible.cfg <==
[defaults]
roles_path = ./roles
inventory = ./inventory.cfg

==> credentials.cfg <==
---
rhn_user: YOUR_RHN_USER
rhn_pass: YOUR_RHN_PASSWORD
rhn_pool_pattern: '^$' # optional, the default pattern is IMHO too "greedy"
rhn_pool_ids: # optional, necessary if you keep the empty pool pattern above
  - 'abcdef01234567890abcdef123456789' # must contain Satellite subscription

==> inventory.cfg <==
[YOUR_GROUP_NAME]
YOUR_SATELLITE_SERVER_FQDN

[YOUR_GROUP_NAME:vars]
satellite_version=6.2

==> satellite6.yml <==
- hosts: YOUR_GROUP_NAME
  user: root
  vars_files:
    - credentials.cfg
  roles:
    - satellite6-server

==> roles/requirements.yml <==
---
- src: https://github.com/YOUR_GITHUB_USER/role-satellite6-server
  version: master
  name: satellite6-server
```

Then you may run the following commands to install the role and configure your Satellite 6 server.

```
ansible-galaxy -v install --force -r roles/requirements.yml
ansible-playbook satellite6.yml
```

The last command is assuming that you''ve already copied your SSH-key to the root user on your Satellite-server, and that the Satellite server has a basic RHEL 7 installation (RHEL 6 might work, hasn''t been tested).

Once the installation is successful, you can point your browser to https://YOUR_SATELLITE_SERVER_FQDN/ and grab the admin user and password, `admin_username` and `admin_password` from the used answers file `{{ installer_answer_file }}`, as defined under `roles/satellite6-server/vars/main.yml`.

Next steps would be to generate a manifest on your account at https://access.redhat.com/ and configure the Satellite server. Have fun!
