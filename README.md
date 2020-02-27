Pulp 3 Ansible installer
========================

The Pulp 3 Ansible installer consists of a collection of roles. Each role installs and configures a
component of Pulp. The roles are not currently available on Ansible Galaxy; to run the Pulp 3
Ansible installer, the [ansible-pulp](https://github.com/pulp/ansible-pulp) git repository must
be cloned or downloaded.

This version of the installer, 3.1.1, installs Pulp 3.1.1 specifically.

If run against an older version of Pulp 3, it will upgrade it to 3.1.1.

System Requirements
-------------------

The [control node](https://docs.ansible.com/ansible/2.5/network/getting_started/basic_concepts.html#control-node)
must have Python 3 and Ansible installed.

The [managed node](https://docs.ansible.com/ansible/2.5/network/getting_started/basic_concepts.html#managed-nodes)
must be one of these currently supported operating systems:
* CentOS 7
* Debian Buster
* Fedora 30 or later

Variables
---------

**Each role documents all the variables that it uses in its own README**. Some variables are
used by multiple roles. In that case, they are be documented in their primary role and mentioned in
the `shared_variables` section the other roles.

**Required Variables:**
Most variables have sane defaults but a few are required. See ``example-use/group_vars/all`` for
the minimal set of required variables.


Ansible Boilerplate
-----------------

These roles can be used against any managed node and are highly configurable.  Knowledge of
[ansible basics](https://docs.ansible.com/ansible/2.5/user_guide/intro_getting_started.html) will
be helpful, but even if you are new to Ansible, this section will get you started, or you can try
the Vagrant installations to bypass the Ansible boilerplate.

First, you will need to configure ssh between your control node and your managed node. When you can
ssh into the managed node without a password, you are ready to move to the next step.

Next, add the managed node's hostname or ip address to `/etc/ansible/hosts`.

It may be helpful to ensure that Ansible can communicate with the managed node.

```
ansible all -m ping -u <managed_node_username>
```

Using the example playbook
--------------------------

The playbook has external requirements which should be installed from ansible-galaxy.

```
ansible-galaxy install -r requirements.yml
```

You should now be able to run the example playbook.

Some of the roles used in the playbook use root privalages on the managed node, so when prompted,
you will need to provide the password for the managed node user.

```
ansible-playbook example-use/playbook.yml -u <managed_node_username> --ask-become-pass
```

To configure a custom install, you will need to set configuration variables. In the simplest case,
they can be set in the playbook. See the ansible docs for more flexible idiomatic alternatives.


Testing
-------

The tests can be run as they are on travis with **tox**, or they can run with various options using
**molecule** directly.

**Requirements:**
Install Docker, and add yourself to the group that is authorized to
administer containers, and log out and back in to make the permissions change
take effect. The authorized group is typically the "docker" group:

```bash
gpasswd --add "$(whoami)" docker
```

**NOTE:** Docker containers can differ from bare-metal or VM OS installs.
They can have different packages installed, they can run different kernels,
and so on.

**Using Tox:**

1. Install [tox](https://tox.readthedocs.io/en/latest/). This can be done
   through the system package manager or into a virtualenv:

   ```bash
   python3 -m venv ~/.venvs/ansible-pulp
   pip install --upgrade pip
   pip install tox
   ```
2. Install at least one of the Python interpreters listed in tox.ini. These are
   currently Python 2.7 and 3.6.
   **WARNING:** Anyone added to the docker group is root equivalent. More
   information [here](https://github.com/docker/docker/issues/9976) and
   [here](https://docs.docker.com/engine/security/security/).

4. Run `tox`. If you only have a subset of the supported Python interpreters
   available, specify which environments to exercise:

   ```bash
   tox -e py36
   ```

**Using Molecule:**

1. Install [molecule](https://molecule.readthedocs.io/en/latest/),
[molecule-inspec](https://github.com/ansible-community/molecule-inspec),
and [ansible-lint](https://docs.ansible.com/ansible-lint/).


It is recommended that you do so with `pip` in a virtualenv.
2. Run molecule commands.

   Test all scenarios on all hosts.
   ```bash
   molecule test --all
   ```

   Test a specific scenario.
   ```bash
   molecule test --scenario-name source
   ```

   Use debug for increased verbosity.
   ```bash
   molecule --debug test --all
   ```

   Create and provision, but don't run tests or destroy.
   ```bash
   molecule converge --all
   ```

Pulp 3 Ansible use-case
========================

   - Create repository:
   http POST http://localhost:24817/pulp/api/v3/repositories/ansible/ansible/ name=newrepo
   Output:
   {
    "description": null,
    "latest_version_href": "/pulp/api/v3/repositories/ansible/ansible/030e9513-a4a3-4dd1-b277-5d63ba7eaab9/versions/0/",
    "name": "newrepo",
    "pulp_created": "2020-02-27T10:13:56.995789Z",
    "pulp_href": "/pulp/api/v3/repositories/ansible/ansible/030e9513-a4a3-4dd1-b277-5d63ba7eaab9/",
    "versions_href": "/pulp/api/v3/repositories/ansible/ansible/030e9513-a4a3-4dd1-b277-5d63ba7eaab9/versions/"
   }
  - Create distribution for repository
  http POST http://localhost:24817/pulp/api/v3/distributions/ansible/ansible/  name='newbaz'   base_path='newcontent'  repository="/pulp/api/v3/repositories/ansible/ansible/030e9513-a4a3-4dd1-b277-5d63ba7eaab9/"
  - Push  role to artifact
  http -f POST http://localhost:24817/pulp/api/v3/artifacts/ file@pulp.tar.gz
  Output:
  {
    "file": "artifact/be/ceea383b9e17c89cb940cc9e14d14e03f50dcd1e7714aeb400f9a27bc7f5b4",
    "md5": "a0eebe3da58d9705f55562e705348c0b",
    "pulp_created": "2020-02-27T10:21:33.692376Z",
    "pulp_href": "/pulp/api/v3/artifacts/182de6bd-ea67-4854-92aa-d7e78f08a07c/",
    "sha1": "b6202b6b64ee873a3e454fcfdffaa5a631c353a4",
    "sha224": "0c1be4d215d0930246eee9cda438319b594ce33252e3b87d586b475a",
    "sha256": "beceea383b9e17c89cb940cc9e14d14e03f50dcd1e7714aeb400f9a27bc7f5b4",
    "sha384": "9b8fb1b35ab6cfdc23da9fbefcb822cbb4da7b08e4049526597acf85f84bf778cb5f7ea6e386ff59ef1e693b99cf0e4a",
    "sha512": "cabdd8f607f679b31bab311f290459377660fed991990ed5f30095368eff671e8b4680a585c14ee0370fdf0b5442367e222114b6f36704c05981c369eec4ac9c",
    "size": 51146
  }
  - Link role to  artifact already created
  http -f POST http://localhost:24817/pulp/api/v3/content/ansible/roles/ version=1.0 name=pulp namespace=pulp artifact="/pulp/api/v3/artifacts/182de6bd-ea67-4854-92aa-d7e78f08a07c/"
    Output :
  {
    "artifact": "/pulp/api/v3/artifacts/182de6bd-ea67-4854-92aa-d7e78f08a07c/",
    "name": "pulp",
    "namespace": "pulp",
    "pulp_created": "2020-02-27T10:41:54.987736Z",
    "pulp_href": "/pulp/api/v3/content/ansible/roles/1fbf1488-be21-42bd-96a2-238a07a301e6/",
    "version": "1.0"
}

  - Add role de repo already created
  echo '{"add_content_units": ["'/pulp/api/v3/content/ansible/roles/336f512b-0d28-49aa-b224-c093a916dc0b/'"]}' | http POST http://localhost:24817/pulp/api/v3/repositories/ansible/ansible/030e9513-a4a3-4dd1-b277-5d63ba7eaab9/modify/
  - configure ansible.cfg tu use internal galaxy
  [galaxy]
  server: http://localhost:24817/pulp_ansible/galaxy/newcontent/
  - Installation role using ansible-galaxy
  ansible-galaxy role install pulp,1.0

  
  

 
