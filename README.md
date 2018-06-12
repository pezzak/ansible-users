[![Build Status](https://travis-ci.org/pezzak/ansible-users.svg?branch=master)](https://travis-ci.org/pezzak/ansible-users)

# ansible-users

Role to manage users on a system.

## Role configuration

* users_create_per_user_group (default: true) - when creating users, also
  create a group with the same username and make that the user's primary
  group.
* users_group (default: users) - if users_create_per_user_group is _not_ set,
  then this is the primary group for all created users.
* users_default_shell (default: /bin/bash) - the default shell if none is
  specified for the user.
* users_create_homedirs (default: true) - create home directories for new
  users. Set this to false if you manage home directories separately.

## Creating users

Add a users variable containing the list of users to add. A good place to put
this is in `group_vars/all` or `group_vars/groupname` if you only want the
users to be on certain machines.
You can set hash_behaviour = merge (see https://docs.ansible.com/ansible/2.4/intro_configuration.html#hash-behaviour)
in your ansible configuration, add all users in group_vars/all and manage users in
group_vars/groupname only by state field.


The following attributes are required for each user:

* username - The user's username.
* name - The full name of the user (gecos field).
* home - The home directory of the user to create (optional, defaults to /home/username).
* uid - The numeric user id for the user (optional). This is required for uid consistency
  across systems.
* gid - The numeric group id for the group (optional). Otherwise, the
  uid will be used.
* password - If a hash is provided then that will be used, but otherwise the
  account will be locked.
* update_password - This can be either 'always' or 'on_create'
  - 'always' will update passwords if they differ. (default)
  - 'on_create' will only set the password for newly created users.
* group - Optional primary group override.
* groups - A list of supplementary groups for the user.
* append - If yes, will only add groups, not set them to just the list in groups (optional).
* profile - A string block for setting custom shell profiles.
* ssh_key - This should be a list of SSH keys for the user (optional). Each SSH key
  should be included directly and should have no newlines.
* generate_ssh_key - Whether to generate a SSH key for the user (optional, defaults to no).
* state - Should be 'present'

In addition, the following items are optional for each user:

* shell - The user's shell. This defaults to /bin/bash. The default is
  configurable using the users_default_shell variable if you want to give all
  users the same shell, but it is different than /bin/bash.

Example:

group_vars/all

    ---
    users:
      foo:
        name: Foo Barrington
        state: present
        groups: ['wheel','systemd-journal']
        uid: 1001
        home: /local/home/foo
        profile: |
          alias ll='ls -lah'
        ssh_key:
          - "ssh-rsa AAAAA.... foo@machine"
          - "ssh-rsa AAAAB.... foo2@machine"
      bar:
        name: Bar Man 
        state: absent
        groups: ['wheel','systemd-journal']
        uid: 1002
        home: /local/home/bar
        profile: |
          alias ll='ls -lah'
        ssh_key:
          - "ssh-rsa AAAAA.... bar@machine"
          - "ssh-rsa AAAAB.... bar2@machine"
    groups_to_create:
      - name: developers
        gid: 10000

group_vars/groupname

    ---
    users:
      bar:
        state: present

## Deleting users

To delete a user you must set 'state' field to 'absent'.
However, it is recommended that you also keep the `uid` field for reference so
that numeric user ids are not accidentally reused.

You can optionally choose to remove the user's home directory and mail spool with
the `remove` parameter, and force removal of files with the `force` parameter.

    users:
      bar:
        uid: 1002
        state: absent
        remove: yes
        force: yes
