# ansible-galaxy-begone

## What?

An opinionated Ansible role and library dependency manager backwards compatible with ansible-galaxy install.

### Features:
* Install roles *and libraries* from a single tool.
* Backwards compatible with `ansible-galaxy install` [requirements.yml syntax](http://docs.ansible.com/ansible/latest/galaxy.html#installing-multiple-roles-from-a-file).
* Human friendly defaults. Just run `ansible-playbook ansible-galaxy-begone.yml` and your roles will be updated.
* Single playbook with no dependancies.
* Global and per-dependancy instalation destination ('ext-roles' is the default)
* Support for complex libraries (recursive git and removing files from upstream repos)

### Non-features:
* Ansible galaxy installs are not possible. You have to use a real github url (and ideally a version SHA1)
* Ansible role dependacies (as defined in `meta/main.yml`) are not fetched. You have to find the dependacies and put them in `requirements.yml` (ideally with a version SHA1)

## How do I use it?

* Define your requirements in a [requirements.yml file](http://docs.ansible.com/ansible/latest/galaxy.html#installing-multiple-roles-from-a-file)
* Configure ansible to load dependacies from specific directories. Create/edit `ansible.cfg` and add `ext-roles` to your `roles_path`
```
[defaults]
roles_path = roles:ext-roles
```
* Install and run the tool:
```
curl -o ansible-galaxy-begone.yml https://raw.githubusercontent.com/peterjenkins1/ansible-galaxy-begone/master/ansible-galaxy-begone.yml
ansible-playbook ansible-galaxy-begone.yml -i localhost,
```

## Why? Why not just use `ansible-galaxy install`?

For basic use `ansible-galaxy install` (AGI from here on) is ok but for larger projects the limitiations get annoying. 

## Other details

* You'll want to add `ext-roles` and any libraries to your `.gitignore` file. That lets you keep some roles in the same repo without mixing in code from other repos.

### Extended `requirements.yml` syntax 

```
- src: https://github.com/ceph/ceph-ansible.git
  dest: library

- src: https://github.com/networktocode/ntc-ansible.git
  dest: library
  # Remove some ntc files which break ansible:
  # https://github.com/networktocode/ntc-ansible/issues/142
  recursive: yes
  remove:
    - setup.cfg
    - setup.py
    - ntc-templates/setup.py
```

### `ansible.cfg` example for libraries

(Your author wonders why colon was used as a list seperator)

```
[defaults]
roles_path = roles:ext-roles:library/ceph-ansible/roles
library = library:library/ntc-ansible/library:library/ceph-ansible/library
```
