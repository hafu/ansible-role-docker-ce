docker-ce
=========

[![Build Status](https://api.travis-ci.com/hafu/ansible-role-docker-ce.svg?branch=master)](https://travis-ci.com/hafu/ansible-role-docker-ce)

Installs and configures Docker CE (Community Edition). On Debian based systems 
the official Docker repository will be added. May conflicting packages provided
by the distribution repository will be removed. 
 
Optionally user namespaces can be used and setup.

Requirements
------------

None

Role Variables
--------------

Default variables are listed in `defaults/main.yml`, OS depended variables are
listed in `vars/`.

```yaml
docker_ce_install_docker_compose: false
docker_ce_install_docker_compose_via_pip: false
```

Installs `docker-compose` as default package provided by the distribution 
repository (`docker_ce_install_docker_compose` to `true`) or as pip package 
when `docker_ce_install_docker_compose_via_pip` is also set to `true`.

When installed via pip the package `python-pip` or `python3-pip` will be
installed on Debian based systems.

```yaml
# daemon configuration
docker_ce_daemon_conf_file: /etc/docker/daemon.json
# see: https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file
docker_ce_daemon_conf_content: {}
```

The `docker_ce_daemon_conf_file` gives the path to the Docker `daemon.json` 
file. `docker_ce_daemon_conf_content` dict will be converted 1:1 into JSON. See
the example Playbook below.

```yaml
# users allowed to run docker commands
docker_ce_users: []
```

List of users to add to the `docker` group, so that they can run docker 
commands.

```yaml
# additional the _userns-remap_ must be set in docker_ce_daemon_conf_content
docker_ce_subuid_subgid_create: false
docker_ce_subuids:
  - { username: "dockremap", num_start: 100000, num_count: 65536 }
docker_ce_subgids:
  - { username: "dockremap", num_start: 100000, num_count: 65536 }

```

To use *user namespace* the option `userns-remap` must be set in 
`docker_ce_daemon_conf_content` dict. When `docker_ce_subuid_subgid_create` set
to `true` the required files `/etc/subuid` and `/etc/subgid` will be created
with the contents of `docker_ce_subuids` respectively `docker_ce_subgids`.

See: [Isolate containers with a user namespace](https://docs.docker.com/engine/security/userns-remap/)

Dependencies
------------

None

Example Playbook
----------------

This simple example allows the user `hafu` to execute docker commands. The user
must exist on the system or the role will fail.

```yaml
- hosts: servers
  roles:
    - role: hafu.docker-ce
      docker_ce_users:
        - hafu
```

Here a more advanced example. It will install `docker-compose`, creates a 
`/etc/docker/daemon.json`, adds the user `hafu` to the docker group and creates
the files `/etc/subuid`/`etc/subgid` with the defaults. The option 
`userns-remap: "default"` uses the `default` keyword, which is special. Docker
will create the user and group `dockremap` automatically. Please refer to the
Docker documentation: [Isolate containers with a user namespace](https://docs.docker.com/engine/security/userns-remap/)

```yaml
- hosts: servers
  roles:
    - role: hafu.docker-ce
      docker_ce_install_docker_compose: true
      docker_ce_daemon_conf_content:
        storage-driver: "overlay2"
        userns-remap: "default"
        ipv6: true
      docker_ce_users:
        - hafu
      docker_ce_subuid_subgid_create: true
```

Resulting files will look like this:

`/etc/docker/daemon.json`:
```json
{
  "ipv6": true,
  "storage-driver": "overlay2",
  "userns-remap": "default"
}
```

`/etc/subuid`:
```
dockremap:100000:65536
```

`/etc/subgid`:
```
dockremap:100000:65536
```

License
-------

MIT

Author Information
------------------

https://github.com/hafu
