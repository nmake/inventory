# CSV based ansible inventory

## Quick start

1) Clone the repository

2) Link the repo into the default collection path

```
mkdir -p ~/.ansible/collections/ansible_collections/nmake
ln -s ~/projects/inventory ~/.ansible/collections/ansible_collections/nmake
```

3) Define a `nmake_inventory_csv` file:

Note:

- A value in the format of `vars:xxx` will be replace with values from the yaml file above
- A column called groups should contain a space delimited list of groups the host should belong to
- All other columns will be added as attributes of the host in the inventory
- See the constructued inventory plugin for details about `compose`, `keyed_groups` and `groups`

```yaml
plugin: nmake.inventory.csv
source: "/full/path/to/inventory.csv"

# add an attribute to each host based on a conditional
compose:
  ansible_become: ansible_network_os == "eos"
  ansible_python_interpreter: python

# build dynamic groups based on csv columns
keyed_groups:
  - key: site
    prefix: site
  - key: ansible_network_os
    prefix: ""
    separator: ""

# allow the csv to contain `vars:xxx` values which reference these
vars:
  ansible_user: "{{ lookup('env', 'ansible_user') }}"
  ansible_password: "{{ lookup('env', 'ansible_password') }}"
  ansible_become_pass: "{{ lookup('env', 'ansible_become_pass') }}"

# add an attribute to each host if it's not in the csv
defaults:
  ansible_become_method: enable
  ansible_connection: network_cli
  ansible_python_interpreter: python


# in case the CSV columns don't match what we need
column_replace:
  os: ansible_network_os
```

4) Define the CSV file:

```
host,os,ansible_user,ansible_password,ansible_become_pass,site,groups
nxos101,nxos,vars:ansible_user,vars:ansible_password,vars:ansible_become_pass,my_lab,red blue
nxos102,nxos,vars:ansible_user,vars:ansible_password,vars:ansible_become_pass,my_lab,blue yellow
nxos103,nxos,vars:ansible_user,vars:ansible_password,vars:ansible_become_pass,my_lab,red blue
nxos104,nxos,vars:ansible_user,vars:ansible_password,vars:ansible_become_pass,my_lab,blue yellow
eos10[1:4],eos,vars:ansible_user,vars:ansible_password,vars:ansible_become_pass,my_lab,orange red
vyos10[1:4],vyos,vars:ansible_user,vars:ansible_password,vars:ansible_become_pass,my_lab,orange red
```

5) Update the ansible.cfg file to allow the CSV inventory plugin:

```ini
[inventory]
enable_plugins = nmake.inventory.csv
```

6) Run ansible:

```yaml
# site.yaml
- hosts: all
  gather_facts: False
  tasks:
  - name: Use the platform facts module for each
    action: "{{ ansible_network_os }}_facts"
    args:
      gather_network_resources:
      - interfaces
  - debug:
      msg: "{{ hostvars[inventory_hostname] }}"
```

```shell
ansible-playbook -i nmake_inventory_csv.yaml site.yaml
```
