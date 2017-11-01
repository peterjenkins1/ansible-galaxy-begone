# ansible-galaxy-begone

## What?

An opinionated Ansible role and library dependency manager backwards compatible with `ansible-galaxy install`.

### Features:
* Install roles *and libraries* from a single tool.
* Single playbook with no dependancies.
* Backwards compatible with `ansible-galaxy install` [requirements.yml syntax](http://docs.ansible.com/ansible/latest/galaxy.html#installing-multiple-roles-from-a-file).
* Indempotent and logical behaviour (no `--force` nonsense).
* Human friendly defaults. Just run `ansible-playbook ansible-galaxy-begone.yml` and your roles will be updated.
* Roles are real git checkouts. Updates are fast. Easy to switch to feature branches for testing/development.
* Global and per-dependancy instalation destination (`ext-roles` is the default instalation directory).
* Support for complex/messy libraries (recursive git and removing files from upstream repos).

### Non-features:
* Ansible galaxy installs are not possible. You have to use a real github url (and ideally a specific SHA1).
* Ansible role dependacies (as defined in `meta/main.yml`) are not fetched. You have to find the dependacies and put them in `requirements.yml` (ideally with a specifc SHA1).

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

Consider the command line I was using for AGI before I wrote this role:
```
ansible-galaxy install \
  -r requirements.yml \ # Even the requirements.yml file is not a default
  -p ext-roles \        # default roles path is `roles` which typically has local roles too. Messy!
  --force \             # Default behaviour is to fail, not update. Not very Ansible-like!
  --verbose \           # Default doesn't show if things changed
  --no-deps             # Role dependancies are a mess and on by default. No versioning!  
```

Then consider you are using some Ansible modules. These aren't imported with AGI (and you can't easily fool the tool either). Sure you can put your modules inside roles and run them 'early' in your playbooks so they are imported, but it's a messy approach (and the awesome `ansible-doc` won't find them either). Many great upstream projects like [ceph-ansible](https://github.com/ceph/ceph-ansible/) bundle modules in git repos, but there isn't a standard structure for doing this. The result is an ugly shell script which is hard to maintain. 

For more background see:
* https://github.com/ansible/galaxy-issues/issues/49
* (Installing ansible roles without ansible-galaxy)[https://medium.com/@peterjenkins/installing-ansible-roles-without-ansible-galaxy-e062d11a3ce0]

## Other details

* If you don't like the name `ext-roles` you can override the default roles path by setting `requirements_ext_roles_path` on the command line with `-e requirements_ext_roles_path=foo` or in your `group_vars`/`host_vars`/`inventory` files. It's just plain old Ansible so anything goes.
* You'll want to add `ext-roles` and any library directories to your `.gitignore` file. Since it's one directory it's dead easy and ensures nobody commits external code to the wrong repo.

### Extended `requirements.yml` syntax 

Added keywords:

* `dest`: You can put roles and libraries in different places. This makes more sense for libraries where the modules might be in different places.
* `remove`: A list of files (relative path) to remove from the upstream repository.
* `recursive`: The git checkout will fetch submodules.

Relevant existing keywords:
* `name`: Rename the role/library. This works as expected.

Some real examples from a recent project:
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

(This author wonders why colon was used as a list seperator!)

```
[defaults]
roles_path = roles:ext-roles:library/ceph-ansible/roles
library = library:library/ntc-ansible/library:library/ceph-ansible/library
```
