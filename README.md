Ansible Role: manage_accounts
=============================

This role offers basic functionalities to create and manage user accounts on your machines. The accounts are managed via
groups. It means that this role does not grant privileges directly to an account, but to the groups it belongs to.

Introduction
------------

Usually, people want to set up their virtual machines with some user accounts. This role was created to facilitate the
process. It performs some tasks as described:

1. Make sure that 2 groups `sudo` and `ssh` exist with proper rights.
    * Accounts in `sudo` group can use `sudo` command.
    * Accounts in `ssh` group can be accessed via `ssh`.
1. Create all groups described in `primary_group` and `other_groups`.
1. Create user accounts with provided setup.
1. Make sure that **only** accounts in `ssh` group can be accessed via `ssh`.
1. Disable root login via SSH.

Role Variables
--------------

The input must be put in an object called `user_accounts`. This is a list of dictionaries. Each of them contains
following key-value pairs:

| Key             | Data type        | Default value | Note                                                                                                                                                                                                                                                              |
|-----------------|------------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| username        | string           |               |                                                                                                                                                                                                                                                                   |
| password        | string           | !             | if the password is not provided, the password of this account will be locked, same as running [passwd --lock][1].                                                                                                                                                 |
| comment         | string           |               |                                                                                                                                                                                                                                                                   |
| primary_group   | string           |               | the primary group of this account                                                                                                                                                                                                                                 |
| other_groups    | list of string   |               | other groups this account also belongs to                                                                                                                                                                                                                         |
| groups_append   | yes/no           | no            | suppose an account already belongs to some groups. Setting this to `no` will remove all groups from the account and make sure that the account only belongs to `primary_group` and `other_groups`. Setting it to `yes` to keep the current groups of the account. |
| authorized_key  | list of string   |               | a list of paths to public keys, which will be added under the created account.                                                                                                                                                                                    |
| update_password | always/on_create | on_create     | `always` will update passwords if the current password and the input one are different. `on_create` will set the password only for newly created accounts.                                                                                                        |

Example Usage
-------------

Suppose that we want to create a new account called `tdoan` and modified the existing account `cloud`. We first declare
the `user_accounts` object as follows:

```yaml
user_accounts:
  - username: tdoan
    password: "my_secret"
    comment: "This is a comment"
    primary_group: tdoan
    other_groups:
      - sudo
      - ssh
      - docker
    groups_append: no
    authorized_key:
      - "{{ playbook_dir }}/keys/tdoan.pub"
      - "{{ playbook_dir }}/keys/another_key.pub"

  - username: cloud
    password: "cloud_password"
    comment: "This account already exists in the VM."
    primary_group: cloud
    other_groups:
      - sudo
      - ssh
    groups_append: yes
    authorized_key:
      - "{{ playbook_dir }}/keys/tdoan.pub"
```

* For account `tdoan`:
    * It belongs **only** to 3 groups: `sudo`, `ssh`, and `docker`.
    * 2 public keys are added under this account.
* For account `cloud`:
    * It keeps whatever groups it has plus two more groups: `sudo` and `ssh`.
    * 1 public key is added under this account.

Suppose that we put the above setup in `vars.yml` file. It can be used in a playbook like this:

```yaml
- name: Setup my virtual machine
  hosts: my-host
  become: yes
  vars_files:
    - vars/vars.yml
  roles:
    - role: tdoan2010.manage_accounts
```

License
-------

MIT

Author Information
------------------

This role was created in 2021 by [Triet Doan](mailto:triet.doan@gwdg.de).

[1]: https://man7.org/linux/man-pages/man1/passwd.1.html
