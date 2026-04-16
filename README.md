# Ansible Role: Users

An Ansible role to manage local users, groups, and SSH authorized keys on Debian and Ubuntu systems. It also handles sudo privileges configuration.

The role is tested against the latest Ubuntu LTS and Debian releases but may work with other Debian-derived distributions.

**Note: this role is intended to be used on personal servers and simple lab environments. Some tests have been added to check for possible misbehavior, which could, for example, result in potentially locking users out. But it is not recommended to use this role in production environments or when you do not have alternative ways of accessing the system in case of misconfiguration.**

## Role Variables

Available variables are listed below, along with some sample values. See `defaults/main.yaml` for a full list of values.

```yaml
users_groups:
  - groupname: testgroup
    state: present # Set to absent to remove the group.
    gid: 1001 # Optional
```

A list of groups to manage on the system.

```yaml
users_users:
  - username: testuser
    name: John Doe
    state: present        # Optional. Set to absent to remove the user. Defaults to present.
    password: '$y$9T$...' # Optional. Hashed password.
    disabled: false       # Optional. If true, removes SSH authorized keys.
    sudo: true            # Optional. Grants sudo privileges via /etc/sudoers.d/02-ansible-users.
    group: testgroup      # Optional. Primary group.
    groups:               # Optional. Secondary groups to add. Existing group memberships not listed here are preserved.
      - adm
    shell: /bin/bash      # Optional. Defaults to users_default_shell.
    ssh_pubkeys: |        # Optional. SSH public keys for the user.
      ssh-ed25519 somepublickey johndoe@example.com
  - username: removeduser
    state: absent
```

A list of users to manage on the system.

Set `state: absent` to remove a user; all other fields are optional for absent users. Active users (the default) are ensured to be present.

Password must be hashed, and you can use the `mkpasswd` command to create a hashed password which you can pass as the password. Ideally, this should be encrypted using Ansible Vault as it otherwise might be at risk for bruteforcing if left exposed.

The `groups` field specifies secondary groups to add the user to. Group memberships not listed here (e.g., groups assigned by another Ansible role) are preserved — the role only adds the listed groups without removing existing ones.

See `defaults/main.yaml` for a full list of variables that can be set for each user.


```yaml
users_sudo_require_password: false
```

Controls whether users must enter their password when running sudo. Defaults to `false` (passwordless sudo).

When set to `true`, the role will **fail before making any changes** if a user has `sudo: true` but no password set (or has a locked account with `!`). This ensures the system is never left in an inconsistent state.

```yaml
users_remove_files: true
```

Attempt to remove files belonging to the user, such as home directory and mail.

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: users
      vars:
        users_groups:
          - groupname: developers
            state: present
        users_users:
          - username: alice
            name: Alice Smith
            sudo: true
            groups:
              - developers
            ssh_pubkeys: |
              ssh-ed25519 AAAAC3NzaC1... alice@example.com
          - username: former_employee
            state: absent
        users_sudo_require_password: true
```

# License

[MIT No Attribution](https://opensource.org/license/mit-0)

# Author

This Ansible role was created in 2025 by Eirik Nicolai Synnes and published as an Open Source project in 2026.

# Acknowledgements

This role uses Docker images created by Jeff Geerling for testing. The approach for developing this role, as well how to use GitHub Actions to manage it, is inspired by his work. You can find him on GitHub at https://github.com/geerlingguy.